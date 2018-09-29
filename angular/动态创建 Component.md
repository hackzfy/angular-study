# 动态创建 Component

> 动态创建Component，其实并不难，是你太悲观～
>
> 我们只需要了解三要素：
>
> 1. ViewContainerRef  动态组件就在这个东西内部创建。
> 2. ComponentFactoryResolver 创建一个ComponentFactory
> 3. ComponentRef 你动态创建了组件之后，对这个组件的引用。



> 下面我们来创建一个动态组件。

```js
import {Component, ComponentFactoryResolver, OnInit, ViewChild, ViewContainerRef} from '@angular/core';
import {HelloComponent} from './hello/hello.component';
import {HiComponent} from './hi/hi.component';
import {LoveComponent} from './love/love.component';
import {AngularComponent} from './angular/angular.component';

@Component({
  selector: 'app-root',
  template:
      `
    <h1> Dynamic Component Demo</h1>
    <template #template></template>
  `
})
export class AppComponent implements OnInit {
   // 注意，第一要素：ViewContainerRef 出现了
  @ViewChild('template', {read: ViewContainerRef}) template: ViewContainerRef;
  componentArr = [
    HelloComponent,   // ng g c hello
    HiComponent,	  // ng g c hi
    LoveComponent,    // ng g c love
    AngularComponent  // ng g c angular
  ];
  index = 0;
  // 注意，第二要素：ComponentFactoryResolver 出现了
  constructor(private componentFactoryResolver: ComponentFactoryResolver) {
  }

  ngOnInit(): void {
    this.loadComponent();
    this.loop();
  }

  loadComponent() {
    // 选择一个 component class
    const component = this.componentArr[this.index++];
    // 你也可以用取余实现，这样比较直观
    if (this.index > this.componentArr.length - 1) {
      this.index = 0;
    }
    // ComponentFactoryResolver 创建 ComponentFactory
    const componentFactory =
      this.componentFactoryResolver
        .resolveComponentFactory(component);

    // 清除容器内部的所有内容，this.template 就是一个 ViewContainerRef
    this.template.clear();
    // 容器需要接收一个 ComponentFactory 来创造组件的实例，返回一个引用。
    // 注意，第三要素：ComponentRef 出现了
    const componentRef = this.template.createComponent(componentFactory);
    // 你可以 通过 instance属性 来访问 componentRef 内部的component实例
    console.log(componentRef.instance);
    // componentRef.instance.title = 'haha' 前提是这个组件有title属性，并且为public
    console.log('abc');
  }

  loop() {
    setInterval(() => this.loadComponent(), 1000);
  }
}

```

> 简单来说，就是我们先要获取一个ViewContainerRef的实例，然后调用它的createComponent 方法来创建组件，但是这个方法需要接收一个 ComponentFactory, 而ComponentFactory 需要通过 ComponentFactoryResolver来创建。
>
> 当你终于将代码写完去运行时，发现控制台报错了，心里开始骂娘。其实你忘了将那四个动态创建的组件加入到 AppModule 的 entryComponents 数组中。

```js
@NgModule({
  declarations: [
    AppComponent,
    DynamicDirective,
    HelloComponent,
    HiComponent,
    LoveComponent,
    AngularComponent

  ],
  imports: [
    BrowserModule
  ],
  entryComponents: [
    HelloComponent,
    HiComponent,
    LoveComponent,
    AngularComponent
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule {
}
```

> 动态组件华丽的运行着，你的心里思考着为什么这东西这么简单。