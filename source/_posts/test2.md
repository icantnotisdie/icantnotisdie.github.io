---
title: 我的第二篇博客
date: 2020-02-26 16:00:45
tags:
---
# 为什么使用vuex?

vuex主要是做**数据交互**，父子组件传值可以很容易办到，但是兄弟组件间传值（兄弟组件下又有父子组件），或者大型SPA单页面框架项目，页面多并且一层嵌套一层的传值，异常麻烦，用vuex来维护共有的状态或数据会显得得心应手。

# 需求

需求：两个组件A和B，vuex维护的公共数据是 餐馆的名称 resturantName,默认餐馆名称是 飞歌餐馆，那么现在A和B页面显示的就是飞歌餐馆。如果A修改餐馆名称 为 A餐馆，则B页面显示的将会是 A餐馆，反之B修改同理。这就是vuex维护公共状态或数据的魅力，在一个地方修改了数据，在这个项目的其他页面都会变成这个数据。

![需求图片展示图](https://note.youdao.com/yws/public/resource/a884292c312af43606f39017c99fd60a/xmlnote/DF5C621522B94E52B2FB7ED1E7CFFBC4/16941)

# 实现步骤

### 1.使用 `vue-cli`脚手架工具创建一个工程项目，工程目录，创建 组件A 和 组件B 路由，如下：

![](https://note.youdao.com/yws/public/resource/a884292c312af43606f39017c99fd60a/xmlnote/C9D8D958725D404894CF11DA090C664F/16947)

**router.js文件：**

```javascript
import Vue from 'vue'
import Router from 'vue-router'
import componentsA from '@/components/componentsA'
import componentsB from '@/components/componentsB'
 
Vue.use(Router)
 
export default new Router({
   mode: 'history',
    routes: [
        {
        path: '/',
        name: 'componentsA',
        component: componentsA
        },
        {
            path: '/componentsA',
            name: 'componentsA',
            component: componentsA
        },
        {
            path: '/componentsB',
            name: 'componentsB',
            component: componentsB
        }
    ]
})
```

**app.vue文件：**

```html
<template>
  <div id="app">
    <router-view/>
  </div>
</template>
 
<script>
export default {
  name: 'App'
}
</script>
 
<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
```

### 2.开始使用vuex，新建一个 sotre文件夹，分开维护 actions mutations getters

![](https://note.youdao.com/yws/public/resource/a884292c312af43606f39017c99fd60a/xmlnote/EFACA0EE4EAF4A709AE4A7E22B217548/16969)

### 3.在store/index.js文件中新建vuex 的store实例
*as的意思是 导入这个文件里面的所有内容，就不用一个个实例来导入了。

```javascript
import Vue from 'vue'
import Vuex from 'vuex'
import * as getters from './getters' // 导入响应的模块，*相当于引入了这个组件下所有导出的事例
import * as actions from './actions'
import * as mutations from './mutations'
Vue.use(Vuex)
// 首先声明一个需要全局维护的状态 state,比如 我这里举例的resturantName
const state = {
    resturantName: '飞歌餐馆' // 默认值
    // id: xxx  如果还有全局状态也可以在这里添加
    // name:xxx
}
 
// 注册上面引入的各大模块
const store = new Vuex.Store({
    state,    // 共同维护的一个状态，state里面可以是很多个全局状态
    getters,  // 获取数据并渲染
    actions,  // 数据的异步操作
    mutations  // 处理数据的唯一途径，state的改变或赋值只能在这里
})
 
export default store  // 导出store并在 main.js中引用注册。
```

### 4.actions.js文件
```javascript

// 给action注册事件处理函数。当这个函数被触发时候，将状态提交到mutations中处理
export function modifyAName({commit}, name) { // commit 提交；name即为点击后传递过来的参数，此时是 'A餐馆'
    return commit ('modifyAName', name)
}
export function modifyBName({commit}, name) {
    return commit ('modifyBName', name)
}
 
// ES6精简写法
// export const modifyAName = ({commit},name) => commit('modifyAName', name)
```

### 5.mutations.js文件
```javascript
// 提交 mutations是更改Vuex状态的唯一合法方法
export const modifyAName = (state, name) => { // A组件点击更改餐馆名称为 A餐馆
    state.resturantName = name // 把方法传递过来的参数，赋值给state中的resturantName
}
export const modifyBName = (state, name) => { // B组件点击更改餐馆名称为 B餐馆
    state.resturantName = name
}
```

### 6.getters.js
```javascript
// 获取最终的状态信息
export const resturantName = state => state.resturantName
```

### 7.在main.js中导入 store实例
```javascript
// The Vue build version to load with the `import` command
// (runtime-only or standalone) has been set in webpack.base.conf with an alias.
import Vue from 'vue'
import App from './App'
import router from './router'
import store from './store'
 
Vue.config.productionTip = false
 
/* eslint-disable no-new */
new Vue({
  el: '#app',
  router,
  store,  // 这样就能全局使用vuex了
  components: { App },
  template: '<App/>'
})
```

### 8.在组件A中，定义点击事件，点击 修改 餐馆的名称，并把餐馆的名称在事件中用参数进行传递。

`...mapactions` 和 `...mapgetters`都是`vuex`提供的语法糖，在底层已经封装好了，拿来就能用，简化了很多操作。

其中`...mapActions(['clickAFn'])` 相当于`this.$store.dispatch('clickAFn'，{参数})`，`mapActions`中只需要指定方法名即可，参数省略。

`...mapGetters(['resturantName'])`相当于`this.$store.getters.resturantName`

```html
<template>
  <div class="componentsA">
      <P class="title">组件A</P>
      <P class="titleName">餐馆名称：{{resturantName}}</P>
      <div>
            <!-- 点击修改 为 A 餐馆 -->
          <button class="btn" @click="modifyAName('A餐馆')">修改为A餐馆</button>
      </div>
      <div class="marTop">
          <button class="btn" @click="trunToB">跳转到B页面</button>
      </div>
  </div>
</template>
 
<script>
import {mapActions, mapGetters} from 'vuex'
export default {
  name: 'A',
  data () {
    return {
    }
  },
  methods:{
      ...mapActions( // 语法糖
          ['modifyAName'] // 相当于this.$store.dispatch('modifyName'),提交这个方法
      ),
      trunToB () {
          this.$router.push({path: '/componentsB'}) // 路由跳转到B
      }
  },
  computed: {
      ...mapGetters(['resturantName']) // 动态计算属性，相当于this.$store.getters.resturantName
  }
}
</script>
 
<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
    .title,.titleName{
        color: blue;
        font-size: 20px;
    }
    .btn{
        width: 160px;
        height: 40px;
        background-color: blue;
        border: none;
        outline: none;
        color: #ffffff;
        border-radius: 4px;
    }
    .marTop{
        margin-top: 20px;
    }
</style>
```

B组件同理

```html
<template>
  <div class="componentsB">
      <P class="title">组件B</P>
      <P class="titleName">餐馆名称：{{resturantName}}</P>
      <div>
          <!-- 点击修改 为 B 餐馆 -->
          <button class="btn" @click="modifyBName('B餐馆')">修改为B餐馆</button>
      </div>
      <div class="marTop">
          <button class="btn" @click="trunToA">跳转到A页面</button>
      </div>
  </div>
</template>
 
<script>
import {mapActions, mapGetters} from 'vuex'
export default {
  name: 'B',
  data () {
    return {
    }
  },
  methods:{
      ...mapActions( // 语法糖
          ['modifyBName'] // 相当于this.$store.dispatch('modifyName'),提交这个方法
      ),
      trunToA () {
          this.$router.push({path: '/componentsA'}) // 路由跳转到A
      }
  },
  computed: {
      ...mapGetters(['resturantName']) // 动态计算属性，相当于this.$store.getters.resturantName
  }
}
</script>
 
<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
    .title,.titleName{
        color: red;
        font-size: 20px;
    }
    .btn{
        width: 160px;
        height: 40px;
        background-color: red;
        border: none;
        outline: none;
        color: #ffffff;
        border-radius: 4px;
    }
    .marTop{
        margin-top: 20px;
    }
</style>
```

<br><br><br><br>






