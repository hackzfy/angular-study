# Action 命名规范

> 此建议来自 ng-conf [参考链接](https://www.youtube.com/watch?v=JmnsEvoy-gY)，演讲者是 ngrx 的贡献者之一。他在大会上提出了构建可维护应用中如何定义 Action 的几条核心原则。
>
> 一、 不要复用 Action。
>
> 二、使用具有描述性的 Action Type。
>
> 三、避免 Action 分类型。
>
> 四、注重简洁明了。
>
> 五、换位思考。
>
> 六、先写出 action。



------



> 使用具有描述性的 Action Type。
>
> 下面是一个简单的 Action 结构：

```js
interface ModUser{ 
    type: 'modUser';
    payload:any;
}
```

> 首先，modUser 可能被不同的页面触发。打开控制台后，我们看到一堆 Action，但是并不知道这些 Action 是在什么地方被触发的。所以，我们要给 Action 的 type 加入上下文：context。

```js
interface ModUser{ 
    type: '[UserPage] modUser';
    payload:any;
}
```

> 其中 type 属性中的 [UserPage] ，就是上下文。 这样，当我们在控制台追踪 Action 的时候，可以很轻易的知道，这个 Action 是在 UserPage 触发的。

> 其次，我们要能很清晰的从 action type 中看出这个 Action 对应的操作类型:增删改查。上面的例子中 ModUser 并不能明确的表明意图。 可能是更新，也可能是删除。更好的方式是：

```js
interface UpdateUser{
	type: '[UserPage] updateUser', 
	payload: any;
}
```

> 经过修改，我们可以很清晰的得知，这是UserPage 触发的一个 Action，目的是更新用户。

> 如果按照这种方式来组织 Action 的话，岂不是 Action 的数量要翻倍？原来只要一个 updateUser 就可以了，现在我需要 [UserPage] updateUser, [AnotherPage] updateUser, [ThirdPage] updateUser ? 而我的 reducer 也会变成这样：

```js
const userReducer = (state:User,action:Action) => {
    switch( action.type){
        case :'[UserPage] updateUser':
        case :'[AnotherPage] updateUser':
        case: '[ThirdPage] updateUser':
            return {...state,...action.payload}
    }
}
```

> 没错，这个倒是真的。项目的代码会膨胀起来。但是，你要做出选择，是在将来辛苦的 debugger 寻找 action 的出处，还是一眼就可以看出来。这也正对应了第一点原则：不要复用 Action。

---

> 三、避免 Action 分类型。
>
> 还是一个简单的例子：

```js
interface AddItem{
    type: '[CartPage] addItem',
    kind: 'food|toy'
}
```

>因为 AddItem 这个 Action 可能会添加一种食物，也可能添加一种玩具。我们的 reducer 要根据 kind 将数据更新到不同的 state 中。

```js
const cartReducer = (state:any,action: Action) => {
    switch(action.type){
        case '[CartPage] addItem':
            // not good
            if(action.kind==='food') {//doSomething ...}
           
    }
}
```

> 如果我们还有 Effect ，也需要添加判断

```js
class CartApiEffects{
    @Effect() addItem$ = 
        this.action$
		.ofType('[CartPage] addItem')
		.pipe(
        	filter(action=>action.kind==='food')
    	)					
}
```

> 试想如果任由这种风格蔓延，代码将变得多么难以维护。所以，避免使用 Action 分类型，我们完全可以用更简明的方式来解决。

```js
interface AddFood{
    type: '[CartPage] addFood'
}
interface AddToy{
    type: '[CartPage] addToy'
}
// reducer
const cartReducer = (state:any,action: Action) => {
    switch(action.type){
        case '[CartPage] addFood':
            // do something ...
        case '[CartPage] addToy':
            // do something ...
           
    }
}
// effect
class CartApiEffects{
    @Effect() addFood$ = 
        this.action$
		.ofType('[CartPage] addFood')				
}
```

---



> 注重简洁明了和换位思考其实一个意思。ngrx 为你处理了大部分底层的东西，你只要写出简单的 action，reducer，effect 就好。保持他们的简洁与清晰。这样，在别人维护你的代码时，或者你维护遵循这些原则的程序员所写的代码时，会轻松愉快。

---

> 先写出 action。
>
> 这其实是一个需求到达后，我们如何去设计代码，组织代码的一个类似草稿的概念。比如说要实现页面的登录，我们可以先进行以下 action 的规划。

```js
// 用户操作触发的 action
[login Page] Login
// 与服务器交互的 action
[Auth API] Login Success
[Auth API] Login Failure
// 与设备相关的 action
[WebSocket] Open
[WebSocket] Disconnected
```

> 预先规划 action 可以帮助我们深入到需求的细节当中，对每一种交互有完整的解决流程，在写代码时逻辑更加清晰。

> 全文完。
