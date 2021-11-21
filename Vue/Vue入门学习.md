---
Vue入门学习
---

[toc]

# 简介

Vue (读音 /vjuː/，类似于 **view**) 是一套用于构建用户界面的**渐进式框架**。与其它大型框架不同的是，Vue 被设计为可以自底向上逐层应用。Vue 的核心库只关注视图层，不仅易于上手，还便于与第三方库或既有项目整合。



# MVVM模型

> Model-View-ViewModel的缩写,对View和ViewModel的双向数据绑定,使得ViewModel的状态改变可以自动传递给View.即数据双向绑定.



## **Model**

**Model:数据层,代表的是模型、数据,可以在Model层中定义数据修改和操作的业务逻辑.**

它仅仅关注数据本身,不关心任何行为(格式化数据由View负责) 可以理解为一个json的数据对象,主要存放页面中的数据

## **View**

**view:代表的是视图、模板,负责将数据模型转化为UI展现出来**

所看到的页面,和MVC/MVP不同的是,MVVM中的View通过使用模板语法来声明式的将数据渲染进DOM,当ViewModel对Model进行更新的时候,会通过数据更新到View.

## **ViewModel**

MVVM模式的核心,它是连接view和model的桥梁.2个方向

1. 将Model转化为View,即将后端传递的数据转化为所看到的页面.实现方式:数据绑定
2. 将View转化为Model,即将所看到的页面转化成后端的数据.实现方式:DOM事件监听

![mvvm](/Users/lushengyang/Desktop/LSY/StudeyNotes/image/mvvm.png)

这两个方向都实现的,我们称之为**数据的双向绑定.**

ViewModel通常要实现一个observer观察者,当数据发生变化,ViewModel能够监听到数据的这种变化,然后通知到对应的视图做自动更新,而当用户操作视图,ViewModel也能监听到视图的变化,然后通知数据做改动,这实际上就实现了数据的双向绑定.

ViewModel通过双向数据绑定将View层和Model层连接起来,使得View层和Model层的同步工作完全是自动的.因此开发者只需要关注业务逻辑,无需手动操作DOM,复杂的数据状态维护交给MVVM统一来管理.



## **Vue.js中MVVM的体现**

![mvvm2](/Users/lushengyang/Desktop/LSY/StudeyNotes/image/mvvm2.png)

Vue.js的实现方式，对数据（Model）进行"劫持"，当数据变动时，数据会出发劫持时绑定的方法，对视图进行更新.

![mvvm3](/Users/lushengyang/Desktop/LSY/StudeyNotes/image/mvvm3.png)

ViewModel通过双向数据绑定把View层和Model层连接了起来,而View和Model之间的同步工作完全是自动的,无需人为干预.







# 数据与方法

v-bind:数据绑定

```vue
 <div id="app-2">
        <span v-bind:title="message">
            鼠标悬停几秒钟查看此处动态绑定的提示信息！
        </span>
    </div>
 var app2 = new Vue({
            el: '#app-2',
            data: {
                message: '页面加载于' + new Date().toLocaleString()
            }
        })
```

v-if:判断

```vue
<div id="app-3">
        <span v-if="display">
            看到了
        </span>
    </div>
var app3 = new Vue({
            el: "#app-3",
            data: {
                display: true
            }
        })
```



v-for: 循环

```vue
 <div id="app-4">
        <ol>
            <li v-for="(todo, index) in todos" :key="index">
                {{todo.text}}
            </li>
        </ol>
    </div>
var app4 = new Vue({
            el: "#app-4",
            data: {
                todos: [{
                        text: '学习 JavaScript'
                    },
                    {
                        text: '学习 Vue'
                    },
                    {
                        text: '整个牛项目'
                    }
                ]
            }
        })
```

v-on: 添加事件监听方法

```vue
 <div id="app-5">
        <p>{{message}}</p>
        <button v-on:click="reverseMessage">反转消息</button>
    </div>
var app5 = new Vue({
            el: "#app-5",
            data: {
                message: "Hello Vue.js!"
            },
            methods: {
                reverseMessage: function () {
                    this.message = this.message.split("").reverse().join('')
                }
            },
        })
```

V-model: 表单和应用状态的双向绑定

```vue
<div id="app-6">
        <p>{{message}}</p>
        <input v-model="message">
    </div>
var app6 = new Vue({
            el: "#app-6",
            data: {
                message: 'Hello Vue!'
            }
        })
```



## Vue实例创建

```vue
<div id="app-7">
        <ol>
            <todo-item 
            v-for="(todo, index) in todoList" :key="index" 
            v-bind:todo="todo" 
            v-bind:key="todo.id"
            ></todo-item>
        </ol>
    </div>
 Vue.component('todo-item', {
            props: ['todo'],
            template: '<li>{{ todo.text }}</li>'
        })

        var app7 = new Vue({
            el: '#app-7',
            data: {
                todoList: [{
                        id: 0,
                        text: '蔬菜'
                    },
                    {
                        id: 1,
                        text: '奶酪'
                    },
                    {
                        id: 2,
                        text: '随便其它什么人吃的东西'
                    }
                ]
            }
        })
```

