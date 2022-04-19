## Angular笔记以及日常问题，每日学习更新
## 一. Route的概述
在单页面应用中，你可以通过显示或隐藏特定组件的显示部分来改变用户能看到的内容，而不用去服务器获取新页面。当用户执行应用任务时，他们要在你预定义的不同视图之间移动。要想在应用的单个页面中实现这种导航，你可以使用 Angular 的Router。

为了处理从一个视图到下一个视图之间的导航，你可以使用 Angular 的路由器。路由器会把浏览器 URL 解释成改变视图的操作指南，以完成导航。

要使用Routes，在程序中定义一个route对象：

1 const routes: Routes = [
2   { path: 'first-component', component: FirstComponent },
3   { path: 'second-component', component: SecondComponent },
4 ];
作为根目录Import到Component中


1 @NgModule({
2   imports: [
3     RouterModule.forRoot(appRoutes)
4   ],  
5   exports: [RouterModule]
6 })
7 export class AppRoutingModule {
8 
9 }

Route支持依赖注入，在Component中可以注入Route对象。

值得注意，注入的router并不知道自己处于哪个层级。通过注入ActivatedRoute可以查询


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

 

二. routerLink
在html template中，要实现通过点击实现Route间的跳转，用hRef就可以。

但是，这种方式实现的时候会刷新整个页面，网页中的活动数据并不会保存，而且耗费资源。

<a hRef="/">Home</a>
<a hRef="/servers">Servers</a>
<a hRef="/users">Users</a>
正确则是用routerLink实现跳转

routerLink支持绝对寻址和相对寻址，在路径中加'/'或不加


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

 

三. 动态Route
通常，当用户导航你的应用时，你会希望把信息从一个组件传递到另一个组件。例如，考虑一个显示杂货商品购物清单的应用。列表中的每一项都有一个唯一的 id。要想编辑某个项目，用户需要单击“编辑”按钮，打开一个 EditGroceryItem 组件。你希望该组件得到该商品的 id，以便它能向用户显示正确的信息。

你也可以使用一个路由把这种类型的信息传给你的应用组件。要做到这一点，你可以使用 ActivatedRoute 接口。

要从路由中获取信息：

1. 把 ActivatedRoute 和 ParamMap 导入你的组件。

import { Router, ActivatedRoute, ParamMap } from '@angular/router';
2.  通过把 ActivatedRoute 的一个实例添加到你的应用的构造函数中来注入它：

constructor(
  private route: ActivatedRoute,
) {}
3. 更新 ngOnInit() 方法来访问这个 ActivatedRoute 并跟踪 id 参数：

ngOnInit() {
  this.route.queryParams.subscribe(params => {
    this.name = params['name'];
  });
}
注意：前面的例子使用了一个变量 name，并根据 name 参数给它赋值。

 

四. Route层级
要实现这样的页面：

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

还需要一个通配页面，用来定向到Not found Page，Route的结构也要层级设计

children 用来实现route的子集
复制代码
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

RedirectTo实现重定向

 

五. Route Guard
使用路由守卫来防止用户未经授权就导航到应用的某些部分。Angular 中提供了以下路由守卫：

CanActivate
CanActivateChild
CanDeactivate
Resolve
CanLoad
要想使用路由守卫，可以考虑使用无组件路由，因为这对于保护子路由很方便。

内容过多，参考：https://angular.cn/guide/router-tutorial-toh#milestone-5-route-guards

 

六. Route间的数据传递
路由中的 data 属性是存放与该特定路由关联的任意数据的地方。每个激活的路由都可以访问 data 属性。可以用它来存储页面标题，文本和其它只读静态数据等项目。你可以尝试使用resolve来检索动态数据。

示例：

定义Resolver：

复制代码
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

注册app：

providers: [ServersService, AuthService, AuthGuard, CanDeactivateGuard, ServerResolver]
route中添加resolve属性：

children: [
      { path: ":id", component: ServerComponent, resolve: {server: ServerResolver} },
      { path: ":id/edit", component: EditServerComponent, canDeactivate: [CanDeactivateGuard]},
]
