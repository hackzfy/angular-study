# 路由守卫 guard

## 在 angular 应用中，要实现对页面的访问控制，就需要用到 `guard`。

> 路由守卫是一个很贴切的解释。如果把每个路由（route）当作一条到达页面的通道，那`gurad`就是站在通道前的守卫。只有守卫放行了，你才能穿过通道，到达目的地。

### 路由守卫的分类

- `CanActivate` 是否可以进入当前路由
- `CanActivateChild` 是否可以进入当前路由的子路由
- `CanDeactivate` 是否可以离开当前路由
- `Resolve` 在进入路由前准备数据
- `CanLoad` 是否可以进入懒加载的路由（lazyload module）

而要想实现一个路由守卫，并不复杂。
- 首先你要建立一个路由守卫。
- 然后你要将这个路由守卫添加到privoders数组中。应用中任何需要使用的服务，都需要添加到相应模块的providers数组中。注意，不要将同一个service放入不同的模块中，当然，这是另一个话题。
- 最后，你要将这个路由守卫放到对应的路由上去。


### `CanActivate` 是否可以进入当前路由

1. 建立路由守卫

```js
import { Injectable }     from '@angular/core';
import { CanActivate }    from '@angular/router';

@Injectable()
export class AuthGuard implements CanActivate {
  canActivate() {
    console.log('AuthGuard#canActivate called');
    return true;
  }
}
```
2. 将路由守卫添加到需要的路由
```js
import { AuthGuard }                from './auth-guard.service';

const adminRoutes: Routes = [
  {
    path: 'admin',
    component: AdminComponent,
    canActivate: [AuthGuard], // ---> 添加到 canActivate 数组,可以添加多个守卫
    children: [
      {
        path: '',
        children: [
          { path: 'crises', component: ManageCrisesComponent },
          { path: 'heroes', component: ManageHeroesComponent },
          { path: '', component: AdminDashboardComponent }
        ],
      }
    ]
  }
];
```
3. 将守卫添加到对应模块的 `providers` 中

```js
@NgModule({
  imports: [
    RouterModule.forChild(adminRoutes)
  ],
  exports: [
    RouterModule
  ],
  
  providers:[AuthGuard] // ---> service 必须添加到providers数组中，才能被注入使用
})
export class AdminRoutingModule {}
```

就这么简单。路由守卫的CanActivate 方法返回true，就表示放行。返回false，你就别想过去了。如果有多个路由守卫，得所有人都点头（都返回true），你才能进去！

> `CanActivateChild` 子路由守卫。如果你进入的路由还拥有子路由（children），那么当你进入其任何一个子路由的时候，还需要问子路由守卫同不同意。

实现方法和上面如出一辙，只是换了一个接口。
1. 创建路由守卫
```js
import { Injectable }     from '@angular/core';
import { CanActivateChild }    from '@angular/router'; //---> 接口改变

@Injectable()
export class AuthGuard implements CanActivateChild { //---> 接口改变
  canActivateChild() {  //---> 方法改变
    console.log('AuthGuard#canActivateChild called');
    return true;
  }
}
```
2. 添加到路由
```js
import { AuthGuard }                from './auth-guard.service';

const adminRoutes: Routes = [
  {
    path: 'admin',
    component: AdminComponent,
    canActivateChild: [AuthGuard], // ---> 添加到 canActivateChild 数组,可以添加多个守卫
    children: [
      {
        path: '',
        children: [
          { path: 'crises', component: ManageCrisesComponent },
          { path: 'heroes', component: ManageHeroesComponent },
          { path: '', component: AdminDashboardComponent }
        ],
      }
    ]
  }
];
```
3. 守卫添加到对应模块的`providers`中
```js
@NgModule({
  imports: [
    RouterModule.forChild(adminRoutes)
  ],
  exports: [
    RouterModule
  ],
  
  providers:[AuthGuard] // ---> service 必须添加到providers数组中，才能被注入使用
})
```
**注意：一个路由守卫可以实现多个接口，这样就可以被放置在路由守卫的不同位置。**

```js
@Injectable()
export class AuthGuard implements CanActivate, CanActivateChild {
  constructor(private authService: AuthService, private router: Router) {}

  canActivate(route: ActivatedRouteSnapshot, state: RouterStateSnapshot): boolean {
    let url: string = state.url;

    return this.checkLogin(url);
  }

  canActivateChild(route: ActivatedRouteSnapshot, state: RouterStateSnapshot): boolean {
    return this.canActivate(route, state);
  }

```
- 放置在路由的不同位置，作为不同性质的守卫实用。

```js
import { AuthGuard }                from './auth-guard.service';

const adminRoutes: Routes = [
  {
    path: 'admin',
    component: AdminComponent,
    canActivate: [AuthGuard], // ---> 添加到 canActivate 数组
    canActivateChild: [AuthGuard], // ---> 添加到 canActivateChild 数组
    children: [
      {
        path: '',
        children: [
          { path: 'crises', component: ManageCrisesComponent },
          { path: 'heroes', component: ManageHeroesComponent },
          { path: '', component: AdminDashboardComponent }
        ],
      }
    ]
  }
];
```
这样，一个守卫只要实现了对应的接口，就可以拥有对应接口的守卫能力。


