# Docker

### Docker 的安装

[docker install manual](https://www.runoob.com/docker/ubuntu-docker-install.html)
可以通过脚本自动安装，也可以手动安装

```
curl -sSL https://get.daocloud.io/docker | sh
```

### docker-compose 安装

```
curl -L https://get.daocloud.io/docker/compose/releases/download/1.12.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose

chmod +x /usr/local/bin/docker-compose
```

### Docker的使用

#### Dockerfile 的使用方法

#### 如何生成docker images

```
docker build -f .\Dockerfile -t first_docker .
```

#### 查看docker images

```
docker images
```

#### docker 启动一个container

** --name: 指的是container的名字， 后面那个是image的名字， -dit：要求后台运行 **

```
docker run -dit --name=first_docker first_docker bash
```

#### 停止一个container

```
docker stop <Container ID>
```

#### Attach One Container

** 这里的first docker是container name **

```
docker exec -it first_docker bash
```

## 