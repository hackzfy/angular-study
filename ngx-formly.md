# ngx-formly（入门）

表单应该算是 Angular 中最常用也是最重要的部分。但是不管用什么 ui 框架，写起表单来都挺繁琐。 ngx-formly 应运而生。它主要解决了两个问题：

- 繁琐的模板书写工作（搬砖）
- 灵活可配置的表单选项。

首先，我们需要安装 ngx-formly：

```shell
npm install --save @ngx-formly/core  @ngx-formly/bootstrap
```

然后在应用中引入：

```ts
import { NgModule } from '@angular/core';
import { ReactiveFormsModule } from '@angular/forms';
import { BrowserModule } from '@angular/platform-browser';
import { FormlyBootstrapModule } from '@ngx-formly/bootstrap';
import { FormlyModule } from '@ngx-formly/core';
import { AppComponent } from './app.component';

@NgModule({
  declarations: [AppComponent],
  imports: [
    BrowserModule,
    ReactiveFormsModule,  // ngx-formly 依赖 ReactiveFormsModule
    FormlyModule.forRoot(), // ngx-formly 模块
    FormlyBootstrapModule,  // 这个也需要引入，它定义了基本的 type
  ],
  bootstrap: [AppComponent]
})
export class AppModule {}

```

我们看一下使用了 ngx-formly 的表单模板有多简洁：

```html
 <form [formGroup]="form" (ngSubmit)="submit(model)">
      <formly-form [form]="form" [fields]="fields" [model]="model">
        <button>Submit</button>
      </formly-form>
</form>
```

要点有三个：

- formly-form 这个组件。
- 组件的 form 属性，对应当前所在的 FormGroup 对象。
- 组件的 fields 属性，这是一个配置对象。
- 组件的 model 属性，相当于 form.value。

我们看下 fields 属性的值

```ts
fields: FormlyFieldConfig[] = [
    {
      key: 'username',   // FormGroup 对象中的 key
      type: 'input',   // 表单元素的类型
      templateOptions: {
        label: 'username,  // label 显示的内容
        required: true  // 必填
      }
    },
    {
      key: 'password',
      type: 'input',
      templateOptions: {
        label: '用户名',
        type: 'password',  // input 有多种类型，这里可以指定子类型
        required: true 
      }
    }
];
```

model 属性的值：

```ts
model = {
    username: 'abc',
    passwrod: '123456'
};
```

本文只是一个简单的介绍。具体情况可以在 github 上查看。

https://github.com/formly-js/ngx-formly

