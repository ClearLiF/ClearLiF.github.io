---
layout: post
title:  "vue基本操作记录"
date:   2020-4-29
categories: vue
tags:  vue
author: Clear Li
---

























* content
{:toc}
## 前言

由于学校要求要做一个跨平台的软件作业，因此必须学习一手vue了 。













## 安装

1,引用cdn

```html
<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
```

2.使用本地的vue文件

<script src="vue.js" type="text/javascript" charset="utf-8"></script>

## 声明式渲染

官网原话： Vue 在背后做了大量工作。现在数据和 DOM 已经被建立了关联，所有东西都是**响应式的**。我们要怎么确认呢？打开你的浏览器的 JavaScript 控制台 (就在这个页面打开)，并修改 `app.message` 的值，你将看到上例相应地更新。

<div id="app">
			{{ message }} {{name}}
		</div>

		<script type="text/javascript">
			var data = {message:'hello vue ',name:'liq'};
			var app = new Vue({
				el: '#app',
				data: data
			});
			data.message='hello li'
		</script>
效果

![image-20200430184830946](/img/image-20200430184830946.png)

## vue的方法

vue调用内部方法使用$这里和jQuery有点像

<script type="text/javascript">
			var data = {message:'hello vue ',name:'liq'};
			var app = new Vue({
				el: '#app',
				data: data
			});

​    

这里的watch方法是监控某一个值的前后变化，而这个变化必须在方法声明之后变化在之前变化是检测不到的。


			app.$watch('message',function(newVal,belVal){
				console.log(newVal,belVal);
			})	
			app.$data.message='1231'
			//data.message='hello li'
		</script>
演示

![image-20200430190316690](/img/image-20200430190316690.png)

及从hello vue  变为1231



##   声明周期函数

![Vue 实例生命周期](https://cn.vuejs.org/images/lifecycle.png)

```
<div id="app">
	{{msg}}
</div>
<script type="text/javascript">
var vm = new Vue({
	el : "#app",
	data : {
		msg : "hi vue",
	},
	//在实例初始化之后，数据观测 (data observer) 和 event/watcher 事件配置之前被调用。
	beforeCreate:function(){
		console.log('beforeCreate');
	},
	/* 在实例创建完成后被立即调用。
	在这一步，实例已完成以下的配置：数据观测 (data observer)，属性和方法的运算，watch/event 事件回调。
	然而，挂载阶段还没开始，$el 属性目前不可见。 */
	created	:function(){
		console.log('created');
	},
	//在挂载开始之前被调用：相关的渲染函数首次被调用
	beforeMount : function(){
		console.log('beforeMount');

	},
	//el 被新创建的 vm.$el 替换, 挂在成功	
	mounted : function(){
		console.log('mounted');
	
	},
	//数据更新时调用
	beforeUpdate : function(){
		console.log('beforeUpdate');
			
	},
	//组件 DOM 已经更新, 组件更新完毕 
	updated : function(){
		console.log('updated');
			
	}
});
setTimeout(function(){
	vm.msg = "change ......";
}, 3000);
</script>



```

测试一下

![image-20200430200737216](/img/image-20200430200737216.png)

模板语法