# Angular ControlValueAccessor

表单是 Angular 的一大亮点。但是有时候，我们的表单需要获取一些自定义组件或第三方组件的值。这些组件不是一个正常的 form 元素，（不是 input , select等标准的表单元素),但是它们确实会有值。怎么才能将这些组件的值也通过 Angular 的表单管理起来呢？

举个例子，说下使用场景。我们的表单中包含了一个自定义组件 <app-button-group>，

它用来选择用户的性别。我希望在点击按钮时，将相应的值同步到 form.value 中去。

看看我们的自定义组件：

```typescript
@Component({
  selector: 'app-button-group'
  template: `
    <div>
      <button [class.active]="value==='male'"
			  (click)="value='male'">male</button>
      <button [class.active]="value==='female'"
			  (click)="value='female'">female</button>
    </div>                       
  `,
    // 点击高亮的样式
   styles: [`
      .active {
        background: blue;
        color: white;
      } `
  ]
})                             
                                  
export class ButtonGroupComponent {
    value='male';
}
```

我们的表单：

```typescript
// a.component.ts
@Component({
    selector:'app-a',
    template:`
		<form [formGroup]="form">
			<input formControlName="username">
			<app-button-group formControlName='gender'></app-button-group>
		</form>
		<div>form values:{{form.value|json}}</div>
	`
})
export class AComponent{
    form: FormGroup;
    constructor(private fb:FormBuilder){}
    ngOnInit(){
        this.form = this.fb.group({
            username:'abc',
            gender:'male'
        })
    }
}
```

打开控制台，发现报错了：

```js
 ERROR TypeError: dir.valueAccessor.writeValue is not a function
```

这是因为 Angular 无法操控这个自定义的表单元素，如何让 Angular 可以控制它呢？

需要两步：

1. 将组件注册到 NG_VALUE_ACCESSOR 这个token中。
2. 实现 ControlValueAccessor 接口。

```typescript
// button-group.component.ts
@Component({
  selector: 'app-button-group',
  providers: [
    {
      provide: NG_VALUE_ACCESSOR,
      useExisting: ButtonGroupComponent,
      multi: true,
    }
  ],
   ...
})
export class ButtonGroupComponent implements  OnInit,ControlValueAccessor {
    ...
}
```

控制台的错误消失了。可是当我们点击 button 时，什么都没有发生，form.values并没有改变。这是因为，我们并没有在接口的方法中写具体的逻辑。

重要的逻辑有两个：

1. 当组件接收到一个值时，应该做什么。
2. 当组件的值改变时，应该做什么。

第一个逻辑通过 writeValue 方法来实现：

```typesc
// button-group.component.ts
export class ButtonGroupComponent implements  OnInit,ControlValueAccessor {
    value:string;
    writeValue(value:string){
        this.value = value;
    }
}
```

formControl 会将值通过 writeValue方法传递进组件。

```typescript
// parent.component.ts 
this.form = this.fb.group({
      'username': 'zfy',
      'gender': 'female', // 将 'female'传递给 <app-button-group>...
 });

// button-group.component.ts
...
 writeValue(value:string){  // value 就是 female
        this.value = value; // 我们在这里将其赋值给组件自身的value 属性。
 }
...
```

组件自身的 value 属性改变了，那么高亮显示的 button 也会相应的改变。因为模板中的逻辑：

```html
<button [class.active]="value==='male'"
              (click)="value='male'">male
</button>
```

但我们点切换点击 button 的时候，发现 form.value 并没有改变。这是因为我们还没有实现第二步：

```typescript
// button-group.component.ts
...
onChange: any;  // 这个属性保存注册的函数

// formControl 会调用这个方法，并传入一个函数 fn，这个函数接收组件内部新的值，并传递给 formControl,formControl将新的值同步,并作些状态检测，校验之类的工作。
registerOnChange(fn: any): void {
    this.onChange = fn;
}
```

那么什么时候我们的组件会产生新的值呢，当然是点击 button 的时候。

```typescript
// button-group.component.ts
...
 constructor(private eleRef:ElementRef){}
 ngOnInit(): void {
    // 点击 button时，调用 this.onChange，并将新值作为参数。
    // 这样， formControl 就可以接收到新值，
    // formControl 接收到新值， form.values 也会同步
    this.eleRef.nativeElement.addEventListener('click', e => {
      const ele = e.target;
      const nodeName = ele.nodeName;
      const isBtn = nodeName === 'BUTTON';
      isBtn ? this.onChange(this.value) : void 0;
    });
  }
...
```

所以，关键在于两点：

1. formControl 可以将值传递给你的自定义组件。
2. 你的新值可以传递给 formControl。

可能你没看出这和 fromControl 有什么关系，其实只要组件的 directive 中有 ngModel,formControl,formControlName这三个指令中的一种，组件就会与一个formControl实例发生关系。

如果有什么问题，欢迎入群交流。