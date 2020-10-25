# Vue使用

## mustache 标签

```
<span> Message: {{ msg }} </span>
```



mustache标签将会被替代为对应数据对象msg properties的值,无论何时,绑定的数据对象上的msg property发生变化,插值处的内容都会更新



## Attribute

mustache 语法不能作用在html attribute上,遇到这种情况应该使用v-bind指令

```
<div v-bind:id="dynamicId></div>
```



## 计算属性和侦听器

### 计算属性缓存vs方法

计算属性是基于它们的响应式依赖进行缓存的,只在相关响应式依赖发生改变时它们才会重新计算

### 计算属性vs侦听属性



### 计算属性的setter

计算属性默认只有getter



## 条件属性

### v-if v-else v-else-if

### v-show

v-show的元素始终会被渲染并保留在dom中.v-show只是简单的切换元素的css property display

### v-if vs v-show

v-if是真正的条件渲染,因为它会确保在切换过程中条件块内的事件监听器和子组件适当的被销毁和重建

v-if也是惰性的,如果在初始渲染时条件为假,则什么都不做知道条件第一次变成真才会开始渲染条件块

一般来说,v-if具有更高的切换开销,而v-show有更高的初始渲染开销.因此如果非常频繁的切换,建议使用v-show,如果在运行时条件很少改变,则使用v-if比较好.

### v-for vs v-if

不推荐同时使用v-for和v-if

当v-for和v-if一起使用时,v-for具有比v-if更高的优先级



