# Angular 应用实现 page-loading 效果

> 如果在angular中实现了懒加载，那么在用户点击链接，到页面显示出来，是需要一小段时间来加载模块的。同样，在用户点击页面获取数据的过程中，如果网络较慢，页面也需要一段时间之后，才能显示出返回的数据。在这个时间段中，用户可能认为是网页停止响应了，于是再次点击。再次点击会引发很多问题，为了防止这样的情况发生，我们可以简单的告诉用户：页面正在加载，请稍后。那么，如何实现呢？



> 首先，需要一个组件，来告诉用户，页面正在加载。

```js
// page-loading.component.ts

NgModule({
    selector:'app-page-loading',
    template: `
    	<p>
    		页面正在加载...
    	</p>
    `,
    styles:[
    `:host{
            display: flex;
            position: fixed;
            width: 100%;
            height: 100%;
            justify-content: center;
            align-items: center;
            background: rgba(0,0,0,.3);
            z-index: 999;
        }`
    ]
})
export class PageLoadingComponent {}
```

>可以看出，这就是一个简单的，把窗口全部遮住的遮罩层，中间有一行文字，告诉用户页面正在加载。此时，用户无法点击页面了（用户也许会很郁闷，因为他还想不停的点下去，如何提升用户体验是另一个话题）。你可以把提示文字换成任何想展现的东西，比如一个图片，logo，动画等等。



> 其次，这个东西放在哪里呢？如果要在整个应用中可用，则必须放在根组件中。



```html
// app.component.html

<app-page-loading></app-page-loading>
```

>打开页面后，你会发现它遮住了整个窗口，无法进行任何操作。我们是希望它在路由跳转的那短暂的页面空白期出现，而不是一直存在。

```html
<app-page-loading *ngIf="isLoading$|async"></app-page-loading>
```

```js
// app.component.ts

//...
export class AppComponent {
    isLoading$:Observable<boolean>;

	constructor(private pageLoadingService: PageLoadingService){}
    ngOnInit(){
        this.isLoading$ = this.pageLoadingService.loading$;
    }
}
```

>是的，我们需要一个service，由它来传递页面是否正在跳转的状态。注意，这个状态是一个包含着布尔值的Observable,所以我们在模板中需要使用 async pipe来订阅并且获取它的值。下面来实现这个service：

```js
import {Injectable} from '@angular/core';
import {NavigationCancel, NavigationEnd, NavigationError, NavigationStart, Router} from '@angular/router';
import 'rxjs/add/operator/do';
import {BehaviorSubject} from 'rxjs/BehaviorSubject';

@Injectable()
export class LoadingService {

  loading$ = new BehaviorSubject<boolean>(false);

  constructor(private readonly router: Router) {
    this.router.events.subscribe(
      event => this.navigationHandler(event),
    );
  }

  loading() {
    this.loading$.next(true);
  }

  loaded() {
    this.loading$.next(false);
  }


  navigationHandler(event: any) {
    if (event instanceof NavigationStart) {
      this.loading();
      return;
    }
    if (
      event instanceof NavigationEnd ||
      event instanceof NavigationCancel ||
      event instanceof NavigationError
    ) {
      this.loaded();
      return;
    }
  }


}

```

>为了防止写错，我把整个文件copy过来了，这可能有点多，但是仍然很清晰。

>听着，要使他正常工作，别忘了把他加入到根模块的providers数组中。

>不要试图在url中切换地址栏来查看效果，那是页面重新加载，而不是懒加载。一定要通过routerLink来切换页面，才能看到效果。

```html
// app.component.html
<button  routerLink="/user">user</button>
<button  routerLink="/home">home</button>
```



```js
// app-routing.module.ts
const routes:Routes = [
    {
        path:'home',
        loadChildren:'app/home/home.module#HomeModule'},
    {
         path: 'user',
         loadChildren: 'app/user/user.module#UserModule'
    },
]
```



>可能，因为你的代码都在本地，所以遮罩层的存在时间很短，会一闪而过。你可以试着打开开发者工具，在 Network 面板找到 Online，点击右边的向下的小箭头，选择 slow 3G 网络模式，查看效果。

