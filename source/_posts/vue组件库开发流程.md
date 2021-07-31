title: hexo + git博客搭建
date: 2021-06-23  19:10:16
tags:
	- 学习
	- 总结
categories: 博客

---

# 前言
> 目前项目中使用的是element-ui组件库，基本上能够满足日常的需求开发，但是在某些复杂交互的实现上，就需要自己重新封装（造轮子），如果时间允许可以根据项目开发一套我们自己的组件库，当然这个过程能够学到很多的知识点，下面以vue2版本和vue3版本为例，简单梳理下开发组件库的流程。

---


 # 环境搭建

> 使用vue-cli脚手架搭建，根据vue不同版本安装即可，这里不多做介绍

# vue2版本

1. ### main.js里全局引入注册方式
>  我们想像element-ui那样，安装后在main.js里全局引入，项目中所有的组件中就都可以使用我们组件库中的组件，以封装VaButton和VaInput组件为例，如：

```main.ts
//modules文件夹va-ui下存放组件

import VaUI from "../modules/va-ui"
import "../modules/va-ui/common.css"

// 第二个参数为一个对象，我们可以将要使用的组件按需注册使用
Vue.use(VaUI, {
  components: [
    "VaButton",
    "VaInput"
  ]
})

new Vue({
  render: h => h(App),
}).$mount('#app')
```
* 如何实现上面这种，我们首先在va-ui文件夹下创建button跟input文件夹，分别在两个文件夹下开发组件,这里以最简单的button为例，里面的功能未完善，只是验证流程

```
va-ui/button/index.vue

<template>
    <div>
        <button :class="['va-button', type]">
            <slot></slot>
        </button>
    </div>
</template>

<script>
    export default {
        name: "VaButton",
        props: {
            type: String
        }
    }
</script>

<style lang="scss" scoped>
.va-button {
    height: 32px;
    padding: 0 15px;
    &.primary {
        color: #fff;
        background-color: #409eff;
        border-color: #409eff;
    }
    &.warning {
        color: #fff;
        background-color: #e6a23c;
        border-color: #e6a23c;
    }
    &.danger {
        color: #fff;
        background-color: #f56c6c;
        border-color: #f56c6c;
    }
    &.success {
        color: #67c23a;
        background: #f0f9eb;
        border-color: #c2e7b0;
    }
}
</style>

```

* 在va-ui文件夹下创建index.ts

```
/modules/va-ui/index.ts

//分别引入两个组件
import Button from "./button/index.vue"
import Input  from './input/index.vue'

//定义组件库对象，vue中插件实际上就是一个对象
const VaUI: any = {}

//定义所有组件的列表，循环注册
const COMPONENTS: any = [
    Button,
    Input
]

//使用install方法注册，其中的options就是Vue.use时传的第二个参数，这时可以根据组件名按需注册
VaUI.install = function(Vue: any, options: any) {
    if(options && options.components) {
        const components = options.components;
        components.forEach((componentName: any) => {
            COMPONENTS.forEach((component: any) => {
                if (componentName === component.name) {
                    Vue.component(component.name, component);
                }
            });
        });
    } else {
        COMPONENTS.forEach((component: any) => {
            Vue.component(component.name, component)
        });
    }
}

//抛出VaUI对象
export default VaUI;
```

* 此时全局注册的方式已经实现了，我们在组件里就可以直接使用我们刚定义好的VaButton和VaInput组件

```
/src/app.vue

<template>
  <div id="app">
    <va-button type="primary">我的按钮</va-button>
    <va-input placeholder="enter you name"></va-input>
  </div>
</template>

```

* 总结：主要是Vue.use以及install方法的使用，现在在我们创建的modules文件夹里直接引入的，后面开发完发布到npm，就可以安装使用，这只是很简单的实现，真正的组件库还涉及很多的知识，比如一些公共方法，混入，自定义指令等等，但思路是一样的

---

2. 按解构的方式引入

> 我们想要像element那样，通过解构的方式引入使用，组件库如何开发呢？

* main.js里解构使用

```
import { VaButton, VaInput } from "../modules/va-ui"

//分别use
Vue.use(VaButton)
Vue.use(VaInput)

```

* 此时index.ts写法

```
/modules/va-ui/index.ts

import Button from "./button/index.vue"
import Input  from './input/index.vue'

//分别定义组件对象
const VaButton:any = {};
const VaInput:any = {};

//分别install
VaButton.install = (Vue: any) => Vue.component(Button.name, Button)
VaInput.install = (Vue: any) => Vue.component(Input.name, Input)

//最后导出即可
export {
    VaButton,
    VaInput
}

```

* 后续开发组件库可以兼容这两种不同的写法，使用者根据需要安装引入


# vue3版本
> vue3版本开发组件库跟vue2开发组件库的思路是一样的，只不过Vue3的写法跟Vue2版本有所区别，主要是组件开发时的写法，以VaSelect组件为例，附上代码

* main.js
```
// 全局引入
import VaUI from "../modules/va-ui"

const app = createApp(App);

app.use(VaUI)
app.mount('#app')

//解构引入
import { VaSelect } from "../modules/va-ui"

const app = createApp(App);

app.use(VaSelect)
app.mount('#app')

```

* /modules/va-ui/select/index.js

```
import Select from "./select/index.vue"

const COMPONENTS = [
    Select
]

//全局注册
const VaUI = {}

VaUI.install = function(Vue) {
    COMPONENTS.forEach(component => {
        Vue.component(component.name, component)
    })
}

export default VaUI

//解构注册
const VaSelect = {};

export VaSelect.install = (Vue) => Vue.component(Select.name, Select);

```

* /modules/va-ui/select/index.vue

```
<template>
    <div class="my-select">
        <div 
            class="result"
            @click="openOption"  
        >
            {{data[curIdx].label}}
        </div>
        <div class="options" v-show="optionsShow">
            <div class="option"
                v-for="(item, index) of data"
                :key="index"
                @click="setOptions(index, item)"
            >
            {{item.label}}
            </div>
        </div>
    </div>
</template>

<script>
    import { reactive, toRefs } from 'vue'
    
    export default {
        name: "VaSelect",
        props: {
            data: Array,
            currentIndex: {
                type: Number,
                default: 0
            },
            callback: Function
        },
        setup(props) {
            //定义响应式数据
            const state  = reactive({
                curIdx: props.currentIndex,
                optionsShow: false
            })

            const openOption = () => {
                state.optionsShow = true;
            }

            const setOptions = (index, item) => {
                state.curIdx = index;
                state.optionsShow = false;
                props.callback(index, item);
            }

            return {
                ...toRefs(state),
                openOption,
                setOptions
            }
        }
    }
</script>
```


* app.vue引入使用

```
<template>
  <va-select
    :data="data"
    :currentId="currentId"
    :callback="selectedOption"
  >
  </va-select>
</template>

<script>
import { ref } from 'vue'

export default {
  name: 'App',
  setup() {
    const data = [
      {
        id: 1,
        value: "select1",
        label: "选择1"
      },
      {
        id: 2,
        value: "select2",
        label: "选择2"
      }
    ];
    
    const currentId = ref(1);

    const selectedOption = (index, item) => {
      console.log(index, item);
    }

    return {
      data,
      currentId,
      selectedOption
    }
  }
}
</script>
```

## 未完待续...