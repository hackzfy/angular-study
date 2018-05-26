# 【进阶】在Angular应用中使用socket.io

> 我们已经知道如何在angular 中使用 websocket 和服务器进行交互。但是程序的健壮性不够，也有很多同学起node服务器的时候会遇到错误，难以解决。客户端代码中的websocket 代码也不够健壮，操作起来也不方便。本文介绍一种更好的方案： socket.io
> 首先，新建服务端文件夹 server。进入文件夹。

```shell
npm init -y
npm install --save socket.io
```

> 建立服务端入口文件 index.js

```js
const http = require('http');
const io = require('socket.io');


const httpServer = http.createServer((req, res) => {
  res.end();
});

const ioServer = io(httpServer);

ioServer.on('connection', socket => {
  console.log('some one connected');
  // 监听客户端推送的 'clientMessage' 事件
  socket.on('clientMessage', msg => {
    console.log(msg);
    // 向客户端推送 'serverMessage' 事件。
    socket.emit('serverMessage', msg);
  });
});


httpServer.listen(
  8080,
  () => console.log('server started')
);

```

> 先用 @angular/cli 新建项目，然后额外安装 socket.io 以及对应的 types。

```
$ npm i --save socket.io
$ npm i --save-dev @types/socket.io-client
```

> 新建 websocket 服务

```js
$ ng g s websocket -m app
```

> 代码：

```js
// websocket.service.ts
import {Injectable} from '@angular/core';
import * as SocketIO from 'socket.io-client';
import {Subject} from 'rxjs/Subject';
import {Observable} from 'rxjs/Observable';

@Injectable()
export class Websocket2Service {

  io;
  message$: Observable<string>;
  private _message$ = new Subject<string>();

  constructor() {

    this.io = SocketIO('ws://localhost:8080');
    this.io.emit('connection');
    // 客户端在监听服务器推送的 'serverMessage'事件。
    this.io.on('serverMessage', (msg) => this._message$.next(msg));
    // 这里为什么进行额外的转换，请看历史文章： Rxjs 之 asObservable
    this.message$ = this._message$.asObservable();
  }

  sendMessage(message: string) {
    // 对比原生的websocket实现，你可以 emit 指定类型的事件，服务器只要在监听
    // 你指定的事件，就会做响应的处理。普通的websocket 里则要通过的发送的信息中
    // 添加额外的类型信息，服务器通过解析才知道这个请求对应的处理方式。
    // 这里服务器在监听着 'clientMessage' 事件。
    this.io.emit('clientMessage', message);
  }
}
```

> 在组件中使用

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

  constructor(private websocketService: WebsocketService) {
  }

  ngOnInit(): void {
    this.websocketService.message$.subscribe(
      msg => {
        console.log(msg);
        this.messages.push(msg);
      }
    );
  }

  send(message: string) {
    this.websocketService.sendMessage(message);
  }
}
```

> 运行截图：

![](/Users/yang/Desktop/docs/屏幕快照 2018-04-28 下午4.56.14.png)
