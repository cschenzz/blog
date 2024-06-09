```bash
# 参考文档
# https://docs.docker.com/engine/reference/commandline/ps/

# 列出所有容器
docker ps -as --format "table {{.ID}}\t{{.Names}}\t{{.Image}}\t{{.Status}}\t{{.Size}}"

# 列出已启动正在运行的容器
docker ps -s --format "table {{.ID}}\t{{.Names}}\t{{.Image}}\t{{.Status}}\t{{.Size}}"
docker ps -f "status=running" --format "table {{.ID}}\t{{.Names}}\t{{.Image}}\t{{.Status}}\t{{.Size}}"

# 列出未启动的容器
docker ps -f "status=exited" --format "table {{.ID}}\t{{.Names}}\t{{.Image}}\t{{.Status}}\t{{.Size}}"


# 启动容器
docker start gitea

# 停止容器
docker stop 3596c5b195d7

# 查看日志
docker logs --since 30m jenkins-01
```

## 安装mysql8.x
```shell
docker run --name mysql-8 -p 13306:3306 -e MYSQL_ROOT_PASSWORD=88888888 -d mysql:8.4.0
# -------------
docker pull mysql:latest
docker pull mysql:8.0.37-debian
docker pull mysql:8.4.0
docker pull mysql:5.7.44
```

## 安装运行redis
```bash
# 开机运行
docker run --restart=always --name redis-01 -p 6379:6379 -d redis redis-server --appendonly yes
```

---------------------------------
## 用docker安装Rabbit MQ

```bash
docker run --restart=always \
-dit --name rabbitmq \
-e RABBITMQ_DEFAULT_USER=admin \
-e RABBITMQ_DEFAULT_PASS=admin%qwe123 \
-p 15672:15672 \
-p 5672:5672 \
rabbitmq:management

# 访问地址
http://localhost:15672/

# rabbit mq 2
docker run -dit --name rabbitmq-9x -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=88888888 -p 9675:15672 -p 9672:5672 rabbitmq:management
```

> rabbitmq用户名密码: admin(88888888)

## 安装运行minio

```bash
docker run -p 9000:9000 --name minio \
-d \
-e "MINIO_ACCESS_KEY=admin" \
-e "MINIO_SECRET_KEY=admin888" \
-v /home/chenzz/minio/data:/data \
-v /home/chenzz/minio/config:/root/.minio \
minio/minio server /data
```

## 安装gitea

```bash
docker pull gitea/gitea
docker run -d --privileged=true --restart=always --name=gitea -p 10022:22 -p 3000:3000 -v /var/lib/gitea:/data gitea/gitea:latest
# 访问地址:http://127.0.0.1:3000/
# 初始化系统用户使用git（不要修改）
```

## 用docker安装Elasticsearch

```bash
docker run --name es \
-p 9200:9200  \
-p 9300:9300   \
-e ES_JAVA_OPTS="-Xms1g -Xmx1g"  \
-d elasticsearch:7.6.1

# -e "discovery.type=single-node" 设置为单节点
# 特别注意：
# -e ES_JAVA_OPTS="-Xms1g -Xmx1g" \ 测试环境下，设置ES的初始内存和最大内存，否则导致过大启动不了ES
```

```bash
docker pull elasticsearch:7.13.2

docker run --name es-7 -d -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" docker.io/library/elasticsearch:7.13.2
```

> docker run -d -p 9200:9200 -p 9300:9300 --name es-7 elasticsearch
> 


## docker安装及运行nginx

```bash
# 先运行一次容器（为了拷贝配置文件）
# 1.
sudo docker run -p 80:80 --name nginx \
-v /mnt/data/nginx/html:/usr/share/nginx/html \
-v /mnt/data/nginx/logs:/var/log/nginx  \
-d nginx


# 将容器内的配置文件拷贝到指定目录：
sudo docker container cp nginx:/etc/nginx /mnt/data/nginx/

# 修改文件名称：
mv nginx conf

# 终止并删除容器：
docker stop nginx
docker rm nginx

# 使用如下命令启动Nginx服务：
docker run -p 80:80 --name nginx \
-v /mnt/data/nginx/html:/usr/share/nginx/html \
-v /mnt/data/nginx/logs:/var/log/nginx  \
-v /mnt/data/nginx/conf:/etc/nginx \
-d nginx
```


## 安装docker
> https://docs.docker.com/engine/install/ubuntu/

## 使用docker安装运行mysql

```bash
sudo docker pull mysql:latest
sudo docker run -itd --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=password mysql
```

## 运行mariadb

```bash
sudo docker run --restart=always \
-d --name mariadb \
-p 13306:3306 \
-e MYSQL_ROOT_PASSWORD=88888888 \
-v /mnt/data/mariadb:/var/lib/mysql \
mariadb:latest
```

## 部署InfluxDB(时序数据库)

```bash
docker pull influxdb:latest
docker run -d --name influxdb --restart always -p 8086:8086 -v /home/influxdb/data:/var/lib/influxdb2 influxdb:latest

# 访问地址
# http://localhost:8086/
```

## docker中安装jenkins

```bash
docker pull jenkins/jenkins:lts
docker pull jenkins/jenkins:jdk17
docker pull jenkins/jenkins:latest-jdk17
docker pull jenkins/jenkins:jdk21

docker images


# 创建一个jenkins目录 
mkdir /home/jenkins-home

# 启动一个jenkins容器
docker run -d --name jenkins-01 -u 0 -p 10086:8080 -v /home/jenkins-home:/var/jenkins_home jenkins/jenkins:lts

# 进入容器内部
docker exec -it jenkins-01 bash

#执行：
cat /var/jenkins_home/secrets/initialAdminPassword
# 得到密码并粘贴过去


# ----------------------
# 启动一个jenkins容器(跟随系统启动)
docker run --restart=always -d --name jenkins-jdk17 -u 0 -p 10086:8080 -v /home/chenzz/.jenkins-home:/var/jenkins_home jenkins/jenkins:jdk17

# 停止服务
sudo docker stop jenkins-jdk17

# 启动服务
sudo docker start jenkins-jdk17
```

-------------------------------------------
- [docker命令官网文档](https://docs.docker.com/engine/reference/commandline/info/)
- [安装docker引擎](https://docs.docker.com/engine/)
- [Docker仓库](https://hub.docker.com/search?q=&type=image)