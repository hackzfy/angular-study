JSON 转 TypeScript Interface

TypeScript 大法好。

但是每次将后端返回的 json 转成 interface ，真的心好累！

可是，现在有了 quicktype ！

你可以直接在浏览器中打开它的网站，在线将 json 转成你喜欢的类型，包括 TypeScript Interface , Java 类，Go 结构体 。。。。

这个地址就是： 

```js
https://app.quicktype.io/
```

这里是它的 git 仓库

```
https://github.com/quicktype/quicktype
```

你可以通过 npm 安装它：

```
npm install -g quicktype
```

这是后端返回的 json:

```json
{
  "name": "How To Live Forever",
  "artist": {
    "name": "Michael Forrest",
    "founded": 1982,
    "members": [
      "Michael Forrest"
    ]
  },
  "tracks": [
    {
      "name": "Get Connected",
      "duration": 208
    }
  ]
}
```

转换成 typescript interface:

```js
export interface Album {
    name:   string;
    artist: Artist;
    tracks: Track[];
}

export interface Artist {
    name:    string;
    founded: number;
    members: string[];
}

export interface Track {
    name:     string;
    duration: number;
}

```



Enjoy yourself! 😁