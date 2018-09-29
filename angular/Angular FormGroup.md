# Angular FormGroup

使用 Angular 来创建响应式表单真的是 so easy。我们先用 fromGroup 来创建一个表单。

```typescript
@Component({
  selector: 'app-root',
  template: `
    <form [formGroup]="myForm">
      <input formControlName="userName">
      <input formControlName="password">
      <button>submit</button>
      <div>{{myForm.value|json}}</div>
    </form>
`
})
export class AppComponent implements OnInit {

  myForm: FormGroup;

  constructor() {

  }

  ngOnInit(): void {
    this.myForm = new FormGroup({
      userName: new FormControl('abc'),
      password: new FormControl('123'),
    });
  }
}
```

需要注意三个要素：

1. 创建响应式表单，需要在模块中引入 ReactiveFormsModule。
2. 注意 form 元素上的 formGroup ，它包含一组 FormControl ，跟踪他们的值和校验状态的变化。
3. 每一个需要跟踪的 FormControl 都需要显式的指定 formControlName，这个指定的名称会做为其在FormGroup中的key。

关于FormControl 的具体使用方式，在上一篇文章中已经做了说明。

通过给 form 元素添加 ngSubmit 事件，来控制表单提交的逻辑。

```typescript
<form [formGroup]="myForm" (ngSubmit)="onSubmit()"> ...

...
onSubmit(){
    console.log(this.myForm.value); // { "userName": "abc", "password": "123" }
    this.http.post('...',this.myForm.value).subscribe(...)
}
```

Angular 提供了 FormBuilder 类，可以在表单域较多时，显著的减少代码量：

```typescript
constructor(private fb:FormBuilder)
ngOnInit(){
     this.myForm = this.fb.group({
      userName: 'abc',
      password: '1234',
    });
}
```

对 Angular 了解的越多，你做项目的速度就越快！ 