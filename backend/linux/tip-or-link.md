
## 文章
- [破解 Linux 文件安放之谜：哪里才是绝佳文件归宿？](https://www.linuxmi.com/linux-files-where.html)
- [安装 Debian 12 后有 10 件必做的事情](https://www.linuxmi.com/debian-12-installation-do.html)
- [驾驭 Linux 超强 source 命令](https://www.linuxmi.com/linux-source-command-2.html)
- [配置 Linux 环境变量的 6 种方法](https://www.linuxmi.com/linux-environment-variables.html)
- [如何以24小时格式显示KDE锁屏时间](https://ubuntu.dovov.com/13535/%E5%A6%82%E4%BD%95%E4%BB%A524%E5%B0%8F%E6%97%B6%E6%A0%BC%E5%BC%8F%E6%98%BE%E7%A4%BAkde%E9%94%81%E5%B1%8F%E6%97%B6%E9%97%B4.html)

## debian中常用软件安装
1. redis: 先执行`sudo apt update`后执行`sudo apt install redis-server`
2. 安装java环境: 参考java环境章节
3. node.js: `sudo apt install nodejs`
4. npm: `sudo apt install npm`
5. 查看node.js, npm, python版本: `node -v`, `npm -v`, `python3 --version`, python已默认安装3.x版本
6. 安装nginx, `sudo apt install nginx`, 安装后使用`nginx -v`查看版本
7. fastfetch: `sudo apt install fastfetch`, 现代化的系统信息工具(类似 neofetch)
8. 安装docker: 参考docker官方网站进行安装
9. 在docker中安装`mysql8.x, rabbitmq, minio, jenkins, gitea`等

## debian12服务器环境安装
```bash
# 1.安装宝塔(或者https://1panel.cn/)
wget -O install.sh https://download.bt.cn/install/install-ubuntu_6.0.sh && sudo bash install.sh ed8484bec

# 2.宝塔里安装mysql, nginx. docker在bt或命令行安装均可

# 3.通过终端命令安装nginx, redis, docker, java环境
sudo apt install openjdk-17-jdk
# 安装文件上传下载工具(服务器上安装), 配合深度终端实现服务器上文件的上传下载
sudo apt install lrzsz

# 从服务器下载文件到本地, 将选定的文件发送(send)到本地机器
sz --version
# 输入以下命令后会提示选择保存下载文件的文件夹, 确认后开始下载文件
sz remote_dir/file.txt

# 从本地上传文件到服务器(receive), 直接输入rz, 提示选择本地需要上传的文件, 选择文件后将上传文件到服务器当前运行命令所在的文件夹
rz --version

# 4.使用docker安装jenkins, rabbitMq, gitea, kafka, prometheus等
```

## debian常用命令及软件安装
```bash
sudo apt install curl
sudo apt install git
# vim, btop, htop, wget, neofetch, vlc, flatpak同样方式安装

sudo apt install redis-server
sudo apt install openjdk-17-jre openjdk-17-jdk
sudo apt install apt-transport-https lsb-release ca-certificates curl dirmngr gnupg


# -----------------------------------------------------
# 查看系统已安装的软件(无需sudo)
apt list --installed

# 在我们安装任何软件之前，通过在终端中运行以下命令来更新本地索引
sudo apt update
# 查看哪些软件可以更新
apt list --upgradable
# 全部更新
sudo apt upgrade
# 更新升级指定软件
sudo apt install --only-upgrade firefox-esr

# 更新Debian系统
sudo apt update && sudo apt full-upgrade

# 显示软件包具体信息
apt show -a firefox-esr

# 删除软件包命令
sudo apt remove <package_name>

# 清理不再使用的依赖和库文件
sudo apt autoremove

# 移除软件包及配置文件
sudo apt purge <package_name>


# -----------------------------------------------------
# deb软件包直接双击调用系统安装器安装(或者sudo dpkg -i xx.deb)
# 安装后要卸载需要先知道软件名字(和安装的deb包名字不一定一样)
# 先搜索找到软件名称(这里以qq为例)
sudo dpkg -l | grep "qq"

# 知道名称后使用命令卸载(-r参数)
sudo dpkg -r linuxqq

# --------------------------------
# 卸载通过apt install安装的软件(比如vlc, 主要是需要知道软件名称)
sudo apt autoremove vlc
```

## 系统升级
```bash
# 检查当前Debian系统版本
cat /etc/debian_version

# 了解Debian发行版详情
lsb_release -d
lsb_release -a
cat /etc/os-release


# ①依次执行下面命令
sudo apt update
sudo apt upgrade
sudo apt full-upgrade
sudo apt --purge autoremove

# 重启
sudo reboot

# 更新 sources.list 文件, 把 bullseye 替换成 bookworm
sudo sed -i 's/bullseye/bookworm/g' /etc/apt/sources.list

# 再执行上面①命令

# 升级发行版软件包, -y: 自动输入yes
sudo apt dist-upgrade -y

# 查看内核版本
uname -rms

# 删除过时的软件包
sudo apt --purge autoremove
```

## 娱乐类工具
```bash
# ----1-----
# cowsay(命令界面的各种图案, 默认一头牛)
# 安装
sudo apt install cowsay

# 说一句话
cowsay 'hello world!'

# 查看别的动物
cowsay -l

# -f设置动物参数
cowsay -f duck hello


# ----2-----
# cmatrix(类似黑客帝国的代码雨)
# bastet(命令行下的俄罗斯方块)
# hollywood(模拟黑客的炫酷终端界面)
# asciiquarium(命令行下的模拟钓鱼程序)
```

## 小提示
- 查看软件/命令版本号: 软件/命令 + `--version`
- 命令前加`sudo`提升权限执行
- 退出(btop, htop, top, git log等退出): q
- 启动btop后快捷键会以橙色显示, esc打开菜单, 可以查看帮助文档
- `命令 --help`, `man 命令`查看命令的帮助文档, 如`ls --help`, `man ls`

## flatpak使用
```bash
# flatpak安装目录, /var/lib/flatpak
# 添加flathub官方库
flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo

# 设置flatpak国内镜像地址
sudo flatpak remote-modify flathub --url=https://mirror.sjtu.edu.cn/flathub

# 安装firefox
flatpak install flathub org.mozilla.firefox
# 安装其他一样(com.google.Chrome, com.microsoft.Edge, com.visualstudio.code, io.dbeaver.DBeaverCommunity)

# 命令运行程序
flatpak run org.mozilla.firefox

# 查看已安装的应用程序和运行时环境
flatpak list

# 只查看已安装的应用
flatpak list --app

# 列出真正运行的应用, Enumerate running instances
flatpak ps

# Show history
flatpak history

# 列出可安装的应用程序，这里以 flathub 为例
flatpak remote-ls flathub --app

# 更新所有的 Flatpak 应用程序
flatpak update

# 更新指定的 Flatpak 应用程序
flatpak update org.mozilla.firefox

# 查看已安装应用程序的详细信息：
flatpak info org.mozilla.firefox

# 删除一个 Flatpak 应用程序, 这里以 firefox 为例
sudo flatpak uninstall org.mozilla.firefox

# 如果你需要更多信息，可以参考 Flatpak 的帮助。
flatpak --help
```
## 🍄站点导航
- [清华大学镜像站](https://mirrors.tuna.tsinghua.edu.cn/)
- [上海交通大学镜像站](https://mirror.sjtu.edu.cn/)

---------------------
- [linux迷](https://www.linuxmi.com/)
- [Debian官网](https://www.debian.org/index.zh-cn.html) | [Kali Linux](https://www.kali.org/) | [Alpine Linux](https://www.alpinelinux.org/)
- [Flatpak安装](https://flatpak.org/setup) | [文档](https://docs.flatpak.org/zh_CN/latest/getting-started.html) | [官方仓库](https://flathub.org/)
- [使用Systemctl命令来管理系统服务](https://zhuanlan.zhihu.com/p/388897743)
