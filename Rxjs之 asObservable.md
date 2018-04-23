# Rxjs 之 asObservable

> 什么意思？Rxjs里不都是Observable么？为什么需要这个操作符？



```js
const interval = Observable.interval(1000);

interval.asObservable().subscribe(x=>console.log(1));

// 你期望的是：
// logs
// 0
// 1
// 2
// ...
// 然而实际上是：
// Uncaught TypeError: interval.asObservable is not a function
```

> 普通的Observable是没有这个方法的。本来就是一个Observable，再包含一个把自己当作Observable的方法？没有人会这样设计代码。其实这个方法只存在于一种特殊的Observable上面，它就是Subject。

```js
const subject = new Subject();

subject.subscribe(x=>console.log(x));

subject.next(1);
// 1
subject.next(2);
// 2
```

> 看起来有点自娱自乐的赶脚。确实是，这家伙既是 Observable，又是 Observer。看：

```js
subject.next(3);
// 3
subject.error('error!')
// Uncaught 'error!'
subject.complete();
// undefined
```

> 这不是observer是什么，observer 不就是有三个方法么？它都有。而asObservable操作符就是让它老老实实的做个Observable，砍掉它的Observer能力。

```js
const observable = subject.asObservable();
observable.next();
// Uncaught TypeError: observable.next is not a function
```

> 现在好了，他不能自娱自乐了。问题是，为什么要这样对他？给我一个理由！

```js
// websocket.service.ts
class WebSocketService {
 	subject = new Subject();
    constructor(){
        // ... 
        websocket.onmessage = data => this.subject.next(data);
    }
}
// in some file
this.websocketService.subject.subscribe(data=>{ /* do something with data */});
                  
// 来了一个搞破坏的
this.websocketService.subject.next('haha!');
        
```

> 我们如何限制这种情况的出现？subject被暴露出来后，代码中任何地方都可以调用它的next方法！



```
// websocket.service.ts
class WebSocketService {
 	private subject = new Subject();
    constructor(){
        // ... 
        websocket.onmessage = data => this.subject.next(data);
    }
    get(){
        return this.subject.asObservable();
    }
}
// in some file
this.websocketService.get().subscribe(data=>{ /* do something with data */});
                  
// 来了一个搞破坏的
this.websocketService.subject.next('haha!');
// error! subject is private !
this.websocketService.get().next('haha!');
// Uncaught TypeError: ...next is not a function
```

