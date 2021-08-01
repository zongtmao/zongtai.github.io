title: 实现v-if/v-show
date: 2021-08-01 23:32:12
tags:
	- 学习
	- 总结
categories: 博客

---

# 前言
> vue中v-if以及v-show开发时经常使用，我们知道他两的区别在于v-show只是控制dom元素display为none以及block，v-if则是移除新增dom，两者使用场景因此也就不同。今天我们不讲两者的区别，主要是思考vue是如何实现这两个指令，如果我们想简单的实现这两个功能，又改从哪些方面思考，所以根据个人理解，仿vue实现一个简单的v-show/v-if, 将主要的核心点整理如下，首先我们看看HTML视图以及完成后js调用

![img](https://raw.githubusercontent.com/zongtmao/markdownPicture/master/images/vue-study/preview.png)

```
<div id="app">
        <div class="box">
            <div class="item-box item-box1" v-show="showBox1">Box 1</div>
            <div class="item-box item-box2" v-if="showBox2">Box 2</div>
        </div>
        <div>
            <button class="box-btn1" @click="handleShowBox1">Box btn 1</button>
            <button class="box-btn1" @click="handleShowBox2">Box btn 2</button>
        </div>
    </div>

```
> html结构很简单，跟vue写法一样，item-box1使用v-show控制，item-box2使用v-if控制

```
<script src="index.js"></script>
    <script>
        new VShow({
            el: '#app',
            data: {
                showBox1: false,
                showBox2: false
            },
            methods: {
                handleShowBox1() {
                    this.showBox1 = !this.showBox1;
                    console.log('click box1 handle');
                },
                handleShowBox2() {
                    this.showBox2 = !this.showBox2;
                    console.log('click box2 handle');
                }
            },
        });
    </script>
```
> index.js为我们要实现的‘轮子’，理想的是跟vue一样，new一个对象实例，我们根据data里的变量控制dom元素显示隐藏，methods里的方法被点击时可通过this.showBox1这种操作data里变量的值

---

# 实现步骤

## 1. 代理数据以及数据劫持

> 由于methods里的方法都要操作实例上的变量而且data里的数据也要根据值的改变而响应视图，所以还是需要将data里的值变为响应式的数据并劫持，当放生变化时及时通知dom而发生改变，所以首先在创建的VShow类里第一步先处理数据，代码如下：

```
class VShow {
    // options就是我们new时传入的对象{el: xx, data: xx, methods: xx}
	constructor(options) {
        // 解构options
        const { el, data, methods } = options;
        this.el = document.querySelector(el);
        this.data = data;
        this.methods = methods;

        this.init();
    }
    
    init() {
        this.initData();
    }
    initData() {
        for (let key in this.data) {
            // 改变this指向，使得methods里this调用时能够调用到构造函数上
            Object.defineProperty(this, key, {
                get() {
                    return this.data[key];
                },

                set(newValue) {
                    this.data[key] = newValue;
                    console.log('设置数据', newValue);
                }
            });
        }

        console.log(this);
    }
}

```
> Object.defineProperty 主要改变this的指向，这时打印出来的this，就包含了我们new时传入的所有数据

![img](https://raw.githubusercontent.com/zongtmao/markdownPicture/master/images/vue-study/new-this.png)

## 	2. 初始化dom
> 根据data里的变量以及dom上绑定的变量初始化dom,这时我们要考虑我们需要的数据结构，因为dom任何元素上都可以使用v-show/v-if, 所以我们就需要一个dom池来存放所有绑定v-show/v-if的dom元素集合，而且每一条dom都应该有它对于的数据，如是否需要显示隐藏、是if还是show等，因此我们理想的结构为：

```
/***
   dom池
 * showPool [
 *   [
 *      dom, //每一个绑有v-show/v-if的dom
 *      {
 *         show: true/false,
 *         type: if/show,
 *         data: 绑定的变量数据,即v-show/v-if后面的变量
 *      }
 *   ]
 * ]
 * 事件池
 * eventPool [
 *    [
 *      dom,
 *      methods
 *    ]
 * ]
 */
```
> 从上面结构来看，map数据结构符合我们的需要，vue也是通过map来存放这些，button上绑定的@click事件池结构跟dom一致，所以首先我们在constructor里新增两个池子变量

```
constructor(options) {
    this.showPool = new Map();
    this.eventPool = new Map();
}
```
> 接着实现initDom方法初始化dom

```
initDom(el) {
	const _ChildNodes = el.childNodes;
	if(!_ChildNodes.length) {
		return;
	}

	_ChildNodes.forEach(element => {
		if(element.nodeType === 1) {//为dom节点时，获取所有带有v-show以及v-if、@ckick属性的dom,组装对应的数据
            const VIf = element.getAttribute('v-if');
            const VShow = element.getAttribute('v-show');
            const VEvent = element.getAttribute('@click');

            if(VIf) {
                this.showPool.set(element, {
                    type: 'if',
                    show: this.data[VIf],
                    data: VIf});
            } else if(VShow) {
                this.showPool.set(element, {
                type: 'show',
                show: this.data[VShow],
                data: VShow});
			}

        if(VEvent) {
        	this.eventPool.set(element, this.methods[VEvent]);
        }
	}

		// 再递归处理子节点
		this.initDom(element);
	});
}
```

## 	3. 初始化视图

> 第三步就是根据数据先让dom里的v-show/v-if生效即初始化视图, 就是操作刚保存的dom池里的每个元素

```
init() {
    this.initData();
    this.initDom(this.el);

    this.initViews(this.showPool);
}
// 初始化视图，根据data里的默认值初始化显示隐藏
initViews(showPool) {
	this.domChange(showPool);
}
domChange(showPool, data) {
        if(!data) {
            // data不存在是初始化dom 循环map数据，存在说明是button点击时要处理的，下一步要处理的问题
            for(let [k, v] of showPool) {
                switch (v.type) {
                    case 'if':
                        // if就比较复杂，要移除dom，但是显示的时候又要在指定位置插入dom,所以移除时要添加标识符
                        v.comment = document.createComment('v-if');
                        // v.show为false时，将dom替换成v.comment
                        !v.show && k.parentNode.replaceChild(v.comment, k)
                        break;
                    case 'show':
                        // show 使用display：none/block控制显示隐藏
                        !v.show && (k.style.display = 'none');
                        break;
                    default:
                        break;
                }
            }

            return;
        }
    }
```

> 代码里容易看出来，v-show不难实现，v-if的话必须要有占位符，不然移除后再需要插入时不知道其位置，效果如下

![img](https://raw.githubusercontent.com/zongtmao/markdownPicture/master/images/vue-study/init-view.png)

## 	4. 根据eventPool 事件处理函数的绑定

```
// 绑定的事件处理
initEvent(eventPool) {
	for(let [k, v] of eventPool) {
		// 按钮点击时需要在劫持的地方修改dom
		k.addEventListener('click', v.bind(this)); //此时html里的methods里		的方法就可以调用
	}
}
```

> 这里就简单，只需要循环eventPool，将里面的方法this指向实例里methods即可，这时点击按钮就可以出发methods里的方法

![img](https://raw.githubusercontent.com/zongtmao/markdownPicture/master/images/vue-study/init-event.png)

> 接下来要在数据劫持里修改dom，也就是调用this.domChange(this.showPool, key);方法
```
set(newValue) {
    this.data[key] = newValue;
    console.log('设置数据', newValue);
    this.domChange(this.showPool, key);
}
```

> 但domChange里我们只实现了初始化的逻辑，点击时的逻辑处理如下

```
domChange(showPool, data) {
        if(!data) {
            // data不存在是初始化dom 循环map数据
            for(let [k, v] of showPool) {
                switch (v.type) {
                    case 'if':
                        // if就比较复杂，要移除dom，但是显示的时候又要在指定位置插入dom,所以移除时要添加标识符
                        v.comment = document.createComment('v-if');
                        // v.show为false时，将dom替换成v.comment
                        !v.show && k.parentNode.replaceChild(v.comment, k)
                        break;
                    case 'show':
                        // show 使用display：none/block控制显示隐藏
                        !v.show && (k.style.display = 'none');
                        break;
                    default:
                        break;
                }
            }

            return;
        }

        // 这里是按钮点击后根据data里的变量值修改showPool的里值
        for(let [k, v] of showPool) {
            if(v.data === data) {
                switch (v.type) {
                    case 'if':
                        v.show ? k.parentNode.replaceChild(v.comment, k)
                            : v.comment.parentNode.replaceChild(k, v.comment);
                        v.show = !v.show;
                        break;
                    case 'show':
                        v.show ? (k.style.display = 'none') : k.style.display = 'block';
                        v.show = !v.show;
                        break;
                    default:
                        break;
                }
            }
        }
    }
```
> v-if主要用到parentNode.replaceChild方法，当需要隐藏时就用我们创建的comment替换之前的dom, 需要显示时将comment用保存的dom替换
> v.show ? k.parentNode.replaceChild(v.comment, k) : v.comment.parentNode.replaceChild(k, v.comment);

# 总结
这样我们就简单实现了v-if以及v-show,使用方法跟vue一样，当然这样的实现是最基本的，vue在解析dom,转虚拟dom,生成diff树等很多优化方案，这里只是提供一种实现思路，希望对理解vue源码有帮助，以下是所有代码
**index.html**
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <style>
        .box {
            width: 400px;
            height: 200px;
            display: flex;
        }
        .item-box {
            border: 1px solid #ddd;
            line-height: 200px;
            text-align: center;
            font-size: 30px;
            width: 50%;
            color: #fff;
        }
        .item-box1 {
            background: chartreuse;
        }
        .item-box2 {
            background: chocolate;
        }
    </style>
</head>
<body>
    <div id="app">
        <div class="box">
            <div class="item-box item-box1" v-show="showBox1">Box 1</div>
            <div class="item-box item-box2" v-if="showBox2">Box 2</div>
        </div>
        <div>
            <button class="box-btn1" @click="handleShowBox1">Box btn 1</button>
            <button class="box-btn1" @click="handleShowBox2">Box btn 2</button>
        </div>
    </div>

    <script src="index.js"></script>
    <script>
        new VShow({
            el: '#app',
            data: {
                showBox1: false,
                showBox2: false
            },
            methods: {
                handleShowBox1() {
                    this.showBox1 = !this.showBox1;
                    console.log('click box1 handle');
                },
                handleShowBox2() {
                    this.showBox2 = !this.showBox2;
                    console.log('click box2 handle');
                }
            },
        });
    </script>
</body>
</html>
```

**index.js**

```
class VShow {
    constructor(options) {
        // 解构options
        const { el, data, methods } = options;
        this.el = document.querySelector(el);
        this.data = data;
        this.methods = methods;

        this.showPool = new Map();
        this.eventPool = new Map();

        this.init();
    }

    init() {
        this.initData();
        this.initDom(this.el);

        this.initViews(this.showPool);
        this.initEvent(this.eventPool);
    }

    initData() {
        for (let key in this.data) {
            // 改变this指向，使得methods里this调用时能够调用到构造函数上
            Object.defineProperty(this, key, {
                get() {
                    return this.data[key];
                },

                set(newValue) {
                    this.data[key] = newValue;
                    console.log('设置数据', newValue);
                    this.domChange(this.showPool, key);
                }
            });
        }

        console.log(this);
    }

    initDom(el) {
        const _ChildNodes = el.childNodes;
        if(!_ChildNodes.length) {
            return;
        }

        _ChildNodes.forEach(element => {
            if(element.nodeType === 1) {
                const VIf = element.getAttribute('v-if');
                const VShow = element.getAttribute('v-show');
                const VEvent = element.getAttribute('@click');

                if(VIf) {
                    this.showPool.set(element, {
                        type: 'if',
                        show: this.data[VIf],
                        data: VIf
                    });
                } else if(VShow) {
                    this.showPool.set(element, {
                        type: 'show',
                        show: this.data[VShow],
                        data: VShow
                    });
                }

                if(VEvent) {
                    this.eventPool.set(element, this.methods[VEvent]);
                }
            }

            // 递归处理子节点
            this.initDom(element);
        });
    }

    // 初始化视图，根据data里的默认值初始化显示隐藏
    initViews(showPool) {
        this.domChange(showPool);
    }

    domChange(showPool, data) {
        if(!data) {
            // data不存在是初始化dom 循环map数据
            for(let [k, v] of showPool) {
                switch (v.type) {
                    case 'if':
                        // if就比较复杂，要移除dom，但是显示的时候又要在指定位置插入dom,所以移除时要添加标识符
                        v.comment = document.createComment('v-if');
                        // v.show为false时，将dom替换成v.comment
                        !v.show && k.parentNode.replaceChild(v.comment, k)
                        break;
                    case 'show':
                        // show 使用display：none/block控制显示隐藏
                        !v.show && (k.style.display = 'none');
                        break;
                    default:
                        break;
                }
            }

            return;
        }

        // 这里本来是两种逻辑
        for(let [k, v] of showPool) {
            if(v.data === data) {
                switch (v.type) {
                    case 'if':
                        v.show ? k.parentNode.replaceChild(v.comment, k)
                            : v.comment.parentNode.replaceChild(k, v.comment);
                        v.show = !v.show;
                        break;
                    case 'show':
                        v.show ? (k.style.display = 'none') : k.style.display = 'block';
                        v.show = !v.show;
                        break;
                    default:
                        break;
                }
            }
        }
    }

    // 绑定的事件处理
    initEvent(eventPool) {
        for(let [k, v] of eventPool) {
            // 按钮点击时需要在劫持的地方修改dom
            k.addEventListener('click', v.bind(this)); //此时html里的methods里的方法就可以调用
        }
    }

}
```
