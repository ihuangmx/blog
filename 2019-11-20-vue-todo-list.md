# Vue 入门第一课

## 准备工作

新建一个 HTMl 页面，在头部直接引入 CDN

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Vue</title>
	<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
</head>
<body>


<div id="app">
	<!-- HTML 的内容写在这 -->
</div>

<script>
	<!-- JavaScript 的内容写在这 -->
</script>
</body>
</html>
```

接下来的所有教程都会在该页面上实现，只需要将 HTML 和 JavaScript 的内容写在对应的地方即可。

## 标题

> 给任务列表创建一个标题。

HTML - `{{ }}` 语法叫做插值模板语法，可用于渲染数据，对应元素的 `textContent` 属性

```html
<h1>{{ title }}</h1>
```

JavaScript

```js
// 1. 应用始终通过 Vue 函数创建实例
var vm = new Vue({
	// 2. Vue 的实例必须挂载到某个元素上，可以通过 el 属性来设置
	el: '#app',
	// 3. data 用于所有的属性，这些数据的更新将反映到视图上面
	data: {
		title: '任务列表'
	}
})
```

Vue 是声明式的，声明自己要什么即可，不需要考虑如何实现

```html
<h1>{{ title }}</h1>
```

与此相比，命令式则需要考虑的是如何去实现

```js
$('h1').text('任务列表');
```

## 输入框

> 创建一个输入框，用于添加任务。

HTML - `{{ }}` 语法对应的是元素的 `textContent` 属性，如果是表单的话则需要使用 `v-model` 指令

```html
<h1>{{ title }}</h1>
<input v-model="task">
```

JavaScript - 新增属性 `task`，初始值为空

```js
var vm = new Vue({
	el: '#app',
	data: {
		title: '任务列表',
		task: null
	}
})
```

## 新增任务

> 用户输入任务，点击新增后，添加任务到任务列表中。

HTML - Vue 中使用 `v-on` 来进行事件监听

```html
<h1>{{ title }}</h1>
<input v-model="task">
<button v-on:click="add">新增</button>
```

JavaScript 

```js
// 1、用于生成唯一标识
function guid() {
    function S4() {
       return (((1+Math.random())*0x10000)|0).toString(16).substring(1);
    }
    return (S4()+S4()+"-"+S4()+"-"+S4()+"-"+S4()+"-"+S4()+S4()+S4());
}

var vm = new Vue({
	el: '#app',
	data: {
		title: '任务列表',
		task: null,
		// 2、新增 tasks，管理任务
		tasks: []
	},
	methods: {
		add() {
			// 3、给新的任务添加唯一标识，并设置为未完成
			var task = {
				id: guid(), 
				name : this.task,
				completed: false 
			}
			// 4、、将任务添加到列表中
			this.tasks.push(task);  
			// 5、清空任务框
			this.task = null; 
		}
	}
})
```

## 显示任务

> 显示任务列表，并且在任务列表为空的时候，隐藏标题。

HTML - 任务列表是一组列表数据，无法直接使用 `{{ }}` 等语法来显示，需要用 `v-for` 指令来渲染一组列表数据。

```html
<h1>{{ title }}</h1>
<input v-model="task">
<button v-on:click="add">新增</button>

<h2>全部任务</h2>
<ul>
	<li v-for="task in tasks">
		{{ task.name }}
	</li>
</ul>
```

当任务列表为空的时候，隐藏标题

```html
<div v-if="tasks.length > 0">
	<h2>全部任务</h2>
	<ul>
		<li v-for="task in tasks">
			{{ task.name }}
		</li>
	</ul>
</div>
```

## 标记任务

> 将任务标记为已完成。

HTML - 新增一个按钮，用于标记任务，当 `task.completed` 的值为 `null`、`undefined` 或 `false` 时，则不会显示属性 `disabled`。

```html
<h2>全部任务</h2>
<ul>
	<li v-for="task in tasks">
		{{ task.name }} <button v-bind:disabled="task.completed" v-on:click="done(task.id)">完成</button>
	</li>
</ul>
```

JavaScript - 根据 `id` 匹配对应的任务，并标记为已完成

```js
methods: {
	done(id) {
		this.tasks = this.tasks.map(function(task){
			if(task.id === id){
				task.completed = true;
			}

			return task;
		})
	}
}
```

## 显示未完成的任务

用之前的方法来显示未完成任务。

HTML

```html
<h2>未完成的任务</h2>

<ul>
	<li v-for="task in uncompletedTasks()">{{ task.name }}</li>
</ul>
```

JavaScript

```js
methods: {
	uncompletedTasks() {
		return this.tasks.filter(function(task){
			return !task.completed;
		});
	}
},
```

这样做有两个比较不好的地方：

* 每次都需要重新计算未完成的任务，即使任务没有发生变化；
* 在模板语法中使用函数显得不够直观

使用计算属性进行重构

HTML

```html
<h2>未完成的任务</h2>

<ul>
	<li v-for="task in uncompletedTasks">{{ task.name }}</li>
</ul>
```

JavaScript

```js
computed: {
	uncompletedTasks() {
		return this.tasks.filter(function(task){
			return !task.completed;
		});
	}
}
```

这样写显得更加的直观，同时，计算属性只有在其关联数据发生变化时才会重新计算，否则会返回先前的结果。

## 组件化 1

任务列表的功能在很多地方都有可能用得到，有没有办法将其复用？有的，那就是将其封装成组件。

组件可以看做是封装过的 Vue 实例，也就是说，组件与实例的大都数功能都相同。将刚才的实例直接进行封装

HTML

```html
<task-list></task-list>
```

JavaScript

```js
	function guid() {
	    function S4() {
	       return (((1+Math.random())*0x10000)|0).toString(16).substring(1);
	    }
	    return (S4()+S4()+"-"+S4()+"-"+S4()+"-"+S4()+"-"+S4()+S4()+S4());
	}

	Vue.component('task-list', {
		template: `
			<div>
				<div>
					<input v-model="task">
					<button v-on:click="add">新增</button>
				</div>,
				<h2>全部任务</h2>
				<ul>
					<li v-for="task in tasks">
						{{ task.name }} <button v-bind:disabled="task.completed" v-on:click="done(task.id)">完成</button>
					</li>
				</ul>
			</div>
		`,
		data() {
			return {
				tasks : [],
				task: null
			}
		},
		methods: {
			add() {
				var task = {
					id: guid(),
					name : this.task,
					completed: false
				}
				this.tasks.push(task); 
				this.task = null; 
			},
			done(id) {
				this.tasks = this.tasks.map(function(task){
					if(task.id === id){
						task.completed = true;
					}

					return task;
				})
			}
		}
	})


	var vm = new Vue({
		el: '#app',
	})
```

该组件与之前的 Vue 实例几乎一样，最大的区别在于使用 `data()` 方法来管理数据。因为组件涉及到复用问题，假如任务列表组件在多个地方使用到，那么每个地方都需要维护自己的数据，如果使用 `data` 属性而不是方法，那么这些组件就会共享一份数据，造成相互污染。

## 组件化 2

之前对任务列表直接进行了封装，实际上，任务列表组件还可以进一步拆分成 **列表组件** 和 **详细任务组件** 两个。

子组件 Task

* 显示具体任务
* 标记任务完成

```js
Vue.component('task', {
	template: `
		<div>
			<li>{{ name }}</li><button>完成</button>
		</div>
	`
})
```

父组件 TaskList

* 维护任务列表
* 使用子组件 `task`

```js
Vue.component('task-list', {
	template:`
		<div>
			<ul>
				<task></task>
			</ul>
		</div>
	`,
})
```

这里的关键在于这两个组件应当如何通讯。

* 父组件与子组件通讯 - 父组件 `task-list` 需要将 `tasks` 数据传递给子组件。
* 子组件与父组件通讯 - 子组件标记完成后，父组件需要更新对应的任务列表

父传子 - 子组件通过 `props` 选项定义可接受的属性

```js
Vue.component('task', {
	props: ['completed', 'id', 'name'],
	template: `
		<div>
			<li>{{ name }}</li><button v-bind:disabled="completed">完成</button>
		</div>
	`
})
```

子传父 - 子组件通过事件向父组件发出消息

```js
Vue.component('task', {
	props: ['completed', 'id', 'name'],
	template: `
		<div>
			<li>{{ name }}</li><button v-bind:disabled="completed" v-on:click="$emit('done', id)">完成</button>
		</div>
	`
})
```

父组件对子组件的事件进行监听

```js
<task v-on:done="done" v-for="task in tasks" v-bind:name="task.name" v-bind:id="task.id" v-bind:completed="task.completed"></task>
```

## 完整代码

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
	<meta charset="UTF-8">
	<title>任务列表</title>
	<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
</head>
<body>
<!-- HTML 部分都会写在这里 -->
<div id="app">
	<task-list></task-list>	
</div>
	
<!-- JavaScript 部分都会写在这里 -->
<script>

	function guid() {
	    function S4() {
	       return (((1+Math.random())*0x10000)|0).toString(16).substring(1);
	    }
	    return (S4()+S4()+"-"+S4()+"-"+S4()+"-"+S4()+"-"+S4()+S4()+S4());
	}

	Vue.component('task', {
		props: ['completed', 'id', 'name'],
		template: `
			<div>
				<li>{{ name }}</li><button v-bind:disabled="completed" v-on:click="$emit('done', id)">完成</button>
			</div>
		`
	})

	Vue.component('task-list', {
		template:`
			<div>
				<input v-model="task">
				<button v-on:click="add">新增</button>
				<h2>全部任务</h2>
				<ul>
					<task v-on:done="done" v-for="task in tasks" v-bind:name="task.name" v-bind:id="task.id" v-bind:completed="task.completed"></task>
				</ul>
			</div>
		`,
		data() {
			return {
				tasks : [],
				task: null
			}
		},
		methods: {
			add() {
				var task = {
					id: guid(),
					name : this.task,
					completed: false
				}
				this.tasks.push(task);   // 将任务添加到列表中，默认为 false
				this.task = null;  // 清空任务框
			},
			done(id) {
				this.tasks = this.tasks.map(function(task){
					if(task.id === id){
						task.completed = true;
					}

					return task;
				})
			}
		}
	})

	var vm = new Vue({
		el: '#app',
	})
</script>
</body>
</html>
```

## 在 Vue Cli 中使用

安装

```bash
$ npm install -g @vue/cli
$ vue create my-vue-app
$ cd my-vue-app
```

创建 Task 组件 - `/src/components/Task.vue`

```vue
<template>
  <div>
    <li>{{ name }}</li>
    <button v-bind:disabled="completed" v-on:click="$emit('done', id)">完成</button>
  </div>
</template>

<script>
export default {
  props: ['completed', 'id', 'name']
}
</script>
```

创建 TaskList 组件 - `/src/components/TaskList.vue`

```vue
<template>
	<div>
		<input v-model="task">
		<button v-on:click="add">新增</button>
		<h2>全部任务</h2>
		<ul>
			<task v-on:done="done" v-for="task in tasks" :name="task.name" :id="task.id" :key="task.id" :completed="task.completed"></task>
		</ul>
	</div>
</template>

<script>
import Task from './Task.vue'

function guid() {
    function S4() {
       return (((1+Math.random())*0x10000)|0).toString(16).substring(1);
    }
    return (S4()+S4()+"-"+S4()+"-"+S4()+"-"+S4()+"-"+S4()+S4()+S4());
}

export default {
	data() {
		return {
			tasks : [],
			task: null
		}
	},
	components: {
		Task
	},
	methods: {
		add() {
			var task = {
				id: guid(),
				name : this.task,
				completed: false
			}
			this.tasks.push(task);   // 将任务添加到列表中，默认为 false
			this.task = null;  // 清空任务框
		},
		done(id) {
			this.tasks = this.tasks.map(function(task){
				if(task.id === id){
					task.completed = true;
				}

				return task;
			})
		}
	}
}
</script>
```

在 App 中使用 - `/src/App.vue`

```vue
<template>
  <div id="app">
    <h1> 任务列表</h1>
    <task-list></task-list> 
  </div>
</template>

<script>
import TaskList from './components/TaskList.vue'

export default {
  name: 'app',
  components: {
    TaskList,
  }
}

</script>
```

最后，查看效果

```bash
$ npm run serve
```