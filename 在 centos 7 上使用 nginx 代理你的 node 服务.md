# 在 centos 7 上使用 nginx 代理你的 node 服务

> 经常，我们一个服务器上不只有一个应用。我们可以通过给不同的应用设置不同的监听端口，通过nginx把请求转发给对应的应用。假设你是管理员用户，不用打那么多 sudo。
>
> 首先安装 nginx

```
yum install nginx
```

>安装后启动服务

```
service nginx start
```

> 查看是否启动成功

```
curl localhost
```

> 如果启动成功，会展示一段 HTML 字符。否则会报错。
>
> 现在准备我们的node应用。新建文件夹 app

```js
// index.js
const http = require('http')

const server = http.createServer((req,res)=>{
  res.writeHead(200,{'Content-Type':'text/plain'})
  res.end('Welcome to Node.js!');
});

server.listen(8080);

console.log('Server running at port %d',8080);
```

> 启动你的应用：

```
node index.js
```

> 测试应用是否已经启动：

```
curl localhost:8080
```

>  看到 Welcome to Node.js!  说明已启动。
>
> 下面就该将应用配置到 Nginx 中去了。

```
cd /etc/nginx/conf.d
```

> 新建文件 app.conf

```
vi app.conf
```

> 输入以下内容 ，将 server_name 设置为你服务器的域名,proxy_pass 的端口设置为你的应用监听的端口。

```
server{
  listen 80;
  server_name www.example.com;
  location /{
    proxy_pass http://127.0.0.1:8080/;
    proxy_set_header Host $host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header  X-Real_IP $remote_addr;
    proxy_set_header X-Forwarded-Proto $scheme;
  }
}
```

> 这样还是不行的，还需要修改另一个文件。

```
vi /etc/nginx/nginx.conf
```

> 将 http 中的 server 给去掉，改完后 http 属性是下面的样子

```
http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    # Load modular configuration files from the /etc/nginx/conf.d directory.
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    # for more information.
    include /etc/nginx/conf.d/*.conf;
    
    # server 被我删掉了，否则它监听80端口，将请求引导到 nginx 的欢迎页面。
}
```

> 测试配置文件是否有效

```
nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

> 说明配置没有问题。重启nginx 服务

```
service nginx reload
```

> 打开客户端的浏览器，访问你的域名。看到 Welcome to Node.js ! 你就成功了。

