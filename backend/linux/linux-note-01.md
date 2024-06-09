## 在debian12中安装ssh服务
```bash
# 在debian 12里一开始安装的时候没有开启ssh服务, 后面怎么手动安装并开启ssh服务

# 1. 安装 OpenSSH 服务器
sudo apt update
sudo apt install openssh-server

# 2. 启动ssh服务
sudo systemctl start ssh

# 3. 设置 SSH 服务开机自动启动
sudo systemctl enable ssh

# 检查 SSH 服务状态，确认是否已成功启动
sudo systemctl status ssh
```