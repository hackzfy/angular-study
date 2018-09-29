# angular与websocket---寻找更加便捷的工具

> 前面两篇文章掌握的话，我们已经可以轻松的在应用中使用websocket了。但是，angular庞大的社区早就为我们准备好了便利的工具，直接使用即可。进入你的angular项目，安装它。

```
npm install ng-socket-io
```

> 在 root module 或者 core module 中引入。

```js
import {BrowserModule} from '@angular/platform-browser';
import {NgModule} from '@angular/core';


import {AppComponent} from './app.component';
import {SocketIoService} from './socket-io.service';
import {SocketIoConfig, SocketIoModule} from 'ng-socket-io';

const config: SocketIoConfig = {url: 'http://localhost:8080', options: {}};

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    SocketIoModule.forRoot(config)
  ],
  providers: [SocketIoService],
  bootstrap: [AppComponent]
})
export class AppModule {
}

```

> 创建 socket-io.service.ts 文件。

```
ng g s socket-io -m app
```

> 代码

```js
import {Injectable} from '@angular/core';
// 请注意，不要引错了
import {Socket} from 'ng-socket-io';
import {Observable} from 'rxjs/Observable';

@Injectable()
export class SocketIoService {

  message$: Observable<string>;

  constructor(
    private socket: Socket
  ) {
    this.message$ =
      this.socket
        .fromEvent('serverMessage');
  }

  sendMessage(msg: string) {
    this.socket
      .emit('clientMessage', msg);
  }
}

```

> 在 组件中使用

```js
import {Component, OnInit} from '@angular/core';
import {SocketIoService} from './socket-io.service';

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

  constructor(private ioService: SocketIoService) {
  }

  ngOnInit(): void {
    this.ioService.message$.subscribe(
      msg => {
        console.log(msg);
        this.messages.push(msg);
      }
    );
  }

  send(message: string) {
    this.ioService.sendMessage(message);
  }
}
```

> 总结一下，以后再也不用为websocket发愁了。