# vue项目创建以及多页面改造

## vue项目创建

### 安装脚手架工具

```
npm install @vue/cli -g
#OR
yarn install @vue/cli -g
```

如果npm 访问慢,可以使用cnpm,cnpm 安装命令

```
npm install -g cnpm --registry=https://registry.npm.taobao.org
```



### 创建一个项目

使用命令行进行初始化

```
vue create vue-project

cd vue-project

cnpm install

cnpm run serve
```

访问 localhost:8080,既可以看到页面.

### 引入 ant-design-vue

```
cnpm i --save ant-design-vue
```

调整main.js

```
import Vue from 'vue';
import App from './App.vue';
import Button from 'ant-design-vue/lib/button';
import 'ant-design-vue/dist/antd.css';

Vue.component(Button.name, Button);
Vue.config.productionTip = false

new Vue({
  render: h => h(App),
}).$mount('#app');
```

调整App.vue

```
<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/logo.png">
    <HelloWorld msg="Welcome to Your Vue.js App"/>
    <a-button type="primary">Button</a-button>
  </div>
</template>

<script>
import HelloWorld from './components/HelloWorld.vue'

export default {
  name: 'App',
  components: {
    HelloWorld
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>

```

刷新页面,就可以看到蓝色的按钮已经显示出来了

