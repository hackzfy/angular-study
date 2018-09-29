# 在 Angular 中使用 websocket

> websocket 可以实现服务端和客户端的双向数据推送。在一些实时性要求高的业务中，不可或缺。那么如何在Angular应用中使用websocket呢？今天，我们先从原理说起。



> 首先，你要有一个websocket服务器。对于 JavaScript 来说，实在过于简单。
>
> 第一步，创建一个文件夹，命名为server。 在文件夹中运行：

```js
npm init -y
npm install --save ws
```

> ws是nodejs中一个实现websocket的库，有了它就可以很方便的创建websocket服务。下面是我们的服务器代码： index.js。因为代码实在太简单，就没有使用 ts 的必要了。

```js
// index.js
const WebSocket = require('ws');
 
const wss = new WebSocket.Server({ port: 8080 });
 
wss.on('connection', function connection(ws) {
  ws.on('message', function incoming(message) {
    console.log('received: %s', message);
    ws.send(message);
  });
 
  ws.send('something');
});
```

> 启动服务器

```js
node index.js
```

> 此时控制台不应该有任何输出，否则，请检查代码。



> 服务端已经准备就绪，就等客户端访问了。使用 @angular/cli 创建客户端代码。

```js
ng new client
```

> 创建一个service，负责处理 websocket 请求。

```
ng g service websocket --module app
```

> 下面是 WebsocketService 的实现。

```js
import {Injectable} from '@angular/core';
import {Observable} from 'rxjs/Observable';
import {retryWhen, tap} from 'rxjs/operators';
import {timer} from 'rxjs/observable/timer';

@Injectable()
export class WebsocketService {

  message$: Observable<MessageEvent>;
  private URL = 'ws://localhost:8080';
  private ws: WebSocket;

  constructor() {
    this.message$ = this.create();
  }

  sendMessage(message: string) {
    this.ws.send(message);
  }

  private create() {

    const observable = Observable.create(observer => {
      if (!this.ws) {
        // 如果没有建立过连接，才建立连接并且添加时间监听
        this.ws = new WebSocket(this.URL);
        this.ws.onopen = () => console.log('websocket connected');
        this.ws.onmessage = msg => observer.next(msg);
        this.ws.onerror = e => {
          throw new Error('webSocket connection error:');
        };
        // 我们需要保持websocket连接始终在打开状态，所以当断开时，要抛出错误，
        // 然后在错误处理中重新连接
        this.ws.onclose = (e) => {
          throw new Error('webSocket connection closed:');
        };
      }
    });
    // 这里注意 retryWhen 的用法
    return observable.pipe(
      // 在控制台输出错误信息
      tap(err => console.log(err)),
      // 3秒后重试
      retryWhen(
        () => timer(3000)
      )
    );
  }
}
```

> 在 AppComponent 中使用。 

```js
import {Component, OnInit} from '@angular/core';
import {WebsocketService} from './websocket.service';

@Component({
  selector: 'app-root',
  template: `
    <ul>
      <li *ngFor="let msg of messages">{{msg}}</li>
    </ul>

    <p *ngIf="error">{{error | json}}</p>
    <p *ngIf="completed">completed!</p>


    <input type="text"
           #message>
    <button (click)="send(message.value);message.value='';">send</button>
  `,
  styleUrls: ['./app.component.css']
})
export class AppComponent implements OnInit {
  messages = [];
  error: any;
  completed = false;

  constructor(private webSocketService: WebsocketService) {
  }

  ngOnInit(): void {
    this.webSocketService.message$.subscribe(
      msg => this.messages.push(msg.data),
      err => this.error = err,
      () => this.completed = true
    );
  }

  send(message: string) {
    this.webSocketService.sendMessage(message);
  }
}

```

> 运行客户端

```
npm start
```

> 运行截图

![](/Users/yang/Desktop/docs/屏幕快照 2018-04-28 下午3.49.46.png)