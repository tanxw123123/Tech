下载批量工具flarectl：<https://github.com/cloudflare/cloudflare-go/releases>
```shell
$ ./flarectl dns -h    #查看帮助
```
```shell
# 添加站点：
$ export CF_API_KEY=9c546dxxxxxxxxxxxxxx08b101b4
$ export CF_API_EMAIL=Tomxxxxxxprise.com
$ for domain in $(cat domains.txt);do ./flarectl zone create --zone=$domain --jumpstart=false; done
```
```shell
# 解析：
$ for domain in $(cat domains.txt); do ./flarectl dns create --zone=$domain --name="www" --type="CNAME" --content="cname值"; done  

# 如果解析之存在则使用：create-or-update
$ for domain in $(cat domains.txt); do ./flarectl dns create-or-update --zone=$domain --name="www" --type="CNAME" --content="cname值"; done

# 删除解析：
$ for domain in $(cat domains.txt); do ./flarectl dns delete --zone=$domain --id="xxxxxxxxxxxxxxx"; done
```
```shell
# 获取每个域的名称服务器：
$ for domain in $(cat domains.txt);do ./flarectl zone info --zone=$domain; done
```
```shell
# 查看域名解析记录：
$ for domain in $(cat domains.txt);do ./flarectl dns list --zone=$domain; done
```

