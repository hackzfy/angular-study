> 表单是现代网页应用构成的重要元素，而其中的难点又是表单校验。
>
> Angular 内置了几个表单校验指令。

```html
<input required minlength="4" />
```

> 如果用户没有输入内容，或输入长度不足4个字符，需要做出错误提示。

```html
<input #name="ngModel" ngModel required minlength="4">
<div *ngIf="name.invalid && (name.dirty||name.touched)">
    input is invalid.
</div>
```

> 上面是一个模板驱动表单，需要注意几点：
>
> 一，要使用 ngModel，需要在模块中引入 FormsModule。
>
> 二，`#name="ngModel"`这行代码是将一个 ngModel 的实例，赋值给一个模板变量。
>
> 三，ngModel 实例有很多属性， invalid属性代表此实例的值是否无效（未通过验证），dirty代表input 是否获取过焦点，touched 代表在移动端是否获取过焦点（你不希望表单在你没碰过的时候就提示错误吧）。
>
> 四，`*ngIf="name.invalid && (name.dirty||name.touched)"`如果此表达式为真，则显示错误提示。
>
> 如果我希望提示更加精确呢？仅仅是一个 `input is invalid`可能用户并不知道哪里错了。

```html
<input #name="ngModel" ngModel required minlength="4">
<div *ngIf="name.invalid && (name.dirty||name.touched)">
     <div *ngIf="name?.errors.required">
        Name is required.
      </div>
      <div *ngIf="name?.errors.minlength">
        Name must be at least 4 characters long.
      </div>
</div>
```

> 是的，就是这样。这里需要注意的是 `name?.errors`这个属性。（？是模板版中的语法，不这样写会报错，在github是仍是一个 open issue）。
>
> 如果在input中输入3个字符，在控制台输出name.errors，会是这样的：

```js
{ "minlength": { "requiredLength": 4, "actualLength": 3 } }
```

> 也就是说，`name?.errors` 是个对象，它的 key 就是未通过校验的规则，value 是一个可能为任意类型的值。为什么这么说？把input里的字符全部删掉，再看一下 错误输出：

```js
{ "required": true}
```

> 现在 key 还是未通过校验的规则，但是value变成了一个布尔类型的值。
>
> Angular 还提供了一些其它的校验规则：

```typescript
class Validators {
  static min(min: number): ValidatorFn
  static max(max: number): ValidatorFn
  static required(control: AbstractControl): ValidationErrors | null
  static requiredTrue(control: AbstractControl): ValidationErrors | null
  static email(control: AbstractControl): ValidationErrors | null
  static minLength(minLength: number): ValidatorFn
  static maxLength(maxLength: number): ValidatorFn
  static pattern(pattern: string | RegExp): ValidatorFn
  static nullValidator(c: AbstractControl): ValidationErrors | null
  static compose(validators: (ValidatorFn | null | undefined)[] | null): ValidatorFn | null
  static composeAsync(validators: (AsyncValidatorFn | null)[]): AsyncValidatorFn | null
}
```

> 即便如此，还是远远满足不了业务需要。就举个小例子：我们在 input 里面输入了四个空格，发现校验居然通过了！

```js
<input required minlength="4" ...
```

> 看来 required 会把空格当作有效输入，minlength 也是一样。既然它们满足不了需求，我们只能自己动手了。
>
> 我们希望最终的结果是这样的：

```
<input required minlength="4" noEmpty ...
```

> 很显然， noEmpty 是一个 Directive。

```js
import {Directive} from '@angular/core';
import {AbstractControl, NG_VALIDATORS, ValidationErrors, Validator} from '@angular/forms';


@Directive({
  selector: '[noEmpty]',
  providers: [{provide: NG_VALIDATORS, useExisting: NoEmptyDirective, multi: true}]
})
export class NoEmptyDirective implements Validator {
  // 这个方法没用到，忽略。但是接口中有定义，又不能不实现，就让它空着吧。
  registerOnValidatorChange(fn: () => void): void {
  }
  // 主要是这个方法。
  // 它会将 input 对应的 ngModel 实例中的 formControl 传入这个方法中进行校验。
  // 虽然有些绕，但事实如此。Angular 在对象的封装上下了很多功夫。
  // formControl 的 value 就是input中你输入的值。
  validate(c: AbstractControl): ValidationErrors | null {
    const value = c.value;
    // 我们仅仅是 去掉了 value 两边的空格。
    // 按照约定，如果校验通过，返回 null,
    // 如果不通过，返回一个对象。
    // 对象的 key 为校验规则，value 为任意类型，你可以定义为你想要的任意格式。
    return value && value.toString().trim() !== '' ? null : {noEmpty: {value}};
  }
}

```

> 需要注意的是 ` providers: [{provide: NG_VALIDATORS, useExisting: NoEmptyDirective, multi: true}]`这一行。
>
> provide 属性对应的是一个 token:`NG_VALIDATORS` 。这个token 是所有 validators 共用的，所以需要 muti:true属性，如果不加会报错。
>
>  `useExisting`表示使用已有的 `NoEmptyDirective`实例。如果使用 `useClass`，则会重新创建一个新的 `NoEmptyDirective`。(这个知识点你要深入学习一下 Angular 的 DI)。
>
> 现在让我们试一下效果，输入四个空格：

```js
null
```

> what?
>
> 哦，忘了把它加到 declarations 数组了。。。

```js
@NgModule({
    declarations:[NoEmptyDirective,...],
    ...
})
```

> 再试一次：

```js
{ "noEmpty": { "value": " " } }
```

> 欢迎对 Angular 感兴趣的同学入群学习！