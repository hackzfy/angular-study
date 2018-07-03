# Angular Http 请求的错误处理方式

在 Angular 中，对 http 请求的错误处理非常重要。有时候客户端由于网络原因，或服务端因为出了些异常，导致请求失败。如果我们能重新请求一下，就会获得正确的响应。今天我们主要来说一下，对于失败的请求，如何重新发起。

- retry

```js
http.get('url').pipe(
	retry(3),
    catchError(error=> this.handleError(error))
)
```

上面的代码在请求失败后，会重新请求三次，如果三次请求同样都失败了，才会触发error，走进 catchError的处理逻辑中。

上面的代码问题是三次请求都是立即发起的，间隔时间短，服务器也许还没有从异常中恢复。三次请求连续失败的可能性较大。我们希望在请求失败后，延迟两秒再次发起请求，这样成功机率大些。这就需要用到另一个操作符。

- retryWhen

```js
http.get('url').pipe(
	retryWhen(error=>error.pipe(delay(2000))),
    catchError(error=>this.handleError(error))
)
```

可以看到，当请求失败后，每隔两秒，就会再次发出请求。这时候问题又来了，如果服务器这个接口真的出问题了，总不能无止境的请求下去，要限制重试次数。

```js
http.get('url').pipe(
	retryWhen(error=>error.pipe(delay(2000),take(2))),
    catchError(error=>this.handleError(error))
)
```

现在，我们最多试两次，就不试了，还不成功就让它走catchError的流程。但是，你会发现即使请求都失败了，也不会走 catchError的流程。

这是因为 retryWhen 重试两次后，就 complete 了。它并没有将 error 传递给后面的操作符。如何解决？我们还需要一个操作符。

- scan

```js
http.get('url').pipe(
    retryWhen(error=>{
        error.pipe(
            scan((retryCount,_error)=>{
                retryCount+=1;
                if(retryCount>2){
                    throw _error;
                }else{
                    return retryCount
                }
            },0),
            delay(2000)
        )
    }),
    catchError(error=>this.handleError)
)
```

通过 scan 我们记录并叠加了重试次数，如果重试次数大于2次，就抛出错误，而这个错误，正是 http 的错误对象。这样，catchError就可以抓到并处理了。试试吧！