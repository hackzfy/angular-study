# Angular 最佳实践—应该做的事

## tsconfig.json 设置 'noImplicitAny':true

设置以后，当代码中的变量或者函数的参数未设置类型时，typescript 会报错。严格的类型检查让你的代码不易出错，typescript 为你提供了这样的功能，应该好好的利用它。



## 组件

组件是 Angular 的核心之一，如果可以把他们很好的组织起来，提升他们的可复用度，那么开发工作的一半就算完成了。

###  创建一个或者多个基本组件类

如果你的很多组件中都用到了相同的数据，或者处理逻辑，而又不想在每个组件中重复去定义这些变量和方法，那基本组件类就派上用场了。



```js
enum Status {
  Unread = 0,
  Read = 1
}

abstract class AbstractBaseComponent {
  status = Status;
}

@Component({
  selector: 'component-with-enum',
  template: `
    <div *ngFor="notification in notifications" 
        [ngClass]="{'unread': notification.status == statuses.Unread}">
		<!-- status 这个属性是从父类中继承下来的 -->
      {{ notification.text }}
    </div>
`
})
class NotificationComponent extends AbstractBaseComponent {
  notifications = [
    {text: 'Hello!', status: Statuses.Unread},
    {text: 'Angular is awesome!', status: Statuses.Read}
  ];
}
```

另一种情况是当你构建应用时，很多页面都会用到表单。一个典型的带表单的组件是这样的：

```js
@Component({
  selector: 'component-with-form',
  template: `...`
})
class ComponentWithForm  {
  /* formGroup 对象*/
  form: FormGroup;
 /* 表单是否已经提交 */
  submitted: boolean = false; 
  
  /* 重置表单 */
  resetForm() {
    this.form.reset();
  }
  /* 提交表单 */
  onSubmit() {
    this.submitted = true;
    if (!this.form.valid) {
      return;
    }
    /* 提交表单的逻辑 */
    // ...
  }
}
```

我们可以把这个典型的逻辑抽离出来：

```js

abstract class AbstractFormComponent {
  form: FormGroup;
  submitted: boolean = false;
  resetForm() {
    this.form.reset();
  }

  onSubmit() {
    this.submitted = true;
    if (!this.form.valid) {
      return;
    }
  }
}
```

所有包含表单的组件都可以继承它，让自己内部更简洁：

```js

@Component({
  selector: 'component-with-form',
  template: `...`
})
class ComponentWithForm extends AbstractFormComponent {
  
  onSubmit() {
    super.onSubmit();
    /* 组件内部逻辑 */
  }
}
```

### 容器组件

容器组件专注于处理数据，它可以帮助你提取出更多可复用的 UI Component。



```js
@Component({
    selector:'app-user',
    template: `
		<div>
			<input [value]="user.name">
			<input [value]="user.age">
		</div>
	`
})
export class UserComponent implements Oninit{
    user;
    constructor(private readonly userService:Userservice){}
    
    onInit(){ 
    	this.user = this.userService.getUser();
    }
}
```

这个组件没有什么问题。但是如果我们本地已经存在一个user 的数组，希望通过这个组件渲染出来，发现不行。因为 组件是通过在初始化时，调用 userService 的方式来获取数据的。这就值得我们思考：组件到底是用来做什么的？我认为组件可以分为两种：一种负责处理数据逻辑，一种负责UI 展示。

 至少 UserComponet 目前看起来没有处在正确的位置上，它既负责数据处理，又负责 UI 展示，结果就是不能被复用。

我们可以通过 容器组件 来改造它：

```js
@Component({
  selector: 'user-container-component',
  template: `<app-user-component [user]="user"></app-user-component>`
})
/* 容器组件，负责处理数据 */
class UserContainerComponent {
  
  constructor(userService: UserService) {}
  ngOnInit(){
    this.userService.getUser().subscribe(res => this.user = user);
  }
  
}
/* 改造后的 负责 UI 展现的 UserComponent */
@Component({
  selector: 'app-user-component',
  template: `
		<div>
			<input [value]="user.name">
			<input [value]="user.age">
		</div>
	`
})
class UserComponent {
  @Input() user;
}
```

现在，只要有 user 数据，我们就可以使用 UserComponent 展示出来，它可以被复用了。

> 小经验： 当使用 ngFor 指令时，可以考虑将循环的代码分离出来，封装为一个小组件。

```js

<-- bad -->
<div *ngFor="let user of users">
  <h3 class="user_wrapper">{{user.name}}</h3>
  <span class="user_info">{{ user.age }}</span>
  <span class="user_info">{{ user.dateOfBirth | date : 'YYYY-MM-DD' }}</span>
</div>

<-- good -->

<user-detail-component *ngFor="let user of users" [user]="user"></user-detail-component>
```



## Services



### 为 API 调用创建基础的 service 类

```js

abstract class BaseService {

  	protected baseUrl: 'http://your.api.domain';

    protected getFullUrl(url:string){
        return this.baseUrl + url;
    }
    protected handleResponse(response:HttpResponse){
        // do something
        return Observable.of(handledResponse);
    }
    protected handleError(error){
        // do something with error
        return Observable.of(error);
    }
}
```

业务中的 UserService 

```js

@Injectable()
class UserService extends BaseService {

  private relativeUrl: string = '/users/';
  constructor(private readonly http:HttpClient){}
  public getAllUsers(): Observable<User[]> {
    const fullUrl = this.getFullUrl(this.relativeurl);
    return this.http.get(fullUrl)
      		  .pipe(
      			 map(response=>this.handleResponse(response))
      			 catchError(error=>this.handleError)
      			)
  }
}

```

一方面，公共的逻辑被抽离出来得以复用，另一方面，你可以随时在 BaseService 中扩展功能，所有的子类都会立刻获得新增的功能。相比在每个 service 中去做同一种修改效率要高的多。

### 创建工具 service 类

业务中对数据的处理，会有很多共同的逻辑，将这些逻辑代码抽离到 UtilsService 中，让它们得以复用。



### 使用 TypeScript 的 Enum 来管理你的 Api

```js
enum UserApiUrls {
  getAllUsers = 'users/getAll',
  getActiveUsers = 'users/getActive',
  deleteUser = 'users/delete'
}
```

为什么不直接使用对象？ 因为对象的属性值是可以修改的。我可以在应用的任何地方修改 getAllUsers 的地址，这并不合理。

为什么不使用 interface ？因为 interface 只能作为类型约束来使用，它的任何属性都是类型，不是一个实际的值，在编译后都会消失。

为什么不使用 class？实际上可以使用 class，将所有的 url 定义为 class 的 static 属性，但是，这难道有 enum 用起来方便吗？



###  在合适的地方缓存http请求返回的数据

对于某些基本不可能发生改变的数据，我们没必要每次用到的时候去请求。第一次请求后，缓存起来就可以了。建议在 http 中间件中处理。

## Templates

### 不要在模板中写大量的逻辑代码，应该抽离到组件的方法中。

一个不好的例子：

```js

@Component({
  selector: 'component-with-form',
  template: `
        <div [formGroup]="form"
        [ngClass]="{
        'has-error': (form.controls['firstName'].invalid && (submitted || form.controls['firstName'].touched))
        }">
        <input type="text" formControlName="firstName"/>
        </div>
    `
})
class SomeComponentWithForm {
  form: FormGroup;
  submitted: boolean = false;

  constructor(private formBuilder: FormBuilder) {
    this.form = formBuilder.group({
      firstName: ['', Validators.required],
      lastName: ['', Validators.required]
    });
  }
}
```

将模板中的逻辑抽离到 component 中

```Js

@Component({
  selector: 'component-with-form',
  template: `
        <div [formGroup]="form" [ngClass]="{'has-error': hasFieldError('firstName')}">
            <input type="text" formControlName="firstName"/>
        </div>
    `
})
class SomeComponentWithForm {
  form: FormGroup;
  submitted: boolean = false;

  constructor(private formBuilder: FormBuilder) {
    this.form = formBuilder.group({
      firstName: ['', Validators.required],
      lastName: ['', Validators.required]
    });
  }
  
  hasFieldError(fieldName: string): boolean {
    return this.form.controls[fieldName].invalid &&
        (this.submitted || this.form.controls[fieldName].touched);
  }

}
```

> 总之，我们所说的所有最佳实践，都是为了提升代码的复用性，结构的清晰度。说到底是提升项目的可维护性，可扩展性。

> 原文地址：https://codeburst.io/angular-bad-practices-eab0e594ce92