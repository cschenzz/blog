## 手动在linux中安装java jdk

```bash
mkdir /usr/local/java

tar -zxvf jdk-17_x64_linux_hotspot.tar.gz -C /usr/local/java/

# 修改文件夹名称
# mv /usr/local/java/jdk-17.0.7+7 /usr/local/java/jdk-17

vim /etc/profile

# 以下为/etc/profile新增内容
# -----------------
# CLASSPATH在jdk17下可以不配置
# java env, open jdk setting

JAVA_HOME=/usr/local/java/jdk-17
JRE_HOME=$JAVA_HOME/jre
PATH=$PATH:$JAVA_HOME/bin
# CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar

export JAVA_HOME
export JRE_HOME
export PATH
# export CLASSPATH
# --------------------------

# 使环境设置生效
source /etc/profile

# 建立超链接
# -b 删除、覆盖以前建立的链接
ln -s /usr/local/java/jdk-17/bin/java /usr/bin/java
```

1. 下载java jdk压缩包(使用root用户执行)
2. 解压到/usr/local/java目录, tar -zxvf jdk-17_x64.tar.gz -C /usr/local/java/
3. 编辑环境变量, vim /etc/profile

输入命令vim /etc/profile，打开环境变量配置文件
在文件末尾输入上门的内容，并保存

```bash
# -------------------------
JAVA_HOME=/usr/local/java/jdk-17
JRE_HOME=$JAVA_HOME/jre
PATH=$PATH:$JAVA_HOME/bin

export JAVA_HOME
export JRE_HOME
export PATH
```
4.
输入命令`source /etc/profile`，刷新环境变量配置文件使其立刻生效；输入`java -version`或者`java --version`查看已安装的jdk版本
你要以为这就完成了，那就掉坑里了。虽然大部分时候这就够了，但还有一步操作最好做一下。建一个/usr/bin/java的java的超链接。

`ln -b -s /usr/local/java/jdk-17/bin/java /usr/bin/java`
为什么要建这个超链接，因为一些自己注册的linux服务（如springboot的jar注册的服务），默认情况下从/usr/bin/java路径使用java，yum安装的时候，这个超链接会自动创建，如果你自己下载包安装的话，这个超链接就需要你手动创建了。

至此，从官网下载包安装jdk完成。


## windows安装jdk17
```bash
# 从https://learn.microsoft.com/zh-cn/java/openjdk/download下载jdk包
# 解压到目录(如C:\Program Files\microsoft-jdk\jdk-17.0.5+8)
# C:\Program Files\adopt-jdk\jdk-17.0.11+9

# 打开 设置->系统->关于/系统信息->高级系统设置->系统属性(高级)->环境变量
# 系统变量添加: JAVA_HOME=上面jdk解压路径
# 系统环境变量: Path添加%JAVA_HOME%\bin
# jre版本添加: %JAVA_HOME%\jre\bin

# 输入java -version查看验证, 也可以执行java, javac命令
```