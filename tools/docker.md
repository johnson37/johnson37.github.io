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

## Docker Volumn

```bash
docker volume ls //查看volume
docker volume rm $(docker volume ls -qf dangling=true) //删除所有无用的volume
docker image prune
docker rmi $(docker images | grep “^” | awk “{print $3}”) //删除无用的image
```

```bash
REPOSITORY                       TAG                 IMAGE ID            CREATED             SIZE
cyberiee/loom-app                dev-meeting         0b6d5f7b4729        40 hours ago        922MB  //
cyberiee/loom-frontend           dev                 e0524fd129df        2 days ago          198MB
cyberiee/loom-frontend           v0.39.3             2d3608f52107        2 days ago          198MB  //
cyberiee/loom-app                dev                 47fa8e91a92f        2 days ago          922MB  //
cyberiee/loom-app                v2.46               7eeb306a4a17        3 days ago          922MB
cyberiee/loom-app                v2.46.3             7eeb306a4a17        3 days ago          922MB
cyberiee/loom-app                dev-preview         9d30e02676bf        3 days ago          922MB  //
cyberiee/loom-app                v2.46.2             a4bf82ac9259        4 days ago          922MB  //
cyberiee/loom-doc-server         v7.0                8e29bee6dfaa        8 days ago          2.78GB
mysql                            5.7                 82d2d47667cf        9 days ago          450MB  //
cyberiee/loom-app                v2.46.0             890a698a2b28        2 weeks ago         922MB  //
cyberiee/loom-frontend           v0.39.0             51c670f4aa11        2 weeks ago         198MB  //
cyberiee/loom-app                dev-academics       ee89446d1bec        3 weeks ago         877MB
cyberiee/loom-frontend           v0.38.0             cb7dddbe6512        6 weeks ago         196MB  //
cyberiee/loom-postgres           v0.10.0             25428a57006b        6 weeks ago         1.4GB
cyberiee/loom-app                v2.45.0             d227a6dbbd79        6 weeks ago         921MB  //
cyberiee/loom-frontend           dev-academics       c38963e9eb35        2 months ago        2.38GB
onlyoffice/documentserver        7.0                 d08ace600ec4        2 months ago        2.78GB
cyberiee/loom-app                hg                  d68a2823fd89        2 months ago        884MB
cyberiee/loom-nginx              latest              e059f84db0fd        2 months ago        109MB
cyberiee/loom-unified-exporter   v0.1.2              f714655dcd3d        2 months ago        127MB
cyberiee/loom-scheduler          v0.9.7              45ee9a52892b        2 months ago        1.38GB
node                             12-buster           48cc1192a7eb        2 months ago        886MB
rabbitmq                         3.8                 05d9246ab547        2 months ago        217MB
onlyoffice/communityserver       11.6.0.1620         16f30049dfae        3 months ago        3.7GB  //
onlyoffice/controlpanel          3.0.3.410           5be028ea65f3        4 months ago        519MB  //
prom/node-exporter               latest              1dbe0e931976        4 months ago        20.9MB
centos                           centos7             eeb6ee3f44bd        7 months ago        204MB
redis                            5.0.7-alpine        b68707e68547        2 years ago         29.8MB
wrouesnel/postgres_exporter      v0.8.0              66b1d4e8d60c        2 years ago         12.8MB
centos                           centos6.6           368c96d786ae        3 years ago         203MB
nginx                            1.15.7              568c4670fa80        3 years ago         109MB
google/cadvisor                  latest              eb1210707573        3 years ago         69.6MB
postgres                         9.3.24-alpine       35e57e3ba3a2        3 years ago         36.4MB

```

