---
MVVM模型和MVC模型了解
---

[toc]



# MVVM模型

> Model-View-ViewModel的缩写,对View和ViewModel的双向数据绑定,使得ViewModel的状态改变可以自动传递给View.即数据双向绑定.



## **Model**

**Model:数据层,代表的是模型、数据,可以在Model层中定义数据修改和操作的业务逻辑.**

它仅仅关注数据本身,不关心任何行为(格式化数据由View负责) 可以理解为一个json的数据对象,主要存放页面中的数据



## **View**

**view:代表的是视图、模板,负责将数据模型转化为UI展现出来**

所看到的页面,和**MVC/MVP**不同的是,MVVM中的View通过使用模板语法来声明式的将数据渲染进DOM,当ViewModel对Model进行更新的时候,会通过数据更新到View.

## **ViewModel**

MVVM模式的核心,它是连接view和model的桥梁.2个方向

1. 将Model转化为View,即将后端传递的数据转化为所看到的页面.实现方式:==数据绑定==
2. 将View转化为Model,即将所看到的页面转化成后端的数据.实现方式:==DOM事件监听==

![mvvm](http://qiliu.luxiaobai.cn/img/mvvm.png)

这两个方向都实现的,我们称之为**数据的双向绑定.**

ViewModel通常要实现一个**observer观察者**,当数据发生变化,ViewModel能够监听到数据的这种变化,然后通知到对应的视图做自动更新,而当用户操作视图,ViewModel也能监听到视图的变化,然后通知数据做改动,这实际上就实现了数据的双向绑定.

ViewModel通过双向数据绑定将View层和Model层连接起来,使得View层和Model层的同步工作完全是自动的.因此开发者只需要关注业务逻辑,无需手动操作DOM,复杂的数据状态维护交给MVVM统一来管理.



## **Vue.js中MVVM的体现**

![mvvm2](http://qiliu.luxiaobai.cn/img/mvvm2.png)

Vue.js的实现方式，对数据（Model）进行"劫持"，当数据变动时，数据会出发劫持时绑定的方法，对视图进行更新.

![mvvm3](http://qiliu.luxiaobai.cn/img/mvvm3.png)

ViewModel通过双向数据绑定把View层和Model层连接了起来,而View和Model之间的同步工作完全是自动的,无需人为干预.





# **MVC模型**

> Model-View-Controller(模型-视图-控制器)模式.这种模式用于应用程序的分层开发

## **Model(模型)**

应用程序中用于处理应用程序逻辑的部分.通常模型对象负责在数据库中存放数据.

Model定义了这个模块的数据模型.在代码中体现为数据管理者,Model负责对数据进行获取及存放.

Model既是数据管理者，也由它来负责获取数据，数据不可能凭空生成的，要么是从服务器上面获取到的数据，要么是本地数据库中的数据，也有可能是用户在UI上填写的表单即将上传到服务器上面存放，所以需要有数据来源。既然Model是数据管理者，则自然由它来负责获取数据.

```javascript
var myapp = {}; // 创建这个应用对象

myapp.Model = function() {
    var val = 0;

    this.add = function(v) {
        if (val < 100) val += v;
    };

    this.sub = function(v) {
        if (val > 0) val -= v;
    };

    this.getVal = function() {
        return val;
    };

    ／* 观察者模式 *／
    var self = this, 
        views = [];

    this.register = function(view) {
        views.push(view);
    };

    this.notify = function() {
        for(var i = 0; i < views.length; i++) {
            views[i].render(self);
        }
    };
};
```

Model和View之间使用了观察者模式，View事先在此Model上注册，进而观察Model，以便更新在Model上发生改变的数据。

## **View(视图)**

是应用程序中处理数据显示的部分。通常视图是依据模型数据创建的

View，视图，简单来说，就是我们在界面上看见的一切。

view和controller之间使用了策略模式，这里View引入了Controller的实例来实现特定的响应策略，比如这个事例中按钮的 click 事件：

```javascript
myapp.View = function(controller){
    var $num = $('#num'),
        $incBtn = $('#increase');
        $decBtn = $('#decrease');
    
    this.render = function(model){
        $num.text(model.getVal() + 'rmb')
    };
    
    /* .  绑定事件 */
    $incBtn.click(controller.increase);
    $decBtn.click(controller.decrease);                
};
```

要实现不同的响应的策略只要用不同的Controller实例替换即可

## **Controller(控制器)**

是应用程序中处理用户交互的部分。通常控制器负责从视图读取数据，控制用户输入，并向模型发送数据

Controller是MVC中的数据和视图的协调者，也就是在Controller里面把Model的数据赋值给View来显示（或者是View接收用户输入的数据然后由Controller把这些数据传给Model来保存到本地或者上传到服务器）

```javascript
myapp.Controller = function(){
            var model = null,
                view = null;

            this.init = function(){
                /* 初始化Model和View */
                model = new myapp.Model();
                view = new myapp.View(this);

                /*View 向Model注册, 当Model更新就会去通知View */ 
                model.register(view);
                model.notify();
            };

            /* 让Model更新数值并通知View更新视图*/ 
            this.increase = function(){
                model.add(1);
                model.notify();
            };

            this.decrease = function(){
                model.sub(1);
                model.notify();
            }
        }
```

执行应用的时候，使用Controller做初始化：

```javascript
 (function () {
                var controller = new myapp.Controller();
                controller.init();
            })();
```



## **MVC中的通讯**

各部分之间的通信都是单向的

![MVC](http://qiliu.luxiaobai.cn/img/MVC.png)

MVC模式的业务逻辑主要集中在Controller，而前端的View其实已经具备了独立处理用户事件的能力，当每个事件都流经Controller时，这层会变得十分臃肿。而且MVC中View和Controller一般是一一对应的，捆绑起来表示一个组件，视图与控制器间的过于紧密的连接让Controller的复用性成了问题

