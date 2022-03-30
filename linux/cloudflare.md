下载批量工具flarectl：<https://github.com/cloudflare/cloudflare-go/releases>
```
$ ./flarectl dns -h    #查看帮助
```
```
添加站点：
$ for domain in $(cat domains.txt);do ./flarectl zone create --zone=$domain --jumpstart=false; done
```
解析：
```
$ for domain in $(cat domains.txt); do ./flarectl dns create --zone=$domain --name="www" --type="CNAME" --content="cname值"; done
如果解析之存在则使用：create-or-update
$ for domain in $(cat domains.txt); do ./flarectl dns create-or-update --zone=$domain --name="www" --type="CNAME" --content="cname值"; done
```

