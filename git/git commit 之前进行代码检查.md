# 在git commit 之前自动对代码进行检查

现在基本所有的前端项目都会使用 eslint 或 tslint 来做代码检查。在代码 commit 和 push 之前对代码做检查是必要的，防止创建一个代码
有问题的 commit记录和阻止不规范的代码被 push 到代码仓库。

## git commit 之前对代码进行检查

实现的方式很多，但说到底到是在 .git/hooks 中创建 pre-commit 文件。git 会在执行 commit 命令之前，先执行 pre-commit 文件中的命令。
如果命令执行中检测到问题了，这次 commit 就会失败。

## git push 之前对代码进行检查

和 git commit 相似，只是这次 git 会去执行 .git/hooks 中的 pre-push 文件。
其它的勾子还有很多，这里不一一介绍。

## 如何让这一切变得简单？

做为一个前端工程师，学写 bash 也不是什么难事。但如果有更快捷的办法，岂不更好。 它就是 husky .

### 安装 husky

```bash
npm i -D husky
```

### 修改 package.json

这里是一个典型的 angular 项目，在 package.json 中加入 "husky" 配置。

```ts
{
  "version": "0.0.1",
  "scripts": {
    "ng": "ng",
    "lint": "ng lint"
  },
  "husky": {
    "hooks": {
      "pre-commit": "npm run lint",
      "pre-push": "npm run lint"
    }
  },
  ...
}
```

### 大功告成

当运行 git commit 以及 git push 时， 会先对项目使用 ng lint 进行检测，检测不通过，则操作会中断，有效防止不规范的代码提交。

> 如果使用出现了问题，最快的解决方式是重新安装 husky。 如果自己已经创建了 pre-commit pre-push 等勾子，请在安装前手动删除。