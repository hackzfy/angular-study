# error: dst refspec matches more than one

## 表现

当前分支变成了 heads/[branchname]

## 原因：分支名和tag名称相同

## 解决方式：

- 删除tag

```shell
# 删除远程服务器上的 tag
git push origin :refs/tags/[tagname]
# 删除本地 tag
git tag -d [tagname]
```

## 恢复正常