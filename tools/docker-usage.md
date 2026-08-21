## 加速配置
> 这里额外添加了docker的生产环境核心配置cgroup
```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://82m9ar63.mirror.aliyuncs.com"],
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

## 找镜像
```bash
#下载最新版
docker pull nginx

# 镜像名:版本名（标签）
docker pull nginx:1.20.1

# redis = redis:latest
# 下载最新
docker pull redis
docker pull redis:6.2.4

# 下载最新的jenkins镜像, 如果本机有此flag的镜像，会覆盖原来flag镜像，原来镜像flag会置空
docker pull jenkins/jenkins:lts

## 下载来的镜像都在本地
# 查看所有镜像
docker images

# 删除本地镜像
docker rmi 镜像名:版本号/镜像id

# 根据镜像id删除镜像
docker rmi 056e9a39e8be
```

## 启动容器
```bash
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

# docker run  设置项   镜像名, 镜像启动运行的命令（镜像里面默认有的，一般不会写）

# -d：后台运行
# --restart=always: 开机自启
docker run --name=mynginx   -d  --restart=always -p  88:80   nginx


# 查看正在运行的容器
docker ps
# 查看所有
docker ps -a
# 删除停止的容器(根据id或者名字, 推荐id)
docker rm  容器id/名字
docker rm -f mynginx   #强制删除正在运行中的

# 停止容器
docker stop 容器id/名字
# 再次启动
docker start 容器id/名字

# 修改应用开机自启
docker update 容器id/名字 --restart=always
```

## 其他操作
```bash
# 查看docker安装配置等
docker info

# 进入容器内部的系统，修改容器内容
docker exec -it 容器id  /bin/bash

# 将镜像保存成压缩包
docker save -o abc.tar guignginx:v1.0

# 别的机器加载这个镜像
docker load -i abc.tar


# 把旧镜像的名字，改成仓库要求的新版名字
docker tag guignginx:v1.0 leifengyang/guignginx:v1.0

# 登录到docker hub
docker login       


docker logout（推送完成镜像后退出）

# 推送
docker push leifengyang/guignginx:v1.0


# 别的机器下载
docker pull leifengyang/guignginx:v1.0

# 排错
docker logs 容器名/id

# docker 经常修改nginx配置文件
docker run -d -p 80:80 \
-v /data/html:/usr/share/nginx/html:ro \
-v /data/conf/nginx.conf:/etc/nginx/nginx.conf \
--name mynginx-02 \
nginx


# 把容器指定位置的东西复制出来 
docker cp 5eff66eec7e1:/etc/nginx/nginx.conf  /data/conf/nginx.conf
# 把外面的内容复制到容器里面
docker cp  /data/conf/nginx.conf  5eff66eec7e1:/etc/nginx/nginx.conf
```

## 修改容器启动配置参数 

有时候，我们创建容器时忘了添加参数 `--restart=always` ，当 Docker 重启时，容器未能自动启动

现在要添加该参数怎么办呢，方法有二：

1、Docker 命令修改
```bash
docker container update --restart=always 容器名字

# 修改jenkins内存最大值
docker container update --memory 500M --memory-swap=800M jenkins-jdk17
```

2、直接改配置文件
首先停止容器，不然无法修改配置文件
配置文件路径为：`/var/lib/docker/containers/容器ID`

在该目录下找到一个文件 hostconfig.json ，找到该文件中关键字 RestartPolicy

```bash
# 修改前配置：
"RestartPolicy":{"Name":"no","MaximumRetryCount":0}

# 修改后配置：
"RestartPolicy":{"Name":"always","MaximumRetryCount":0}
```
最后启动容器。

## 补充
```bash
# 登录docker hub
docker login

#给旧镜像起名
docker tag java-demo:v1.0  leifengyang/java-demo:v1.0

# 推送到docker hub
docker push leifengyang/java-demo:v1.0

# 别的机器
docker pull leifengyang/java-demo:v1.0

# 别的机器运行
docker run -d -p 8080:8080 --name myjava-app java-demo:v1.0 
```

-------------------------------
- [docker命令官网文档](https://docs.docker.com/engine/reference/commandline/info/)
- [debian上安装docker](https://docs.docker.com/engine/install/debian/)
- [Docker仓库](https://hub.docker.com/search?q=&type=image)
- [Docker 备忘清单(QuickRef)](https://quickref.cn/docs/docker.html)