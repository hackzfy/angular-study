# ng-zorro-antd 项目搭建

1. 全局安装 angular-cli。项目使用版本号为 6

```shell
npm i -g @angular/cli
ng -v

     _                      _                 ____ _     ___
    / \   _ __   __ _ _   _| | __ _ _ __     / ___| |   |_ _|
   / △ \ | '_ \ / _` | | | | |/ _` | '__|   | |   | |    | |
  / ___ \| | | | (_| | |_| | | (_| | |      | |___| |___ | |
 /_/   \_\_| |_|\__, |\__,_|_|\__,_|_|       \____|_____|___|
                |___/
    

Angular CLI: 6.0.0
```

2. 创建项目

```shell
ng new my-project --style less // 使用 less 样式表
```

3. 进入项目，安装 ng-zorro-antd。如果你使用的是 0.7版本，需要安装 rxjs-compat做兼容。

```shell
cd my-project
npm install --save ng-zorro-antd
npm install --save rxjs-compat
```

4. 在项目的 src/styles.less 中引入 ng-zorro-antd 样式表

```less
/* You can add global styles to this file, and also import other style files */
@import "../node_modules/ng-zorro-antd/src/ng-zorro-antd.min.css";
```

5. 打开 src/app/app.module.ts, 引入 ng-zorro-antd 模块。

```typescript
@NgModule({
  declarations: [
    AppComponent,
  ],
  imports: [
    BrowserModule,
    NgZorroAntdModule.forRoot(),
  ],
  bootstrap: [AppComponent],
})
export class AppModule {
}
```

6. 在 AppComponent 中使用组件。

```html
<button nz-button nzType="primary">
    button
</button>
```

应该正常显示蓝色按钮。