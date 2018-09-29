# ngx-formly 自定义组件以及整合 ng-zorro-antd

​	前面我们已经说了 ngx-formly 的基本使用，以及校验规则和错误提示如何配置。这次说一下如何整合自定义组件。

​	ngx-formly 已经将 bootstrap ，material， ionic ，primeNG, Kendo 等 UI 框架整合好了，你可以安装并直接使用。如果你使用的不是以上框架，就需要自己整合。

首先我们还是要提在 app.module.ts 中引入ngx-formly的方式

```ts
FormlyModule.forRoot(config)
```

重点在这个 config ，它的类型定义为

```ts
export interface ConfigOption {
    types?: TypeOption[];  // 自定义组件放在这个数组中
    wrappers?: WrapperOption[]; // 自定义组件的包装容器,后面再说
    validators?: ValidatorOption[]; // 自定义校验器
    validationMessages?: ValidationMessageOption[]; // 自定义错误提示
    manipulators?: ManipulatorOption[];
    extras?: {
        fieldTransform?: any;
        showError?: (field: Field) => boolean;
    };
}
```

重点在 types 和 wrappers 属性的定义。

先说 types, 我们定义一个简单的自定义组件：

```
import { Component, OnInit } from '@angular/core';
import { FieldType } from '@ngx-formly/core';

@Component({
  selector: 'app-text-input',
  template: `
  <input [formControl]="formControl"
         [formlyAttributes]="field">
  `
})
export class TextInputComponent extends FieldType {}
```

- 自定义组件需要继承 FieldType
- 自定义组件模版中的表单元素传入两个属性
  - formControl  当前元素关联的 FormControl 对象
  - formlyAttributes  当前元素对应的 FormlyFieldConfig

这两个属性是从父类继承得来：

```TS
export declare abstract class FieldType extends Field ...
export declare abstract class Field {
    form: FormGroup;
    field: FormlyFieldConfig;
    model: any;
    options: FormlyFormOptions;
    readonly key: string;
    readonly formControl: AbstractControl;
    readonly to: FormlyTemplateOptions;
    readonly showError: boolean;
    readonly id: string;
    readonly formState: any;
}
```

此时，你的自定义 input 已经写好了，只要加入到 config 中就可以使用了。

```ts
export const config = {
  types: [
    { name: 'textInput', component: TextInputComponent },
  ],
};
    
```

在页面中使用它：

```ts
import { Component, OnInit } from '@angular/core';
import { FormGroup } from '@angular/forms';
import { FormlyFieldConfig } from '@ngx-formly/core';

@Component({
  selector: 'app-three',
  template: `
    <form [formGroup]="form">
      <formly-form [fields]="fields" [form]="form" [model]="model">
      </formly-form>
    </form>
  `,
  styleUrls: ['./three.component.less']
})
export class ThreeComponent {

  model = { username: 'abc' };
  form = new FormGroup({});
  fields: FormlyFieldConfig[] = [{
      key: 'username',
      type: 'textInput',  // 注意这里的 type ，需要和配置中的 name 属性对应
      templateOptions: {
        label: '用户名'
      }
    }]
}；

```

如果使用了 ng-zorro-antd ，只需要在 Input 中加入 nz-input 就可以了，不过要记得安装  ng-zorro-antd 包，并将样式表引入项目中。



```ts

import { Component, OnInit } from '@angular/core';
import { FieldType } from '@ngx-formly/core';

@Component({
  selector: 'app-text-input',
  template: `
  <input nz-input
       [formControl]="formControl"
       [formlyAttributes]="field">
  `
})
export class TextInputComponent extends FieldType {}

```



但是这个表单元素如何显示 label 之类的元素呢，这就需要用到 wrappers 了。下一次讲如何使用 wrappers。等不及的同学可以自己看一下官方文档，有很详细的介绍。

官方文档传送门：https://formly-js.github.io/ngx-formly/