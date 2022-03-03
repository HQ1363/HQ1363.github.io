---
title: 前端常见面试题   
date: 2021-09-27 12:01:40   
tags:
  - vue   
  - javascript   
  - 面试   
categories:
  - vue   
  - javascript   
  - 面试   
excerpt: 最近公司在招人，面了挺多人，这里总结下招人的心得和常见问题.
---

## 前端工程师
### 技能要求
- 熟悉`Vue`/`React`其中一种
- 熟悉`Webpack`配置
- 精通`Html5`、`Javascript`，熟练掌握主流`mvc`、`mvvm`前端框架
- 具备良好的数据结构和算法知识
- 对前端有浓厚的兴趣，具有快速学习能力，乐于探索，并有良好的编码习惯

### 面试问题
- vue指令，v-show指令和v-if指令的区别是什么？
- data为什么是一个函数而不是对象
- vue常用指令
  * v-model 多用于表单元素实现双向数据绑定（同angular中的ng-model）
  * v-bind 动态绑定 作用： 及时对页面的数据进行更改
  * v-on:click 给标签绑定函数，可以缩写为@，例如绑定一个点击函数 函数必须写在methods里面
  * v-for 格式： v-for="字段名 in(of) 数组json" 循环数组或json(同angular中的ng-repeat)
  * v-show 显示内容 （同angular中的ng-show）
  * v-hide 隐藏内容（同angular中的ng-hide）
  * v-if 显示与隐藏 （dom元素的删除添加 同angular中的ng-if 默认值为false）
  * v-else-if 必须和v-if连用
  * v-else 必须和v-if连用 不能单独使用 否则报错 模板编译错误
  * v-text 解析文本
  * v-html 解析html标签
  * v-bind:class 三种绑定方法 1、对象型 '{red:isred}' 2、三元型 'isred?"red":"blue"' 3、数组型 '[{red:"isred"},{blue:"isblue"}]'
  * v-once 进入页面时 只渲染一次 不在进行渲染
  * v-cloak 防止闪烁
  * v-pre 把标签内部的元素原位输出
- 组件传值方式有哪些，父子之间的数据如果需要相互访问，可以有哪些方式？
  * 父传子：子组件通过props['xx'] 来接收父组件传递的属性 xx 的值
  * 子传父：子组件通过 this.$emit('fnName',value) 来传递,父组件通过接收 fnName 事件方法来接收回调
  * 使用Vuex
- vuex的核心概念
  * state => 基本数据
  * getters => 从基本数据派生的数据
  * mutations => 修改数据，同步
  * actions => 修改数据，异步 (Action 提交的是 mutation，而不是直接变更状态)
  * modules => 模块化Vuex
- 目前的前端开发架子，是？
  * 路由是用哪个库？
  * UI组件是哪个？
  * css解析是哪个？
  * ajax异步接口请求，用的是哪个库？
  * 图表类库，都用过哪些
  * 全局状态维护，用的是哪个库？
- css如何只在当前组件起作用？
- vue实现数据双向绑定的原理
- vue的生命周期函数，都有哪些？
- css的选择器，都有哪些？
- cookie和session的区别
- localStorage和SessionStorage的区别
- get和post的区别
- 说一说http的缓存
- 常见状态码的含义和产生原因
- 从浏览器输入url后都经历了什么
