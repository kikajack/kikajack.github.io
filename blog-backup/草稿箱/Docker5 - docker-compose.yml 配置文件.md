[v1.6 官方参考](https://docs.docker.com/v1.6/compose/yml/)
[中文参考](http://blog.csdn.net/pushiqiang/article/details/78682323#t27)
#1. docker-compose.yml
Compose 配置文件的默认路径是 `./docker-compose.yml`，扩展名可以是 `.yml` 或 `.yaml`。文件格式可以[参考官网](https://docs.docker.com/compose/compose-file/)，示例：
```
version: "3"
services:

  redis:
    image: redis:alpine
    ports:
      - "6379"
    networks:
      - frontend
    deploy:
      replicas: 2
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure

  db:
    image: postgres:9.4
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - backend
    deploy:
      placement:
        constraints: [node.role == manager]

  vote:
    image: dockersamples/examplevotingapp_vote:before
    ports:
      - 5000:80
    networks:
      - frontend
    depends_on:
      - redis
    deploy:
      replicas: 2
      update_config:
        parallelism: 2
      restart_policy:
        condition: on-failure

  result:
    image: dockersamples/examplevotingapp_result:before
    ports:
      - 5001:80
    networks:
      - backend
    depends_on:
      - db
    deploy:
      replicas: 1
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure

  worker:
    image: dockersamples/examplevotingapp_worker
    networks:
      - frontend
      - backend
    deploy:
      mode: replicated
      replicas: 1
      labels: [APP=VOTING]
      restart_policy:
        condition: on-failure
        delay: 10s
        max_attempts: 3
        window: 120s
      placement:
        constraints: [node.role == manager]

  visualizer:
    image: dockersamples/visualizer:stable
    ports:
      - "8080:8080"
    stop_grace_period: 1m30s
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
    deploy:
      placement:
        constraints: [node.role == manager]

networks:
  frontend:
  backend:

volumes:
  db-data:
```
|字段名|解释|
|---|---|
|version|Compose 文件版本，需要跟 Docker 引擎的版本匹配，[参考官网](https://docs.docker.com/compose/compose-file/#compose-and-docker-compatibility-matrix)|
|services|服务定义，包含将被应用到为该服务启动的每个容器的配置|
|networks|网络定义|
|volumes|存储定义|

1. build
build 可以指定包含构建上下文的路径：