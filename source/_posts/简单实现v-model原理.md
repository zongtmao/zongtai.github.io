title: 简单实现vue v-model双向数据绑定
date: 2021-07-31 22:10:12
tags:
	- 学习
	- 总结
categories: 博客

---

# 前言
> 对于传统的dom操作，当数据变化时更新视图需要先获取到目标节点，然后将改变后的值放入节点中，视图发生变化时，需要绑定事件修改数据。双向数据恰好能解决这种复杂的操作，当数据发生变化时会自动更新视图，视图发生变化时也会自动更新数据，极大的提高了开发效率。那双向数据绑定到底是怎么实现的了，下面的例子来实现vue中v-model，完成后调用如下

```
<div id="mvvm-app">
        <input type="text" v-model="name" placeholder="姓名" />
        <input type="text" v-model="age" placeholder="年龄" />
        <div>
            <p>姓名：<span>{{name}}</span></p>
            <p>
                <p>年龄：<span>{{age}}</span></p>
            </p>
        </div>
        <button id="btn">设置姓名为zongtmao并不操作dom</button>
    </div>

```

```
<script>
        const MvvmApp = new MVVM('#mvvm-app', {
            name: '',
            age: ''
        });

        document.querySelector('#btn').addEventListener('click', () => {
            MvvmApp.setName('name', 'zongtmao');
        });
    </script>
```

---

# 实现双向数据绑定的几种做法
目前几种主流的mvc(vm)框架都实现了单向数据绑定，而我所理解的双向数据绑定无非就是在单向绑定的基础上给可输入元素（input、textare等）添加了change(input)事件，来动态修改model和 view，并没有多高深。所以无需太过介怀是实现的单向或双向绑定。

实现数据绑定的做法有大致如下几种：
> 发布者-订阅者模式（backbone.js）
> 脏值检查（angular.js）
> 
> 数据劫持（vue.js）

**本文以vue数据劫持的两种方式实现v-model**

## 	1. Object.defineProperty
> Vue实现双向数据绑定是采用数据劫持和发布者-订阅者模式。数据劫持是利用ES5的Object.defineProperty(obj,key,val)方法来劫持每个属性的getter和setter，在数据变动是发布消息给订阅者，从而触发相应的回调来更新视图

- 创建MVVM构造函数并初始化数据为响应式数据
```
class MVVM {
    constructor(el, data) {
        this.el = document.querySelector(el);
        // 定义临时存储数据的变量_data
        this._data = data; //Object.defineProperty使用

        this.init();
    }
    init() {
        // 初始化数据，将数据变为响应式的数据
        this.initData();
    }
    
    initData() {
        const _this = this;
        this.data = {};
        for (const key in this._data) {
            // 数据劫持
            Object.defineProperty(this.data, key, {
                get() {
                    return _this._data[key];
                },

                set(newValue) {
                    _this._data[key] = newValue;
                }
            });
        }
     }
}
```
- 数据处理完成后，第二步来监听绑定input的keyup事件，事件改变，数据劫持改变
> 思路：找到el下面所有input输入框，找出它的属性v-model,再将它上面的value赋值给上面响应式数据
```
// 处理input绑定的事件input/keyup
    bindInput(el) {
        const _allInput = el.querySelectorAll('input');
        _allInput.forEach(input => {
            // 获取v-model属性
            const _vModel = input.getAttribute('v-model');
            if(_vModel) { //这里简单判断，只为了实现v-model，其他的不扩展
                input.addEventListener('keyup', this.handleInput.bind(this, _vModel, input), false)
            }
        });
    }

    handleInput(key, input) {
        const _value = input.value;
        this.data[key] = _value;

        console.log(this.data);
    }
```
- 处理dom中带有双花括号的模板语法
> 思路：使用递归找到所有符合双花括号语法的父节点，用一个dom池存放起来，再通过双花括号里特定的key值替换dom池中节点的innerHTML即可，每次数据发生变化就会执行，这样就实现了数据更新视图，统称：模板解析

```
constructor(el, data) {
        this.el = document.querySelector(el);
        // 定义临时存储数据的变量_data
        this._data = data; //Object.defineProperty使用
        // dom池，存放符合双花括号模板语法的dom
        this.domPool = {};

        this.init();
    }
```

```
// 在#el下递归处理带有双花括号标识的dom
    bindDom(el) {
        const _childNodes = el.childNodes;
        _childNodes.forEach(item => {
            // 判断当前节点是不是文本节点
            if(item.nodeType === 3) {
                const _value = item.nodeValue;
                if(_value.trim().length) {
                    // 判断是否符合双花括号语法条件
                    let isValid = /\{\{(.+?)\}\}/.test(_value);
                    if(isValid) {
                        // 拿到双花括号中的key值
                        const _key = _value.match(/\{\{(.+?)\}\}/)[1].trim();
                        // 这样每次就可以根据key替换它的innerText
                        this.domPool[_key] = item.parentNode;
                        // dom节点赋值，没有的话默认undfined
                        item.parentNode.innerText = this.data[_key] || undefined;
                    }
                }
            }
            // 如果item存在子节点，递归处理dom
            item.childNodes && this.bindDom(item);
        });
    }
```
- 到这里基本的v-model已经实现，如果想要设置数据改变视图，不通过操作dom就很简单，看下面例子
> 点击按钮设置名字，但是不通过dom, 在构造函数内新增setName方法,只需要传对应的key和value即可，通过改变响应式的数据就能使视图发生变化

```
// 设置dom数据，不操作dom
    setName(key, value) {
        this.data[key] = value;
    }
```
## 	2. Proxy实现
> Proxy主要是vue3版本实现双向数据绑定的方法，对于上面实现的例子，将初始化数据initData函数Object.defineProperty改造成proxy即可，代码如下

```
constructor(el, data) {
        this.el = document.querySelector(el);
        this.data = data;
        // dom池，存放符合双花括号模板语法的dom
        this.domPool = {};

        this.init();
    }
```

```
initData() {
        const _this = this;
        /**
         * Proxy方式实现
         */
        this.data = new Proxy(this.data, {
            get(target, key) {
                return Reflect.get(target, key);
            },

            set(target, key, value) {
                _this.domPool[key].innerHTML = value;
                return Reflect.set(target, key, value);
            }
        });
        console.log(this.data);
    }

```

## 	3 完整js代码
```
class MVVM {
    constructor(el, data) {
        this.el = document.querySelector(el);
        // 定义临时存储数据的变量_data
        // this._data = data; //Object.defineProperty使用
        this.data = data;
        // dom池，存放符合双花括号模板语法的dom
        this.domPool = {};

        this.init();
    }

    init() {
        // 初始化数据，将数据变为响应式的数据
        this.initData();
        // dom初始化
        this.initDom()
    }

    initDom() {
        this.bindDom(this.el);
        this.bindInput(this.el);
    }


    initData() {
        const _this = this;
        // this.data = {};
        // for (const key in this._data) {
        //     // 数据劫持
        //     Object.defineProperty(this.data, key, {
        //         get() {
        //             return _this._data[key];
        //         },

        //         set(newValue) {
        //             // 有了domPool，就可以根据key值替换成新的数据
        //             _this.domPool[key].innerHTML = newValue;
        //             _this._data[key] = newValue;
        //         }
        //     });
        // }
        
        /**
         * Proxy方式实现
         */
        this.data = new Proxy(this.data, {
            get(target, key) {
                return Reflect.get(target, key);
            },

            set(target, key, value) {
                _this.domPool[key].innerHTML = value;
                return Reflect.set(target, key, value);
            }
        });
        console.log(this.data);
    }

    // 在#el下递归处理带有双花括号标识的dom
    bindDom(el) {
        const _childNodes = el.childNodes;
        _childNodes.forEach(item => {
            // 判断当前节点是不是文本节点
            if(item.nodeType === 3) {
                const _value = item.nodeValue;
                if(_value.trim().length) {
                    // 判断是否符合双花括号语法条件
                    let isValid = /\{\{(.+?)\}\}/.test(_value);
                    if(isValid) {
                        // 拿到双花括号中的key值
                        const _key = _value.match(/\{\{(.+?)\}\}/)[1].trim();
                        // 这样每次就可以根据key替换它的innerText
                        this.domPool[_key] = item.parentNode;
                        // dom节点赋值，没有的话默认undfined
                        item.parentNode.innerText = this.data[_key] || undefined;
                    }
                }
            }
            // 如果item存在子节点，递归处理dom
            item.childNodes && this.bindDom(item);
        });
    }


    // 处理input绑定的事件input/keyup
    bindInput(el) {
        const _allInput = el.querySelectorAll('input');
        _allInput.forEach(input => {
            // 获取v-model属性
            const _vModel = input.getAttribute('v-model');
            if(_vModel) { //这里简单判断，只为了实现v-model，其他的不扩展
                input.addEventListener('keyup', this.handleInput.bind(this, _vModel, input), false)
            }
        });
    }

    handleInput(key, input) {
        const _value = input.value;
        this.data[key] = _value;

        console.log(this.data);
    }

    // 设置dom数据，不操作dom
    setName(key, value) {
        this.data[key] = value;
    }
}
```


# 总结
主要通过实现MVVM这个模块来阐述了双向绑定的原理。并根据思路流程渐进梳理讲解了一些细节思路和比较关键的内容点，以及通过展示部分关键代码讲述了怎样一步步实现一个双向绑定MVVM。文中肯定会有一些不够严谨的思考和错误，欢迎大家指正，有兴趣欢迎一起探讨和改进~
