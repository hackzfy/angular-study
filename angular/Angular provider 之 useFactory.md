# Angular Provider

我们可以很容易的编写一个 service ，并将其通过DI注入到Component 或别的 service 中去。

```typescript
// a.service.ts
@Injectable({})
class ServiceA {
    ...
}
    
// a.module.ts
@NgModule({
    ...
    providers:[ServiceA]
 })
class AModule{}

// 
```

但是如果我们想注入的东西并不是一个 service ，该怎么做？看一个简单的例子：

```typescript
@Injectable({})
class ServiceB{
    constructor(private config:ServiceBConfig){}
    
    log(message:string){
        if(this.config.isProd){
            return;
        }
        window.console.log(message);
    }
}
```

这个 service 需要在构造函数中传入一个 config，config 是一个对象，对象中的属性决定了service中 log 方法的逻辑：如果目前是生产环境，则不在控制台输出信息，可能是出于安全性的考虑。

```
interface ServiceBConfig{
    isProd:boolean;
}
```

很明显，config 只是一个对象字面量，并不是一个可注入的 service。如果直接运行项目，会报错，因为 angular 不知道从哪里得到这个 config 对象，用以实例化 ServiceB。

这时候我们就需要用到 provider 的另一种模式： useFactory。

```typescript
// b.module.ts
const config:ServiceBConfig = {
    isProd: false
}
@NgModule({
    ...
    providers:[
        {
			provide: ServiceB,
    		useFactory: serviceBFactory(config),
        }
    ]
})

const serviceBFactory = 
      (config:ServiceBConfig) =>
		() => new ServiceB(config);
```

userFactory 的值必须为一个返回 ServiceB 实例的函数。因为我们需要接收一个 config 来实例化 ServiceB，所以这里使用了高阶函数。serviceBFacotry(config) 执行后返回的正是 useFactory 需要的函数:

```
()=> new ServiceB(config)
```

这样，我们就可以很灵活的根据环境来实例化所需要的service。

我们仍然像以前一样使用它：

```typescript
@Component({...})
export class ComponentB{
    constructor(private serviceB:ServiceB){}
}
```

但是在 ng build 时，控制台会有警告：

```js
Can't resolve all parameters for ServiceB in ... This will become an error in Angular ...
```

这是因为 angular 在 build 时会去检查所有声明了@Injectable 的类的 constructor 需要的参数。Angular 要在程序用到这个类的实例时对其进行实例化。如果参数也是可注入的，Angular 自己可以找到。但是 config 并不是，所以 Angular 会给出警告，因为如果找不到，程序运行时就会出错。

但是我们自己写的程序，很清楚的知道，config 参数我们是用 ServiceBFactory 传进去的。如何避免这个警告呢？很简单，去掉 ServiceB 的 @Injectable 声明：

```typescript
//@Injectable()
export class ServiceB{
    ...
}
```

再进行build，发现警告消失了。

去掉了@Injectable ，我的 serviceB 还可以被注入么？

实际上，当你使用依赖注入时，Angular 寻找的只是一个 InjectionToken：

```typescript
// b.component.ts
...
constructor(private serviceB:ServiceB){}
...
```

它找的是 ServiceB 这个 token ，并不是 ServiceB这个类，我们平时在 providers 数组中声明的 provider，其实相当于

```typescript
@NgModule({
    ...
    //providers:[ServiceA]
    // 相当于
    providers:[
   	{
    	provider: ServiceA, // angular 会创建一个名称一样的 token
    	useFactory: (...args)=> new ServiceA(...args),
    	deps:[...]
	  }
    ]
})

```

angular 会自动生成一个和类名一样的token。并生成一个 factory 函数，函数的参数就是constructor 中的参数。 参数的值是地 deps数组中指定的其它可注入类。

如果我们的 service C 依赖了另一个可注入类：

```typescript
// b.service.ts
export class ServiceC{
    constructor(private userService:UserService){}
}
```

那么实际上 angular 会将基转换为

```typescript
@NgModule({
    ...
    providers:[
    	{
    		provide: ServiceC, 
    		useFactory: (userService:UserService)=>new ServiceC(userService),
    		deps:[UserService]
   		}
    ]
})
```

需要注意的是 UserService 必须声明为可注入的

```typescript
@Injectable()
export class UserService{...}
```

所以，DI 使用时，在构造函数中声明的是 token，而使用的，是这个token对应的实例。

我们也可以偷梁换柱：

```typescript
@NgModule({
    ...
    providers:[
    	{
    		provide: ServiceC, 
    		useFactory: ()=>new ServiceA(),
    		deps:[]
   		}
    ]
})
```

现在，当我们在组件中注入 ServiceC 时，实际得到的是 ServiceA的实例。因为 ServiceA 的 constructor 中没有依赖，所以我们不需要在 factory 函数中声明参数，deps数组也是空的，可以不写。

当然，在实际项目中，我们不会这么使用。因为当你使用 ServiceC时，会创建一个新的 ServiceA 实例，而服务应该（大部分情况) 是单例的。如果真要给 ServiceA 起个别名，应该使用 useExisting

```typescript
...
{provide: ServiceC, useExisting: ServiceA}
```

这样， angular 会在注入 ServiceC 时，寻找已经存在的 ServiceA 的实例，而不是再创建一个。

依赖注入的知识点很多，今天先说这么多。代码是基于 angular5的，angular6中又有一些改变，后面慢慢写。如果有什么问题，欢迎入群交流。

