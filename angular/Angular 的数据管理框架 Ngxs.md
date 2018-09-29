# Angular 的数据管理框架 Ngxs 

> 相信用过 redux 和 ngrx 的人都有体会，一大堆的action和reducer为项目开发和维护增加了不少难度。Angular社区为了解决这个问题，推出了新一代数据管理框架 Ngxs。文末有传送门。



> Ngxs 是集 redux 的思路，结合 RxJS 的 Observable，以及Angular 的依赖注入，TypeScript的强类型，面向对象思想于一身的牛叉框架。看看它的出现改变了什么。



- 消灭了大量的 Effect。
- 消灭了大量的 Reducer。
- 消灭了大量的 Selector。
- 完全面向对象。
- 可以利用 angular 的依赖注入。

> 看个例子：

```js


// users.action.ts
export class User{
    constructor(public name:string,public age:number){}
}
export class AddUser{
    static readonly type = '[app] add user';
	constructor(public user: User){}
}

// users.state.ts
export interface UsersStateModel{
    users:User[]
}
@State<UsersStateModel>({
    name:'users',
    defaults:{
        users:[]
    }
})
export class UsersState{
    @Action(AddUser)
    addUser(context:StateContext<UsersStateModel>,action:AddUser){
        const {user} = action;
        const state = context.getState();
        context.patchState({
            users: [...state.users,user]
        })
    }
}

// users.component.ts
@Component({
    selector:'app-users',
    template: `<button (click)="addUser()"></button>`
})
export class UsersComponent{
    constructor(private store: Store){}
    addUser(){
        const user = new User('Mike',18);
        const action = new AddUser(user);
        this.store.dispatch(action);
    }
}
```



> 是不是简单清晰？ 来分析一下它是如何做到的。
>
> 首先，如何去掉 Reducer?
>
> 关键在UsersState 的 decorator 上。 @State。@State 接收的参数是一个对象，其中包括 name 字段，这个字段是必须被赋值并且不可重复的。UsersState 包含的 State 将被包含在全局State的 name 属性中。
>
> reducer中大量的switch case 也不见了。因为每一个 Action 都通过 @Action decorator 与类中的方法建立了对应关系。只要Action被触发，就会调用对应的方法。
>
> 其次，如何消灭大量的 Effect ？
>
> 副作用在大型应用中非常复杂，一个action的触发，会导致连锁反应。所以，NGRX 使用Effect来实现这种Action之间的关联。但是代码越来越多，维护这些Effect的难度也越来越大。



> Ngxs 采用简洁明了的方式来实现异步Action。
>
> 下面是个例子：

```js
// users.state.ts

export class UsersState{
    
    constructor(private usersService:UsersService){}
    @Action(AddUser)
    addUser(context:StateContext<UsersStateModel>,action:AddUser){
        const {user} = action;
        // 注意我们返回了这个 Observable
        return this.usersService.addUser(user).pipe(
            tap(user => {
                const state = context.getState();
                context.patchState({
                    users:[...state.users,user]
                })
            })
        )
    }
}

// users.service.ts
class UsersService {
    constructor(private http: HttpClient){}
    addUser(user:User):Observable<User>{
        return this.http.post('...')
    }
}
```

> Ngxs 采用简单明了的方式来实现副作用。

```js

// users.state.ts

export class GetUsers(){
    static readonly type = '[app] get users'
}
export class UsersState{
  constructor(private usersService:UsersService){}
  @Action(AddUser)
  addUser(context: StateContext<UsersStateModel>, action: AddUser) {
    return this.userService.add(action.user).pipe(
      // 发起一个新的Action，和Effect一样的效果
      map(() => context.dispatch(new GetUsers()))
    );
  }
  @Action(GetUsers)
    getUsers(ctx,action){
        // 注意，一定要return 这个 Observable, Ngxs才可以订阅它。
        // 别忘了，Observable 是惰性的，只有被订阅后才会执行。
        return this.usersService.getUsers().pipe(
        	tap(users=>ctx.patchState([users]))
        )
    }
}
// users.service.ts
class UsersService {
    constructor(private http: HttpClient){}
    getUsers():Observable<User[]>{
        return this.http.get<User[]>(...);
    }
}
```



> 再来看一下组件中如何消灭大量的 subscribe 和 select

```js
export class UsersComponent{
    // 使用 @Select ，你可以传入State的class，它会自动帮你选择class对应的State
    // 这里 users$ 就是 UsersState 内部保存的 state，记住它是一个Observable
    @Select(UsersState) usersState$:Observable<UsersStateModel>;
    // 也可以传入一个函数
    @Select(state => state.users) usersState$:Observable<UsersStateModel>;
    // 也可以不传，只要被装饰的属性名称与 StateClass 的 name 属性一致（除了末尾的$符）
    @Select() users$:Observable<UsersStateModel>;
    
    constructor(private store:Store){}
    
}
```

> 这样就省掉了大量的 Ngrx 中的 subscribe 和 select 代码，如果你用过的话就会明白。
>
> 不过上面的 Select 有个问题，选中的是 UsersState 中保存的 整个State 。它的结构是：

```js
interface UsersStateModel{
    users:User[]
}
```

> 我们如果想直接取 users 这个数组怎么办？ 使用 Ngxs 提供的Selector，它是定义在 UsersState 中的。

```js
// users.state.ts
export class UsersState{
  constructor(private usersService:UsersService){}
  
  // 就这么简单
  @Selector()
  static users(state:UsersStateModel){
      return state.users;
  }
}

// users.component.ts
export class UsersComponent{
    // 注意这里，知道为什么是静态方法了吧。
    @Select(UsersState.users) 
    users$:Observable<User[]>;//--> 我们的到的数据是类型是User数组了
}

```

> 最后将一下如何在组件中dispatch 一个action

```js
@Component({
    selector:'app-users',
    template: `<button (click)="getUsers()">getUsers</button>`
})
class UsersComponent{
    constructor(private store:Store){}
    
    getUsers(){
        this.store.dispatch(new GetUsers());
    }
}
```

> 是不是很面向对象？代码以后维护起来应该很简单了。

官方文档： https://ngxs.gitbook.io/ngxs/getting-started