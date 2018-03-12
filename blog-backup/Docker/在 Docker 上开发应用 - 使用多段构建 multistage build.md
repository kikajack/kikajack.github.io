[原文地址](https://docs.docker.com/develop/develop-images/multistage-build/)

Docker 17.05 或更高版本才支持多段构建这个新特征。如果想要优化 Dockerfile 使其易读可维护，那么多段构建很有用。

下面的例子来自 [Builder pattern vs. Multi-stage builds in Docker](http://blog.alexellis.io/mutli-stage-docker-builds/) 这篇文章。
#1. 之前的多段构建
构建镜像的一个很有挑战性的事情是保持镜像的小体积。Dockerfile 中的每条指令都会为镜像添加一个层，并且需要你记得在进行到下一层之前清理所有不需要的文件。为了编写高效的 Dockerfile，通常需要使用 shell 技巧和其他逻辑来尽可能地减小层，并确保每个层都从上一层获取且仅仅获取需要的文件。

通常，开发环境使用一个包含构建应用所需的所有东西的 Dockerfile，而生产环境使用另一个精简过的只包含应用程序和应用程序运行时所需的东西的 Dockerfile。这被称为“构建器模式”（builder pattern）。维护两个 Dockerfile 并不理智。

下面是一个 `Dockerfile.build` 和 `Dockerfile` 的例子，它遵循上面的构建器模式：

`Dockerfile.build`：
```
FROM golang:1.7.3
WORKDIR /go/src/github.com/alexellis/href-counter/
COPY app.go .
RUN go get -d -v golang.org/x/net/html \
  && CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o app .
```
注意，此示例还使用 `Bash &&` 运算符人为合并两个 RUN 命令（不推荐），以避免在镜像中创建额外的层。这很容易失败并且很难维护。例如，很容易在插入另一个命令时忘记继续使用 `\` 字符。

`Dockerfile`：
```
FROM alpine:latest  
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY app .
CMD ["./app"] 
``` 
`build.sh`：
```
#!/bin/sh
echo Building alexellis2/href-counter:build

docker build --build-arg https_proxy=$https_proxy --build-arg http_proxy=$http_proxy \  
    -t alexellis2/href-counter:build . -f Dockerfile.build

docker container create --name extract alexellis2/href-counter:build  
docker container cp extract:/go/src/github.com/alexellis/href-counter/app ./app  
docker container rm -f extract

echo Building alexellis2/href-counter:latest

docker build --no-cache -t alexellis2/href-counter:latest .
rm ./app
```
运行 `build.sh` 脚本时，它需要构建第一个镜像，然后创建容器并把文件复制出来，然后再构建第二个镜像。所有的镜像都需要在系统上占据空间，而与此同时在本地磁盘上也有应用代码。

多阶段构建大大简化了这种情况！
#2. 使用多段构建
使用多段构建，可以在 Dockerfile 中使用多个 FROM 语句。每个 FROM 指令可以使用不同的基础镜像，并且各自分别开启一个新的构建阶段。可以选择性地将文件从一个阶段复制到另一个阶段，并在最终镜像中留下所有不需要的内容。为了演示这是如何工作的，让我们修改上一节中的 Dockerfile 以使用多段构建。

`Dockerfile`：
```
FROM golang:1.7.3
WORKDIR /go/src/github.com/alexellis/href-counter/
RUN go get -d -v golang.org/x/net/html  
COPY app.go .
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o app .

FROM alpine:latest  
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=0 /go/src/github.com/alexellis/href-counter/app .
CMD ["./app"]  
```
只需要一个 Dockerfile 并且不再需要一个单独的构建脚本。直接运行 `docker build`:
```
$ docker build -t alexellis2/href-counter:latest .
```
最终结果是与之前的例子一样生成了相同的小型生产映像，复杂性显着降低。不需要创建任何中间镜像像，也不需要将任何文件提取到本地系统。

这是如何工作的？第二个 FROM 指令开启了一个使用 `alpine:latest` 镜像作为基础镜像的新的构建阶段。`COPY --from=0` 行将刚才构建的结果从前一阶段复制到这个新阶段。 Go SDK 和任何中间文件都被留下，并未保存在最终镜像中。
本段原文：
How does it work? The second FROM instruction starts a new build stage with the  as its base. The  line copies just the built artifact from the previous stage into this new stage. The Go SDK and any intermediate artifacts are left behind, and not saved in the final image.
#3. 命名每个构建步骤
默认情况下，步骤没有名字，可以用它们的整数号码表示它们。第一个 FROM 指令作为 0 开始。然而，可以通过添加 `as <NAME>` 到 FROM 指令来为每个步骤命名。这个例子通过为构建步骤命名并在 COPY 指令中使用命令来改进了之前的例子。这意味着即使后面重新排序 Dockerfile 中的指令，COPY 也不会出问题。
```
FROM golang:1.7.3 as builder
WORKDIR /go/src/github.com/alexellis/href-counter/
RUN go get -d -v golang.org/x/net/html  
COPY app.go    .
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o app .

FROM alpine:latest  
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=builder /go/src/github.com/alexellis/href-counter/app .
CMD ["./app"]  
```
#4. 在指定的构建步骤停止
构建镜像时，不一定需要构建包括所有阶段的整个 Dockerfile。可以指定一个目标构建阶段。以下命令假定你在使用上面例子中的 Dockerfile，但在名为 builder 的阶段停止：
```
$ docker build --target builder -t alexellis2/href-counter:latest .
```
几种适合的情况是：

- 调试一个特点的构建阶段
- 在启用了所有调试符号或工具的情况下使用调试阶段，或用于精简生产
- 使用应用程序只能获取测试数据的测试阶段，但是生产的构建使用另外一个使用真实用户数据的阶段（Using a testing stage in which your app gets populated with test data, but building for production using a different stage which uses real data）
#5. 使用外部镜像作为“stage”
使用多段构建时，对之前通过 Dockerfile 创建的阶段的复制是不受限制的。可以使用 `COPY --from` 指令从单独的镜像中复制，可以用本地镜像名，本地或 Docker registry 中可用的标记或标记 ID。Docker客户端在必要时拉取图像并从那里复制文件。语法是：
```
COPY --from=nginx:latest /etc/nginx/nginx.conf /nginx.conf
```