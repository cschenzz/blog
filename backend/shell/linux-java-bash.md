## java中用到的命令
```bash

# >和>>都属于输出重定向，都可以输出内容到指定文件。
# >会覆盖目标的原有内容，当文件存在时，会先删除原文件，再重新创建文件，然后把内容写入该文件，否则直接创建文件
# >>会在目标原有内容后追加内容，当文件存在时直接在文件末尾进行内容追加，否则直接创建文件

# 后台运行java程序(运行日志放当前目录),运行后会输出线程id
# nohup java -jar sb-admin.jar > __log.txt 2>&1 &
# 指定端口
# nohup java -jar sb-admin.jar --server.port=9995 > __log.txt 2>&1 &

# 显示文件前50行
head -n 50 ./___xx_log.txt

# tail -fn 50 __log.txt
# 滚动显示日志(最新50行,f滚动,n行数),CTRL+C退出

# 导出java运行日志(最后2000行)到新文件(当前文件夹), 因为原始日志文件会比较大
tail -n 2000 /home/java-jar/spring-demo/logs/run.log >> ./temp-xx-01.log

# 在run.log日志文件中查找指定文本, 并输出到指定文件(当前文件夹下文件)
grep search_text /home/java-jar/xx-api/logs/run.log > ./temp-xx-01.txt

# 在run.log日志文件中查找指定订单号的日志
grep 'order_sn' /home/java-jar/xx-api/logs/run.log

# -c, 打印匹配次数
grep -c '/create-order' /root/xx-api/logs/run.log

# 在run.log文件中查找指定文本(这里为订单号), 只输出前2次匹配，并输出行号
grep -m 2 -n 'ooo-00000-8888' /root/xx-api/logs/run.log

# 读取日志文件中指定行(600-900行)并输出到指定文件
sed -n '600,900p' /root/xx-api/logs/run.log > ./temp-xxx-01.txt
# -------------------------------------------------------------

#  ps -aux | grep 'java -jar'
#  ps -ef | grep 'java -jar'
#  也可以使用'jps -l'查看java进程
#  匹配通过java -jar命令运行的java进程,找到pid.用kill -9 [pid]结束进程

# 查看path环境变量
echo $PATH

[chenzz@ubuntu-pc]$ nohup java -jar sb-admin.jar > __log.txt 2>&1 &
[1] 10030

# 关闭通过ps -aux | grep sb-admin 找到pid.用kill -9 [pid]结束进程
ps -aux | grep sb-admin

# 显示最近的50条历史命令(history为全部)
history 50

# 键入ctr+r来在命令历史中搜索命令,回车执行
# history用法:!num执行history中对应数字的命令

# 显示目前登入系统的用户信息。执行w指令可得知目前登入系统的用户有哪些人，以及他们正在执行的程序
# 单独执行 w 指令会显示所有的用户，您也可指定用户名称，仅显示某位用户的相关信息

# 查看cpu的使用情况
top
# 根据内存降序排列, 并列出命令的全路径
top -o RES -c

# wget帮助
wget --help

# 前台下载
wget https://github.com/file-redis.zip

# 后台下载
wget -b https://github.com/microsoftarchive/redis/releases/download/win-3.2.100/Redis-x64-3.2.100.zip
Continuing in background, pid 54259.
Output will be written to ‘wget-log’.
# Output will be written to ‘wget-log.1’.
# 下载任务在后台运行, 可退出控制台，如果退出控制台不会中断下载
# 此时，wget 会将输出信息写入到 wget-log.1 (此文件由wget自动生成。可通过 -o 修改此文件名称)文件中
# 查看wget进度：tail -f wget-log.1

# 下载文件到指定目录(没有此目录则创建)
wget -O gitea https://dl.gitea.io/gitea/1.8.0-rc1/gitea-1.8.0-rc1-linux-amd64
# 使用 wget -c 断点续传


# 显示详细信息(GET请求)
curl -v http://localhost:8888

# 下载文件,选项-o将下载数据写入到指定名称的文件中，并使用--progress-bar显示进度条
curl http://man.linuxde.net/test.iso -o filename.iso --progress-bar
curl --progress-bar -o xx-01.zip https://example.com/path/to/filename.zip

# --------------------------
# 启动项目
java -jar yiiu.jar > log.file 2>&1 &

# 关闭服务
ps -ef | grep yiiu.jar | grep -v grep | cut -c 9-15 | xargs kill -s 9

#检查是否还有相应tomcat进程
ps -ef | grep tomcat
kill 14736: 结束进程
#------------------
```


## linux命令
```bash
# 查看/etc/profile文件内容，并从1开始对所有输出的行数(包括空行)进行编号
cat -n /etc/profile

# .当前目录, ..上一级目录, ~当前用户家目录

# 删除webapp下所有文件
rm -rf ./webapps/*

# 删除当前文件夹下所有文件
rm -rf ./*

# 列出~/app目录中文件, (-a列出所有文件, 包括隐藏文件), -l一行一项
ls ~/app/

# 当前目录
pwd

# 在当前目录递归创建多个目录
# -v, --verbose  每次创建新目录都显示信息
mkdir -pv ./doc/test1

# 改目录名为tomcat-9999
# 支持多级目录只修改一级, 如将aa/bb/cc修改为aa/xx/cc, 使用mv aa/bb/ aa/xx
mv apache-tomcat-8.5.32/ tomcat-9999

# 删除当前目录下的所有文件(夹)
rm -rf ./*

# 拷贝复制文件到当前目录, -f如果文件存在, 覆盖并且没有提示
cp ~/download/jenkins.war ./

# 复制指定目录中的所有文件(包括文件夹)到当前目录
cp ~/download/* ./

# 用比较友好的方式列出文件大小
ls -lh /home
# -t sort by modification time, newest first
# -S sort by file size, largest first
ls -lht

# 设置永久使用ll命令别名，可以将该命令写入~/.bashrc文件里面, 注销用户重新登录就可以使用了
echo "alias ll='ls -alht --color=auto'" >> ~/.bashrc

# 列出/dev下和vdb匹配的文件和目录
ll /dev | grep vdb
```

## tree命令的使用
```bash
# 安装(debian默认未安装)
sudo apt install tree

# 树形目录显示文件夹, 显示当前src文件夹中文件
tree ./src/

# 显示当前目录树形结构, 不带任何参数默认从当前目录开始展开树
tree
tree ./

# 显示树形目录及路径
tree -f
tree -f "$(pwd)"

# 只显示目录, 不显示文件
tree -d

# 指定目录层数
tree -L 3

# 列出文件权限
tree -ph ./

# 列出目录的文件大小
tree --du -h ./

# 显示帮助文档
tree --help
```


## 文件压缩解压
```bash

# 解压(当前目录中)tar.gz文件到指定文件夹, 当前用户目录的app目录(目录必须存在)中
tar -zxvf ./apache-tomcat-8.5.32.tar.gz -C ~/app/

# 解压当前目录中xxx.tar.gz压缩文件到当前目录
tar -zxvf xxx.tar.gz -C ./

# 打包并压缩, 打包当前目录下logs目录到当前文件夹的ff-1.tar.gz压缩包
tar -zcvf ./ff-1.tar.gz logs/


# 打包当前文件夹下所有文件到xxx.tar.gz(也是当前文件夹)
tar -zcvf xxx.tar.gz ./*

# 解压文件到指定目录
tar -zxvf /home/zip/xxx.tar.gz -C /home/www/tmp
```


## linux文件查找
```bash

# 搜索指定目录下超过指定大小的文件(详细显示文件的属主、属组、文件大小(M为单位))
find /home  -type f -size +100M  -print0 | xargs -0 ls -lh  | sort -nr

# 根据文件名查找文件
find /home -name mysqld.log
```


## linux磁盘命令
```bash

# 查找系统中的大目录(从大到小排序，取前5个)
du -hm / --max-depth=1 | sort -nr | head -5

# 查看目录大小
du -h /boot
du -hs /boot

# 显示磁盘剩余空间,单位M
df -BM

# 展示块设备（block devices）的信息，包括磁盘、分区和挂载点等
lsblk -f

# 显示设备的名称、大小和类型等信息
lsblk -o NAME,MAJ:MIN,RM,SIZE,TYPE,FSTYPE,FSVER,FSAVAIL,FSUSE%,MOUNTPOINTS

#进入磁盘，对磁盘进行分区
fdisk /dev/sdb

# 查看磁盘大小
fdisk -l |grep Disk
fdisk -l

# 为所有用户给目录添加读写执行权限,4r,2w,1x
chmod 777 /mnt/vdbxx/

# 格式化
mkfs.ext4 /dev/vdb1
mkfs.xfs -f /dev/vdb2
mkfs -t ext3 /dev/vdb1
mkfs -t ext4 /dev/vdb2

# 开始挂载分区：
mount /dev/vdb2 /mnt/vdbxx

# 取消挂载
umount /mnt/vdbxx

# 强行解除挂载
umount -l /home

# 查看系统有哪些挂载点可以直接使用mount命令
df -T 只可以查看已经挂载的分区和文件系统类型
df -h -T
df -hT
df -Th

# 查看内存使用情况
free -h
free -ht
```

## BT宝塔面板命令
```shell
# 一般用户启动前面加 sudo
#  停止
/etc/init.d/bt stop

#  启动
/etc/init.d/bt start

#  重启
/etc/init.d/bt restart

# 查看当前面板端口
cat /www/server/panel/data/port.pl
```

## linux下bash编码报错问题
```bash
# 安装dos2unix
apt install dos2unix

# 转换当前目录指定文件为unix格式
dos2unix ./jenkins-cmd.sh
```

---------------------
- [linux 添加环境变量 PATH](https://blog.csdn.net/hj1993/article/details/81570807)
