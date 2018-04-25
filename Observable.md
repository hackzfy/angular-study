#  RxJS 之 Observable

> 相信每一个刚接触RxJS的人，都会有些困惑。从今天开始，我们来一点点揭开她的神秘面纱。



> Observable 是什么？引用原作者的话：
>
> **Observables are a function that take an observer and return a function.**
>
> 也就是说，Observable 是一个函数，它接收一个observer作为参数，然后返回一个函数。

```js
function Observable(observer){
    
    return () => 1;
}
// 这就是一个Observable！
```



> observer  是什么？
>
> 一个包含了 next, error, complete 方法的对象。

```js
const observer = {
    next: data => console.log('data',data),
    error: error => console.error('error',error),
    complet: () => console.log('complete')
}
// Yes! 这是一个Observer
```



> observer 在 Observable 中起了什么作用？完善一下我们的 Observable

```js
function Observable(observer){
    
    const body = document.body;
    body.addEventListener('mousemove', e=>observer.next(e));
    body.addEventListener('click', e => observer.error(e));
    body.addEventListener('dbclick',() => observer.complete());
    
    return () => body.removeEventListener(...) // remove EventLisnters
}
```



> 可以看到，observable内部有一个生产数据的源 >>> body上的事件监听函数。 每当 body 上触发了事件，相应的事件监听函数就会调用observer的方法，并传入event对象（生产出来的数据）作为参数。至于如何处理数据，是由observer 的next，error，以及complete方法指定的。这个observerble如何使用？

```js
Observable(observer);
// 开始点击body。。
```



> 可以看到，Observable 确实只是一个函数。不去调用它的时候，它就安静的待在那里，没有运行，没有给body添加事件监听，就是安静的待着。只有当被调用的时候，它才会给body添加事件监听。监听到事件后，事件监听函数会调用observer对应的方法。 那么，Observable 到底有什么职责？



> 首先，Observable 内部有一个生产数据的源：producer。这样Observable才会拥有数据，拥有数据才能发射数据。



> 其次，Observable 需要接收一个 observer，才能被真正激活。要记得，他是一个函数，调用以后才能执行。接收到一个observer后，他内部的producer才真正被创建，开始生产并发射数据。这些数据处理按照一定规则传递给observer对应的方法，再由observer的方法去处理。Observable要指明内部的数据应当如何传递给observer，以及传递给observer的哪个方法。



> 最后，Observable的返回值是一个函数。这个函数的作用是让Observable内部的producer停止生产数据。在本文中就是停止body对事件的监听。这样，observable不会再有新的数据传递给observer，observer也不会再执行自己的next等方法。如何使用？

```js
const subscription = Observable(observer);

subscription() // Observable停止发射数据。
```





> 为什么没有看到subscribe方法？
>
> 本文只是对observable原理的解析。并不是真正的Observable实现。官方实现中要考虑程序的健壮性，功能性，实现会复杂很多。但是，只要了解了原理，学习起来就会很快。