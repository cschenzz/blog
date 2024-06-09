## 在linux中安装配置maven

1. 下载maven包(xx-bin.tar.gz包), 新建文件夹`sudo mkdir /usr/local/maven`
2. 解压到/usr/local/maven目录, `sudo tar -zxvf maven-bin.tar.gz -C /usr/local/maven/`
3. 编辑环境变量, `sudo vim /etc/profile`

```bash
# -------添加以下内容--------
# MAVEN_HOME=/usr/local/maven
MAVEN_HOME=/usr/local/maven/apache-maven-3.9.5

export MAVEN_HOME
export PATH=$PATH:$MAVEN_HOME/bin
```
4. 使环境设置生效, `source /etc/profile`
5. 验证命令`mvn -v`，显示maven版本等信息表示配置成功


```bash
Maven home: /usr/local/maven/apache-maven-3.9.5
Java version: 17.0.8, vendor: Debian, runtime: /usr/lib/jvm/java-17-openjdk-amd64
Default locale: zh_CN, platform encoding: UTF-8
OS name: "linux", version: "6.1.0-12-amd64", arch: "amd64", family: "unix"
```

## win10中配置maven
```bash
# 高级系统设置->环境变量->系统变量
# Path添加idea中maven插件位置, 请确认下面位置存在mvn文件
# D:\app\dev\idea-ic-last\plugins\maven\lib\maven3\bin

# 使用下面命令查看是否生效
# mvn --version
```



---------------------
- [maven下载](https://maven.apache.org/download.cgi)