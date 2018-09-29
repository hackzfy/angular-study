# Observable的冷与热

> 接触RxJS的同学都应该听说过，Observable分为 cold 和 hot 两种类型。这究竟是什么意思？
>
> 我们来看一个例子：

```js
function myColdObservable(observer){
    int i = 0;
    // 每两秒发射一个数字
    const intervalId = setInterval(()=>observer.next(i++),2000);
    // 返回函数，调用其可以让Observable内部停止发射数据。
    return () => clearInterval(intervalId); 
}
```

> myColdObservable有个特点：每次调用它，都会创建一个定时器，每隔两秒发射一次数据。发射的数据被传递给observer的next方法。上面的代码相当于：

```js
Rx.Observable.interval(2000)
```

> 每次订阅，都会创建一个新的定时器。所以第二次订阅接收到的值又是从0开始的。这就是cold Observable。

```js
let interval$ = Rx.Observable.interval(2000).take(3);

interval$.subscribe(x=> console.log('a',x));
setTimeout(()=>interval$.subscribe(x=> console.log('b',x)),2000);
// 输出：
a 0
a 1
b 0 // --> 从零开始，说明是一个新的定时器
a 2
b 1
b 2
```

> 下面是另一个例子：

```js
function myHotObservable(observer){
    document.body.onclick = (e) => observer.next(e.clientX);
    return ()=> document.body.onclick = null;
}
```

> 不同处在于，数据的产生依赖的是 body 的点击事件。而不是在myHotObservable内部创造的数据。如果body没有被点击，订阅它多少次也不会接收到值。上面的代码相当于：

```js
Rx.Observable.fromEvent(document.body,'click');
```

> 无论订阅多少次，所有的observer在同一个时间接收到的值都是一样的。

```js
let click$ = Rx.Observable.fromEvent(document.body,'click').map(e=>e.clientX);

click$.subscribe( x => console.log('a',x));

setTimeout(()=>click$.subscribe(x=>console.log('b',x)),5000);

// 开始点了
a 384
a 473
a 496
b 496
a 493
b 493
```

> 我们可以简单的认为，数据源如果在Observable内部，那么就是cold Observable. 因为每次订阅都会导致Observable内部的代码重新执行，创造一个全新的数据源。
>
> 相反，hot Observable 无论被订阅几次，数据源都是同一个。本文中就是body的click事件。因为所的有observer都订阅了同一个数据源，自然在同一时间得到的数据也是一样的。



### 如何冷的变成热的？

> 我想让多个Observer 订阅 interval$, 而且需要在同一时间，它们接收到数据是一样的。

```js
let interval = Rx.Observable.interval(2000);
```

> 我们需要 Subject 来帮忙了。 Subject ，既是一个 Observable ，又是一个 observer:

```js

let subject = new Rx.Subject();

subject.subscribe( x => console.log('a',x));
setTimeout(()=>subject.subscribe( x => console.log('b',x)),5000);  // 像 Observable 一样可以被订阅。

interval.take(10).subscribe(subject); // 像 observer 一样，可以订阅别的 Observable.

// 输出：
 a 0
 a 1
 a 2
 b 2
 a 3
 b 3
```

> 这是因为 interval 只被订阅了一次，订阅它的正是subject。因为只被订阅了一次，所以只建立了一个定时器。interval 将值推送给扮演 observer 角色的subject，subject一转身转换成Observable的角色，把接收到的值推送给自己的订阅者。看来，是subject这个中间人才是关键。



> 我们来简单的看一下 Rx.Subject 的原理。它既然可以扮演 observer ，那一定具有 observer 的能力：

```js
class Subject {
    next(data){...},
    error(error){...},
    complete(){},        
}
```

> 它也能扮演 Observable ，那也要具备 Observable 的能力。

```js
class Subject{
    ...
    subscribe(observer){...}
}
```

> 在 subscribe 方法中，它做的事情和 Observable 并不相同：

```js
class Subject{
    observerList = [];
    // ...
    subscribe(observer){
        this.observerList.push(observer);
        return () => this.observerList.filter(obs=>obs!==observer);
    }
}
```

> 就是将订阅它的 observer 加入到自己的 observerList 中。返回的函数则是将对应的 observer 从数组中移除。

> 当Subject扮演 observer 角色去订阅一个Observable时，Observable会在发射数据时调用subject的next方法。那Subject的next方法做了什么？

```js
class Subject{
    observerList = [];
    next(data){
        // 每接收到一个值，就推送给所有订阅自己的 observer。
        observerList.forEach(observer=>observer.next(data))
    }
}
```

> 其实原理就这么简单。在RxJS中，为了方便的将一个cold Observable 转化为 hot Observable，加入了 public, multicast, refCount,share等操作符。它们功能类似，却各有不同。下面是share操作符的例子：

```js
let interval = Rx.Observable.interval(1000).take(10).share();

interval.subscribe(x => console.log('a',x));
setTimeout(
  ()=> interval.subscribe(x => console.log('b',x)),
  5000
)

// 输出：
 a 0
 a 1
 a 2
 a 3
 a 4
 b 4
 a 5
 b 5
 a 6
 b 6
 a 7
 b 7
 a 8
 b 8
 a 9
 b 9
```

> 今天先到这里。👋

