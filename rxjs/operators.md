#Operators

虽然RxJS的基础是`Observable`，但是它的精髓在于`Operator`。`Operator`是实现有条理的管理异步代码的关键。

## 什么是`Operator`

`Operators`是`Observable`类型对象的方法，比如`.map(...)`,`.filter(...)`,`.merge(...)`,等等。当这些方法被调用时，并不会改变原始的`Observable`,而是返回一个新的`Observable`.新的`Observable`的执行逻辑以初始的`Observable`为基础.

> 一个`Operator`是一个`function` ,它创建以初始`Observable`为基础的一个新的`Observable`,这是一个纯粹的操作:初始的`Observable`并不会改变.

`Operator`本质上是一个纯函数,以一个`Observable`为输入,生成一个新的`Observable`作为输出.对输出的`Observable`进行`subscribe`,将会导致输入的`Observable`也被`subscribe`.在下面的例子中,我们新建了一个自定义的`Operator`,它把作为输入的`Observable`提交的每个值乘以10.

```
function multiplyByTen(input) {
 var output = Rx.Observable.create(function subscribe(observer) {
   input.subscribe({
     next: (v) => observer.next(10 * v),
     error: (err) => observer.error(err),
     complete: () => observer.complete()
   });
 });
 return output;
}

var input = Rx.Observable.from([1, 2, 3, 4]);
var output = multiplyByTen(input);
output.subscribe(x => console.log(x));
```
输出:
```
10
20
30
40
```
注意对输出的`Observable`的`subscribe`会导致输入的`Observable`被`subscribe`.我们称之为"Operator subscribe chain"(操作符订阅链).

##Instance operators versus static operators(实例操作符和静态操作符)

**什么是instance operator?** 通常提到`operators`,我们认为是`Observable`实例上的`instance operator`.举例来说,如果 `multiplyByTen`操作符是一个正式的操作符,它大概是下面的样子:

```
Rx.Observable.prototype.multiplyByTen = function multiplyByTen() {
 var input = this;
 return Rx.Observable.create(function subscribe(observer) {
   input.subscribe({
     next: (v) => observer.next(10 * v),
     error: (err) => observer.error(err),
     complete: () => observer.complete()
   });
 });
}
```
> `instance operator`是内部的`this`指向输入的`Observable`的函数.

注意作为输入的`input Observable`不再做为函数的参数,它被认为是`this`指向的对象.这是我们为什么能够像下面这样调用它的原因.

```
var observable = Rx.Observable.from([1, 2, 3, 4]).multiplyByTen();

observable.subscribe(x => console.log(x));
```
**什么是`static operator`?**和`instance operator`不同,`static operator`是直接绑定在`Observable`类上面的方法.`static operator`内部不使用`this`,它的执行完全取决于传递过来的参数.

> `static operator`是绑定在`Observable`类上的纯函数,通常用来创建一个`Observable`.

最常见的`static operator`是所谓的`creation operator`.和转换一个`input observable`到一个`output observable`不同,他们接收一个非`Observable`参数,像数字,字符串等,然后创建一个新的`Observable`.

`static operator`的一个典型的例子是`interval operator`.它接受一个数字作为输入参数,然后创建并输出一个`Observable`.

```
var observable = Rx.Observable.interval(1000 /* number of milliseconds */);
```
另一个`creation operator`的例子是我们在前面的例子中大量使用过的`create`.

尽管如此,`static operator`除了简单的`create`,还有很多其他的类型.一些组合操作符可能是静态的,比如`merge`,`concat`,`combineLatest`,等等.他们之所以是静态操作符,原因在于他们以多个`Observable`作为输入,而不只是一个.例如:
```
var observable1 = Rx.Observable.interval(1000);
var observable2 = Rx.Observable.interval(400);

var merged = Rx.Observable.merge(observable1, observable2);
```

##choose an operator

你是否在寻找一个你需要的操作符?先从下面的列表中选择一项:

* 我有一个`Observable`,而且......
* 我有一些`Observables`需要组合成一个`Observable`,而且......
* 我现在没有`Observable`,而且......

##categories of operators

不同的操作符有不同的目的,他们可以被分为:创建,转换,过滤,组合,错误处理,基础工具等等.



本系列文章均翻译自:http://reactivex.io/rxjs/manual/overview.html
