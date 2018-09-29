# 路由守卫 guard

### 在 angular 应用中，要实现对页面的访问控制，就需要用到 `guard`。

> 路由守卫是一个很贴切的解释。如果把每个路由（route）当作一条到达页面的通道，那`gurad`就是站在通道前的守卫。只有守卫放行了，你才能穿过通道，到达目的地。

#### 路由守卫的分类

- `CanActivate` 是否可以进入当前路由
- `CanActivateChild` 是否可以进入当前路由的子路由
- `CanDeactivate` 是否可以离开当前路由
- `Resolve` 在进入路由前准备数据
- `CanLoad` 是否可以进入懒加载的路由（lazyload module）

而要想实现一个路由守卫，并不复杂。

- 首先你要建立一个路由守卫。
- 然后你要将这个路由守卫添加到privoders数组中。应用中任何需要使用的服务，都需要添加到相应模块的providers数组中。注意，不要将同一个service放入不同的模块中，当然，这是另一个话题。
- 最后，你要将这个路由守卫放到对应的路由上去。

#### `CanActivate` 是否可以进入当前路由

> 建立路由守卫

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

> 将路由守卫添加到需要的路由

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

> 将守卫添加到对应模块的 `providers` 中

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



#### `CanActivateChild` 子路由守卫。

> 如果你进入的路由还拥有子路由（children），那么当你进入其任何一个子路由的时候，还需要问子路由守卫同不同意。

实现方法和上面如出一辙，只是换了一个接口。

> 创建路由守卫

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

> 添加到路由

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

> 守卫添加到对应模块的`providers`中

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

> 放置在路由的不同位置，作为不同性质的守卫实用。

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

#### CanDeactive

> 使用场景: 当用户要离开当前页面时,提醒用户是否要保存在当前页面所做的修改.

> 建立 guard

```js
// can-deactivate-guard.service.ts
import { Injectable }    from '@angular/core';
import { CanDeactivate } from '@angular/router';
import { Observable }    from 'rxjs/Observable';

export interface CanComponentDeactivate {
 canDeactivate: () => Observable<boolean> | Promise<boolean> | boolean;
}

@Injectable()
export class CanDeactivateGuard implements CanDeactivate<CanComponentDeactivate> {
  canDeactivate(component: CanComponentDeactivate) {
    return component.canDeactivate ? component.canDeactivate() : true;
  }
}
```



```js
// crisis-detail.component.ts
// ...
 canDeactivate(): Observable<boolean> | boolean {
    // Allow synchronous navigation (`true`) if no crisis or the crisis is unchanged
    if (!this.crisis || this.crisis.name === this.editName) {
      return true;
    }
    // Otherwise ask the user with the dialog service and return its
    // observable which resolves to true or false when the user decides
    return this.dialogService.confirm('Discard changes?');
  }
// ...
```



> 将guard添加到对应路由.

```js
const routes:Routes = [
    {
        path: ':id',
        component: CrisisDetailComponent,
        canDeactivate: [CanDeactivateGuard]
    },
]
```



> 将guard添加到对应的路由模块的proviaders

```js
@NgModule({
  imports: [
    RouterModule.forRoot(
      appRoutes,
    )
  ],
  exports: [
    RouterModule
  ],
  providers: [
    CanDeactivateGuard
  ]
})
```



#### `CanLoad`

> 如果一个模块时懒加载的,要使用`CanLoad`守卫来控制模块是否可以被加载.这在一些需要权限才可以访问的页面非常有用.如果守卫不同意,受保护模块的代码就不会被加载到本地.



> 创建守卫,把守卫添加到对应路由,以及路由模块中的providers中.上面已经重复和很多遍.

```js
// xxx-routing.module.ts
const routes:Routes = [
  //...
  {
      path: 'admin',
      loadChildren: 'app/admin/admin.module#AdminModule',
      canLoad: [AuthGuard]
  },
  // ...
];

@NgModule({
  // ...
  providers:[AuthGuard]
  // ...
})

// auth-guard.ts

export class AuthGuard implements CanLoad {

  // ...
  canLoad(route: Route): boolean {
    let url = `/${route.path}`;

    return this.checkLogin(url);
  }
  
  // ...
}

```



####`resolver`

> 如果需要在路由跳转到目标页面之前，提前获取页面需要的数据，以便在页面展示时能第一时间显示出对应的数据，就需要使用 resolve。

> 建立 resolver

```
@Injectable()
export class CrisisDetailResolver implements Resolve<Crisis> {
  constructor(private cs: CrisisService, private router: Router) {}

  resolve(route: ActivatedRouteSnapshot, state: RouterStateSnapshot): Observable<Crisis> {
    let id = route.paramMap.get('id');

    return this.cs.getCrisis(id).take(1).map(crisis => {
      if (crisis) {
        return crisis;
      } else { // id not found
      	// 在 resolver 中也可以重定向
        this.router.navigate(['/crisis-center']);
        // 记得当没有获取数据的时候也
        return null;
      }
    });
  }
}
```

> 将`resolver`添加到对应的路由：

```
const routes:Routes = [
         {
            path: ':id',
            component: CrisisDetailComponent,
            // 整个 resolver 的数据会在被设置到ActivatedRoute的data属性中
            // 路由中的 resolve 可以配置多个 resolver
            // resolver将返回的数据存储在data对应的字段中(这里是 crisis)
            resolve: {
              // 然后在组件中注入的 ActivatedRoute 对象中,
              // 通过 data的对应字段名称获取对应的数据.
              crisis: CrisisDetailResolver
            }
         },
];

```

> 将`resolver`加入路由模块的`providers`数组中

```
@NgModule({
  ...
  providers:[CrisisDetailResolver]
})
```

> 在 component 中使用

```js
...
constructor(private route:ActivatedRoute){}
ngOnInit() {
  // 从ActivatedRoute对象的data属性中获取需要的数据
  this.route.data
    .subscribe((data: { crisis: Crisis }) => {
      this.editName = data.crisis.name;
      this.crisis = data.crisis;
    });
}
...
```

**注意** resolver 返回的Observable必须complete,否则路由会一直处于等待状态. 具体请查看Rxjs关于Observable的介绍.





文中的代码引用自Angular官方网站 https://angular.io/guide/router





###  