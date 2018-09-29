# Angular 动态创建表单

在日常开发中，表单模板的书写工作乏味而繁冗。如何减少这部分工作量？

表单元素无非就那么几种。

```html
<form [formGroup]="form">
    <input type="text" formControlName="username">
    <input type="password" formControlName="password">
    <select formControlName="area">
        <option value="100000">北京</option>
    </select>
	<input type="radio" formControlName="gender" value="male">男	
    <input type="radio" formControlName="gender" value="female">女
	<button>提交</button>
</form>
<pre>{{form.value|json}}</pre>
```

这个表单只有很少的字段，但是工作中遇到的可不是这么简单，写起来很累。

我们可以先看下对应的 formGroup 对象：

```ts
export class DyFormComponent implements OnInit {
  form: FormGroup;
  constructor(private fb: FormBuilder) {}

  ngOnInit() {
    this.form = this.fb.group({
      username: '',
      password: '',
      area: '',
      gender: ''
    });
  }
}
```

可以看到，要实例化一个 formGroup ，我们需要一个对象：

```ts
{
      username: '',
      password: '',
      area: '',
      gender: ''
}
```

要动态创建表单模板，我们需要这样一个格式：

```ts
export interface FormData{
    name: string;
    element: string;
    label: string;
    value: string;
    options: {label:string,value:string}[];
}
export type FormDataFormat = FormData[];
this.data: FormDataFormat = [
  {
    name: 'username',
    element: 'input-text',
    label: '用户名',
    value: '老王'
  },
  {
    name: 'password',
    element: 'input-password',
    label: '密码',
    value: '123456'
  },
  {
    name: 'area',
    element: 'select',
    label: '地区',
    value: '100000',
    options: [{ label: '北京', value: '100000' }, { label: '上海', value: '200000' }]
  },
  {
    name: 'gender',
    element: 'radio',
    label: '性别',
    value: 'male',
    options: [{ label: '男', value: 'male' }, { label: '女', value: 'female' }]
  }
];
```

利用这个数据生成实例化 FormGroup 所需要的参数

```html
generateFormGroupParam() {
    return this.data.reduce((acc, curr) => {
      const { name, value } = curr;
      acc[name] = new FormControl(value);
      return acc;
    }, {});
}
```

同样的利用 data 来循环出表单模板

```html
<form [formGroup]="dyForm">
  <ng-container *ngFor="let item of data">
    <ng-container [ngSwitch]="item.element">
      <ng-container *ngSwitchCase="'input-text'">
        <input type="text" [formControlName]="item.name">
      </ng-container>
      <ng-container *ngSwitchCase="'input-password'">
        <input type="password" [formControlName]="item.name">
      </ng-container>
      <ng-container *ngSwitchCase="'select'">
        <select [formControlName]="item.name">
          <option *ngFor="let option of item.options" [value]="option.value">{{option.label}}</option>
        </select>
      </ng-container>
      <ng-container *ngSwitchCase="'radio'">
        <ng-container *ngFor="let option of item.options">
          <input type="radio" [formControlName]="item.name" [value]="option.value">{{option.label}}
        </ng-container>
      </ng-container>
    </ng-container>

  </ng-container>
</form>
<pre>{{dyForm.value|json}}</pre>
```

实例化 FormGroup

```ts
ngOnInit() {
    this.dyForm = this.fb.group(this.generateFormGroupParam());
}
```

![image-20180703232010470](/var/folders/vv/0z94v1sn4n9glmw8p_xwy6g80000gn/T/abnerworks.Typora/image-20180703232010470.png)

我们将其封装为一个组件：

```ts
import { Component, OnInit, Input } from '@angular/core';
import { FormGroup } from '@angular/forms';

@Component({
  selector: 'app-simple-form',
  templateUrl: './simple-form.component.html'
})
export class SimpleFormComponent implements OnInit {
  @Input() data: FormDataFormat;
  @Input() group: FormGroup;
  constructor() {}

  ngOnInit() {}
}

```

simple-form.component.html

```html
<ng-container [formGroup]="group">
  <ng-container *ngFor="let item of data">
    <ng-container [ngSwitch]="item.element">
      <ng-container *ngSwitchCase="'input-text'">
        <input type="text" [formControlName]="item.name">
      </ng-container>
      <ng-container *ngSwitchCase="'input-password'">
        <input type="password" [formControlName]="item.name">
      </ng-container>
      <ng-container *ngSwitchCase="'select'">
        <select [formControlName]="item.name">
          <option *ngFor="let option of item.options" [value]="option.value">{{option.label}}</option>
        </select>
      </ng-container>
      <ng-container *ngSwitchCase="'radio'">
        <ng-container *ngFor="let option of item.options">
          <input type="radio" [formControlName]="item.name" [value]="option.value">{{option.label}}
        </ng-container>
      </ng-container>
    </ng-container>
  </ng-container>
</ng-container>
```

在需要的地方使用

```ts
import { Component, OnInit } from '@angular/core';
import { FormBuilder, FormGroup, FormControl } from '@angular/forms';

@Component({
  selector: 'app-dy-form',
  templateUrl: `dy-form.component.html`,
  styles: []
})
export class DyFormComponent implements OnInit {
  form: FormGroup;
  constructor(private fb: FormBuilder) {}
  data:FormDataFormat = data; // 上面贴的 data 数据
  ngOnInit() {
    this.form = this.fb.group(this.generateFormGroupParam());
  }

  generateFormGroupParam() {
    return this.data.reduce((acc, curr) => {
      const { name, value } = curr;
      acc[name] = new FormControl(value);
      return acc;
    }, {});
  }
}
```

dy-form.component.html

```html

<form [formGroup]="form">
  <app-simple-form [group]="form" [data]="data"></app-simple-form>
</form>
<pre>{{form.value|json}}</pre>

```

![屏幕快照 2018-07-03 下午11.37.38](/Users/yang/blog/屏幕快照 2018-07-03 下午11.37.38.png)

这只是一个雏形，可以根据项目需要进行完善，加入更多表单元素，以及表单验证，以后再也不怕大表单了。