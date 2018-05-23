# Angular 响应式表单校验	

> 动态表单的验证更加简单。

```typescript
<input
  [formControl]="name">
<div *ngIf="name.invalid && (name.dirty||name.touched)">
  <div *ngIf="name.errors.required">
    name is required
  </div>
  <div *ngIf="name.errors.minlength">
    minlength is 4
  </div>
  <div *ngIf="name.errors.noEmpty">
    name can not be empty string
  </div>
</div>
<div>{{name.errors | json}}</div>
```

> 模板很简洁，只需要给 input 添加一个 `[formControl]="name"` 。这个 name 并不是固定的，你可以自己取名，但要和组件类里的属性名相同。

```typescript
export class DyUserInputComponent implements OnInit {
  name: FormControl;

  constructor() {
  }
    
  ngOnInit(): void {
    this.name = new FormControl(
      'hello', // 初始值
      [Validators.required, Validators.minLength(4)], // 同步校验器
      [] // 异步校验器
    );
  }
}
```

> 这样就完成了。因为 formControl 在 ReactiveFormsModule 中，所以需要在模块中引入 ReactiveFormsModule。
>
> 同样的，当在 input 中输入四个空格时，发现校验居然通过了。很明显我们需要自定义一个校验规则，保证输入不能为空格。这非常简单。你只需要定义一个函数：

```typescript
function noEmpty(c: AbstractControl) {
  const value = c.value;
  return value && value.trim() !== '' ? null : {'noEmpty': true};
}
```

> 这个函数定义要遵循一定的规则：
>
> 一。函数接收一个类型为 AbstractContrl 的参数。
>
> 二。通过参数的 value 属性即可以拿到需要校验的输入。
>
> 三。自定义判断逻辑。但要保证在校验通过时返回null，不通过时返回一个对象。
>
> 四。如果返回了对象，对象的 key 应和校验器名称对应，value 可以为任意值，只要你觉得方便使用就行。
>
> 现在我们把它加入到 input 的校验规则中：

```typescript
...
ngOnInit(): void {
    this.name = new FormControl(
      'hello', // 初始值
      [Validators.required, Validators.minLength(4),noEmpty], // 同步校验器
      [] // 异步校验器
    );
}
...
```

> 现在输入四个空格：

```
{ "noEmpty": true }
```

> 可以发现，比模板表单的自定义校验要简单的多，不需要写 Directive,不需要定义 Providers。如果有问题，欢迎进群讨论。