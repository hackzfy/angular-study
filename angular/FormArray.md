# FormArray

在 Angular 中，使用 formArray 的地方很多，比如 checkbox，只要你想在 form 中提交数组类型的值，就需要使用它。更常见的是动态添加表单项。比如表单中可以添加多个收货地址，多个联系人信息等。

我们直接说如何在后一种情况下使用 FormArray。

情景：表单可以动态添加收货地址。每当点击 【添加】按钮，就会出现一个新的输入框。

```html
<form [formGroup]="myForm">
   <div formArrayName="addresses">
       <div *ngFor="let address of addresses.controls;let i=index">
           <input [formControlName]="i">
       </div>
    </div>
    <button (click)="add($event)">
        add
    </button>
</form>
```

这里要注意三点：

1. 声明一个拥有 formArrayName 指令的父元素。
2. 用 *ngFor 循环得出每一个 formControl 对应的输入框。
3. 每个输入框的 formControlName 是循环中的 index。（0,1,2 ,数组下标)

ts 文件中，需要创建 FormGroup 和 FormArray 实例:

```typescript
...
myForm: FormGroup;
constructor(private fb: FormBuilder) {
    this.myForm = this.fb.group({
        addresses:this.fb.array([['',Validators.required]])
    })
}

get addresses(){
    return this.myForm.get('addresses') as FormArray;
}
add(event){
    event.preventDefault();
    this.addresses.push(new FormControl('',Validators.required))
}
```

formArray 也可以由多个 formGroup 组成.

```html
<form [formGroup]="myForm">
    <div formArrayName="addresses">
        <div *ngFor="let address of addresses.controls;let i=index;"
             [formGroupName]="i">
            <input formControlName="city">
            <input formControlName="street">
        </div>
    </div>
</form>
```

ts:

```typescript
...
constructor(private fb: FormBuilder) {
    this.myForm = this.fb.group({
        addresses: this.fb.array([
            this.fb.group({
                city:'',
                street:''
            })
        ])
    })
}

get addresses(){
    return this.myForm.get('addresses') as FormArray;
}

add(event){
    event.preventDefault();
    const newGroup = this.fb.group({
        city:['',Validators.required],
        street:'',
    });
    this.addresses.push(newGroup);
}
```

动态添加表单项在 angular 中非常简单。Enjoy 😁！