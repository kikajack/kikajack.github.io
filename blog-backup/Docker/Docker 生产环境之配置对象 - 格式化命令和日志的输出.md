[原文地址](https://docs.docker.com/config/formatting/)

Docker 使用 [Go template](https://golang.org/pkg/text/template/)，你可以使用它来操纵某些命令和日志驱动程序的输出格式。

Docker 提供了一组基本的函数来操作模板元素。所有这些示例都使用 `docker inspect` 命令，但许多其他 CLI 命令也具有 `--format` 标志，并且许多 CLI 命令手册中都包含自定义输出格式的示例。
#1. join
`join` 连接字符串列表以创建单个字符串。它在列表中的每个元素之间放置一个分隔符。
```
$ docker inspect --format '{{join .Args " , "}}' container
```
#2. json
`json` 将元素编码为 JSON 字符串。
```
$ docker inspect --format '{{json .Mounts}}' container
```
#3. lower
`lower` 将字符串转为小写。
```
$ docker inspect --format "{{lower .Name}}" container
```
#4. split
`split` 将字符串切分为由分隔符分隔的字符串列表。
```
$ docker inspect --format '{{split (join .Names "/") "/"}}' container
```
#5. title
`title` 将字符串的首字母大写。
```
$ docker inspect --format "{{title .Name}}" container
```
#6. upper
`upper` 将字符串转为大写。
```
$ docker inspect --format "{{upper .Name}}" container
```
#7. println
`println` 打印完每个值后加一个换行符。
```
$ docker inspect --format='{{range .NetworkSettings.Networks}}{{println .IPAddress}}{{end}}' container
```