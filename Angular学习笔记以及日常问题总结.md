## Angular笔记以及日常问题，每日学习更新
## 一. Route的概述 
在单页面应用中，可以通过显示或隐藏特定组件的显示部分来改变用户能看到的内容，而不用去服务器获取新页面。当用户执行应用任务时，他们要在你预定义的不同视图之间移动。要想在应用的单个页面中实现这种导航，处理从一个视图到下一个视图之间的导航，你可以使用 Angular 的路由器。路由器会把浏览器 URL 解释成改变视图的操作指南，以完成导航。
在单页面应用中，可以通过显示或隐藏特定组件的显示部分来改变用户能看到的内容，而不用去服务器获取新页面。当用户执行应用任务时，他们要在你预定义的不同视图之间移动。要想在应用的单个页面中实现这种导航，

现在程序中定义一个route对象：
```js
const routes: Routes = [
  { path: 'first-component', component: FirstComponent },
  { path: 'second-component', component: SecondComponent },
 ];
```
作为根目录Import到Component中

```js
1 @NgModule({
2   imports: [
3     RouterModule.forRoot(appRoutes)
4   ],  
5   exports: [RouterModule]
6 })
7 export class AppRoutingModule {
8 
9 }
```

Route支持依赖注入，在Component中可以注入Route对象。

注入的router并不知道自己处于哪个层级。通过注入ActivatedRoute可以查询

```js
 1 import { Component, OnInit } from '@angular/core';
 2 import { Router, ActivatedRoute } from '@angular/router';
 3 
 4 @Component({
 5   selector: 'app-servers',
 6   templateUrl: './servers.component.html',
 7   styleUrls: ['./servers.component.css']
 8 })
 9 export class ServersComponent implements OnInit {
10 
11   constructor(private router: Router,
12               private route: ActivatedRoute) {
13   }
14 
15   ngOnInit() {
16   }
17 
18 }

 ```

## 二. routerLink
在html template中，要实现通过点击实现Route间的跳转，用hRef就可以。

但是，这种方式实现的时候会刷新整个页面，网页中的活动数据并不会保存，而且耗费资源。
```js
<a hRef="/">Home</a>
<a hRef="/servers">Servers</a>
<a hRef="/users">Users</a>
```
正确则是用routerLink实现跳转

routerLink支持绝对寻址和相对寻址，在路径中加'/'或不加

```js
<li role="presentation"
  routerLinkActive="active"
  [routerLinkActiveOptions]="{exact: true}">
<a routerLink="/">Home</a>
</li>
  <li role="presentation"
  routerLinkActive="active">
  <a routerLink="servers">Servers</a>
</li>
<li role="presentation"
  routerLinkActive="active">
  <a [routerLink]="['users']">Users</a>
</li>
```

## 三. 动态Route
通常，当用户导航应用时，会把信息从一个组件传递到另一个组件。比如说，考虑一个显示杂货商品购物清单的应用。列表中的每一项都有一个唯一的 id。要想编辑某个项目，用户需要单击“编辑”按钮，打开一个 EditGroceryItem 组件。肯定希望该组件得到该商品的 id，以便它能向用户显示正确的信息。

就也可以使用一个路由把这种类型的信息传给应用组件。要做到这一点，使用 ActivatedRoute 接口。

要从路由中获取信息：

1. 把 ActivatedRoute 和 ParamMap 导入组件。


import { Router, ActivatedRoute, ParamMap } from '@angular/router';
2.  通过把 ActivatedRoute 的一个实例添加到你的应用的构造函数中来注入它：
```js
constructor(
  private route: ActivatedRoute,
) {}
3. 更新 ngOnInit() 方法来访问这个 ActivatedRoute 并跟踪 id 参数：

ngOnInit() {
  this.route.queryParams.subscribe(params => {
    this.name = params['name'];
  });
}


```
 

## 四. Route层级
要实现这样的页面：
```js
root

　|-Servers

   |    |- Server1

   |    |       |- Server1/id

   |    |       |- Server1/edit

   |    |- Server2

   |            |- Server2/id

   |            |- Server2/edit

　|-Users

        |- User1

        |- User2　　
        ```

还需要一个通配页面，用来定向到Not found Page，Route的结构也要层级设计

children 用来实现route的子集
```js
const appRoutes: Routes = [
  { path: '', component: HomeComponent },
  { path: 'users', component: UsersComponent, children: [
    { path: ':id/:name', component: UserComponent }
  ] },
  {
    path: 'servers',
    children: [
    { path: ':id', component: ServerComponent, resolve: {server: ServerResolver} },
    { path: ':id/edit', component: EditServerComponent] }
  ] },
  { path: 'not-found', component: ErrorPageComponent, data: {message: 'Page not found!'} },
  { path: '**', redirectTo: '/not-found' }
];
```
RedirectTo实现重定向

 

五. Route Guard
使用路由守卫来防止用户未经授权就导航到应用的某些部分。Angular 中提供了以下路由守卫：

CanActivate
CanActivateChild
CanDeactivate
Resolve
CanLoad
要想使用路由守卫，可以考虑使用无组件路由，因为这对于保护子路由很方便。

内容太多了暂时整理不完，参考官方文档：https://angular.cn/guide/router-tutorial-toh#milestone-5-route-guards

 

六. Route间的数据传递 （这里没有手打，去看的示例）
路由中的 data 属性是存放与该特定路由关联的任意数据的地方。每个激活的路由都可以访问 data 属性。可以用它来存储页面标题，文本和其它只读静态数据等项目。你可以尝试使用resolve来检索动态数据。

示例：

定义Resolver：

```js
 1 import { Injectable } from '@angular/core';
 2 import { Resolve, ActivatedRouteSnapshot, RouterStateSnapshot, Router } from '@angular/router';
 3 import { Observable } from 'rxjs/Observable';
 4 import { ServersService } from '../servers.service';
 5 
 6 interface Server {
 7     id: number;
 8     name: string;
 9     status: string;
10 }
11 
12 @Injectable()
13 export class ServerResolver implements Resolve<{id:number, name:string, status:string}> {
14 
15     constructor(private serverService: ServersService) {}
16 
17     resolve(route: ActivatedRouteSnapshot, state: RouterStateSnapshot):
18      Observable<Server> | Promise<Server> | Server {
19         return this.serverService.getServer(route.params['id']);
20     }
21 }
```

注册app：
```js
providers: [ServersService, AuthService, AuthGuard, CanDeactivateGuard, ServerResolver]
route中添加resolve属性：

children: [
      { path: ":id", component: ServerComponent, resolve: {server: ServerResolver} },
      { path: ":id/edit", component: EditServerComponent, canDeactivate: [CanDeactivateGuard]},
]
```
## 构造函数与ngOnInit的区别 day4.19
虽然大概了解两者在使用上的不同，这里我想得到一个比较全面的比较，去查了一些博客，挖掘组件初始化的过程。

## **JS/TS语言相关的区别**
我们先从一个与语言本身有关的最明显的区别开始。ngOnInit只是一个类上的方法，结构上与类上的任何其他方法没有什么不同。 只是Angular团队决定以这种方式命名，但它可能是任何其他名称：
```js
class MyComponent {
  ngOnInit() { }
  otherNameForNgOnInit() { }
}

是否要在组件类上实现该方法，完全取决于你。在编译期Angular编译器检查组件是否实现此方法，并使用适当的标志标记该类：

export const enum NodeFlags {
  ...
  OnInit = 1 << 16,

此标志会在检测变化时用来决定是否调用组件类实例的这个方法：

if (def.flags & NodeFlags.OnInit && ...) {
  componentClassInstance.ngOnInit();
}

构造函数又是一个不同的东西。 无论是否在TypeScript类中实现它，在创建类的实例时仍然会被调用。 这是因为typescript类构造函数被转换成JavaScript构造函数：

class MyComponent {
  constructor() {
    console.log('Hello');
  }
}

转换为

function MyComponent() {
  console.log('Hello');
}

使用new操作符调用此函数来创建此类的实例：

onst componentInstance = new MyComponent(

所以如果你在类里移除了构造函数，它会转换为一个空函数：

class MyComponent { }

转换为空函数

function MyComponent() {}
```
这就是为什么我说构造函数被调用，无论你是否在类上实现它。

组件初始化过程相关的区别
从组件初始化阶段的角度来看，两者之间有很大的区别。 Angular启动过程包括两个主要阶段：


- 构造组件树
- 执行变更检测

当Angular构造组件树时，调用组件的构造函数。 包括ngOnInit的所有生命周期钩子都作为以下变更检测阶段的一部分被调用。 通常，组件初始化逻辑需要有依赖注入（DI）提供程序或可用的输入绑定或渲染的DOM。 这些可用于Angular启动过程的不同阶段。

当Angular构造组件树时，已经配置了根模块注入器，因此可以注入任何全局依赖关系。 此外，当Angular实例化子组件类时，父组件的注入器也已设置，因此可以注入在父组件（包括父组件本身）上定义的提供程序。 组件构造函数是在注入器的上下文中调用的唯一方法，因此如果需要任何依赖关系，这是获取这些依赖关系的唯一位置。 @Input通信机制作为以下变更检测阶段的一部分进行处理，因此输入绑定在构造函数中不可用。

当Angular开始变更检测时，组件树已经构建好，并且已经调用树中所有组件的构造函数。 此外，此时每个组件的模板节点都添加到DOM中。 在这里，你可以使用初始化组件所需的所有数据——DI provider，DOM和输入绑定。

你可以在Angular里你所需要知道关于变更检测的一切读到更多关于变更检测的内容，以及在Angular属性绑定更新机制里了解Angular是如何处理输入的。

我们以一个快速的例子来展示这些阶段。 假设你有以下模板：

<my-app>
   <child-comp [i]='prop'>

所以Angular开始引导应用程序。 如上所述，它首先为每个组件创建类。 所以它调用MyAppComponent构造函数。 执行组件构造函数时，Angular将解析注入到MyAppComponent构造函数中的所有依赖项，并将它们作为参数。 它还创建一个DOM节点，它是my-app组件的宿主元素。然后，它继续为child-comp创建宿主元素，并调用ChildComponent构造函数。在这个阶段Angular不关心输入绑定 i 和任何生命周期钩子。 所以当这个过程完成时Angular以以下的组件视图树结束：
```js
MyAppView
  - MyApp component instance
  - my-app host element data
       ChildComponentView
         - ChildComponent component instance
         - child-comp host element data
```
接着Angular运行更改检测，更新my-app的绑定并调用MyAppComponent实例上的ngOnInit。 然后，它继续更新child-comp的绑定，并在ChildComponent类上调用ngOnInit。

你可以在这里了解更多关于我在上面提到的视图，这就是为什么你不会在Angular中找到组件。

使用上的区别
现在我们来看下在使用方面的区别。

构造函数
Angular类构造函数主要用于注入依赖关系。 Angular称此构造函数注入模式，这里详细说明。 对于更深入的架构理解，您可以阅读MiškoHevery的“构造器注入与Setter注入”。

但是，构造函数的使用不限于DI（依赖注入）。 例如，@angular/router模块的router-outlet指令使用它在路由器生态系统内注册自身及其位置（viewContainerRef）。 我在这就是如何在@ViewChild查询计算之前获取ViewContainerRef 描述了该方法。

然而，通常的做法尽可能少放逻辑到构造函数中。

- NgOnInit
如上所述，Angular在创建组件的DOM，使用构造函数注入所有必须的依赖，以及处理完输入绑定完成后调用ngOnInit。所以这里你有所有必需的信息，使它成为执行初始化逻辑的好地方。

通常的做法是使用ngOnInit来执行初始化逻辑，即使该逻辑不依赖于DI，DOM或输入绑定。

