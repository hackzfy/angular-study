#Angular 之 named outlet

>named outlet 顾名思义，就是有名字的 router-outlet。

> 平时我们使用的outlet都是这样的：

```html
<router-outlet></router-outlet>
```

> named outlet 是这样的：

```html
<router-outlet name="popup"></router-outlet>
```

> 多了个name属性。其实，每一个outlet都有name。我们平时没有写name的outlet，它有一个默认name：primary,相当于：

```html
<router-outlet name="primary"></router-outlet>
```

###要这么多outlet有什么用？

> 用处太多了，因为网页中展示的内容不再局限于一个outlet，你可以在网页的不同位置，展示不同的内容。包括但不限于弹出窗口，confirm，还有广告。。。

我想让窗口的右下角出现一个弹窗，里面有四个字：联系客服。重要的是，我需要在任意页面，都可以控制弹窗的出现和隐藏。如何实现？

> 首先，我要在根模板中加入一个named outlet

```html
// app.component.html

<router-outlet></router-outlet>
<router-outlet name="popup"></router-outlet>
```

> 我想让联系客服这个组件出现在 popup 这个outlet中。创建组件：

```js
@NgModule({
    selector:'app-concat',
    template:`<div>联系客服</div>`,
    styles:[
        `div{
			position:fixed;
			width: 200px;
			height: 100px;
			right: 0;
			bottom: 0;
			background: #fff;
		}`
    ]
})
class ConcatComponent{
    
}
```

> 可以想象，这个组件是比较丑的，这是另外一个话题。接下来，我们既然想让他显示在router-outlet中，肯定要配置路由才是。

```js
// app-routing.module.ts

const routes:Routes = [
    //...
    {path:'concat',component:ConcatComponent,outlet:'popup'}
    //...
];
```

> 不要指望手工改变url这个路由就会显示出对应的组件，因为named outlet解析出来的url不一样。先看一下如何添加 link 来指向这个路由。

```html
// app.component.html
<a [routerLink]="[{outlets:{'popup':'concat'}}]">concat</a>
```

> 意思是，要在name为popup的router-outlet上挂载组件，挂载哪个组件？挂载路由定义中 path 为 concat 的路由指定的组件。



> 在组件成功显示出来后，剩下的问题就是如何关闭它了。可以在页面上任意位置，包括ConcatComponent内部建立一个关闭按钮。

```html
<button (click)="closePopup()">
    close
</button>
```

> 对应的方法

```js
closePopUp(){
    this.router.navigate([
        {outlets:{popup:null}}
    ])
}
```

> 是的，让其显示的路由路径为null，就会清空该路由上挂载的所有组件。你可以用这个特性来实现很多东西！