#ngx-formly自定义校验规则

 ngx-formly 入门中已经说了如何使用它创建基本的 form。今天说一下如何添加自定义校验规则和错误提示。

首先看一下我们的需求：

要求：

- 创建一个包含两个输入框的 form。一个用来添写用户名，一个用来添写用户密码。
- 用户名必添，不能少于6位，首尾空格不算有效字符。
- 密码必添，不能少于6位，并且当用户名不合格时，处于禁用状态。

先把表单搞起来再说：

```ts
import { Component, OnInit } from '@angular/core';
import { FormGroup, Validators } from '@angular/forms';
import { FormlyFieldConfig } from '@ngx-formly/core';

@Component({
  selector: 'app-one',
  templateUrl: './one.component.html',
  styleUrls: ['./one.component.less']
})
export class OneComponent {
  form = new FormGroup({});
  model = {username: '', password: ''};
  fields: FormlyFieldConfig[] = [
    {
      key: 'username',
      type: 'input',
      templateOptions: {
        label: '用户名'
      },
    },
    {
      key: 'password',
      type: 'input',
      templateOptions: {
        type: 'password',
        label: '密码'
      }
    }
  ];

  submit(model) {
    console.log(model);
  }
}

```

要加入校验功能，需要在引入 FormlyModule 时，传入配置对象：

```TS
FormlyModule.forRoot(config)
```

我们看下 config 对象如何创建：

- config  是一个对象。
- 它的 validators 属性是一个数组，每一项都是一个对象，类型定义为：

```ts
export interface ValidatorOption {
    name: string;
    validation: FieldValidatorFn;
}
```

- 它的 validationMessages 属性也是一个数组，用来配置错误提示信息，每一项都是一个对象，类型定义为：

```ts
export interface ValidationMessageOption {
    name: string;
    message: string | ((error: any, field: FormlyFieldConfig) => string);
}
```

目前我们只需要了解这两项就可以自定义错误消息和校验规则了，配置如下：

```TS
export const config = {
  validators: [
    { name: 'noEmpty', validation: noEmptyValidator }
  ],
  validationMessages: [
    { name: 'noEmpty', message: noEmptyMessage },
    { name: 'minlength', message: minLengthMessage },
  ]
};

// 防止用户输入6个空格
export function noEmptyValidator(control: FormControl) {
  return control.value && control.value.trim().length > 0
    ? null
    : { noEmpty: true };
}

export function noEmptyMessage(err, field: FormlyFieldConfig) {
  return `请添写${field.templateOptions.label}`;
}
// 虽然 angular 内置了 Validators.minlength 这样的校验器，但是错误信息并没有内置，我们需要
// 自己实现，并配置进去
export function minLengthMessage(err, field: FormlyFieldConfig) {
  return `${field.templateOptions.label}不能少于${err.requiredLength}位`;
}
```

 好了，现在将校验规则添加到我们的 Component 中去：

```ts
export class OneComponent {
  form = new FormGroup({});
  model = {username: '', password: ''};
  fields: FormlyFieldConfig[] = [
    {
      key: 'username',
      type: 'input',
      templateOptions: {
        label: '用户名'
      },
      validators: {
        validation: ['noEmpty', Validators.minLength(6)]
      }
    },
    {
      key: 'password',
      type: 'input',
      templateOptions: {
        type: 'password',
        label: '密码'
      },
      validators: {
        validation: ['noEmpty', Validators.minLength(6)]
      },
    }
  ];

  submit(model) {
    console.log(model);
  }
}
```

现在只剩最后一个需求了，在用户名无效时禁用密码输入。这需要在 FormlyFieldConfig 中配置 expressionProperties 属性。 也就是在 password 的配置里加入这一项，完整代码：

```ts
import { Component, OnInit } from '@angular/core';
import { FormGroup, Validators } from '@angular/forms';
import { FormlyFieldConfig } from '@ngx-formly/core';

@Component({
  selector: 'app-one',
  templateUrl: './one.component.html',
  styleUrls: ['./one.component.less']
})
export class OneComponent {
  form = new FormGroup({});
  model = {username: '',password: ''};
  fields: FormlyFieldConfig[] = [
    {
      key: 'username',
      type: 'input',
      templateOptions: {
        label: '用户名'
      },
      validators: {
        validation: ['noEmpty', Validators.minLength(6)]
      }
    },
    {
      key: 'password',
      type: 'input',
      templateOptions: {
        type: 'password',
        label: '密码'
      },
      validators: {
        validation: [Validators.minLength(6), 'noEmpty']
      },
      expressionProperties: {
        'templateOptions.disabled': (model: any, formState: any) => {
          // model 就是 this.model
          return !(model.username && model.username.trim().length >= 6);
        }
      }
    }
  ];

  submit(model) {
    console.log(model);
  }
}

```

ngx-formly  is awesome, 值得深入学习。

