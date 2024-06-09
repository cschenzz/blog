## windows下nginx的使用
下载nginx压缩包, 完成后解压, 下载地址：http://nginx.org/en/download.html 
下载
```bash
# 先在命令行中定位到nginx所在目录, 执行以下命令启动(或者直接双击nginx图标启动), 程序启动后可以在进程管理中看到nginx进程
.\nginx.exe
# start .\nginx.exe

# 重新加载配置文件
.\nginx.exe -s reload

# 关闭
.\nginx.exe -s stop

# 退出
.\nginx.exe -s quit
```

## nginx 二级路由部署前端项目
```bash
# =========================================
location ^~ /admin/ {
  alias "D:/app/cc/test-admin/dist/";
  index index.html;
  try_files $uri $uri/ /index.html;
}
# =========================================
```

## ruoyi前端部署配置
```bash
location / {
  root /home/ruoyi/projects/ruoyi-ui;
  index index.html index.htm;
  try_files $uri $uri/ /index.html;
}
```

## bt宝塔中配置ry-vue
```bash
location / {
  try_files $uri $uri/ /index.html;
  index  index.html index.htm;
}
```

---------------------
- [nginx官方文档](https://nginx.org/en/docs/)
- [nginx之location规则详解](https://mp.weixin.qq.com/s?__biz=MzIwMTA4ODQ3Mw==&mid=2453588385&idx=1&sn=47ce42f7775fe6b328d9ae9275ecb199&chksm=81392517b64eac01cac5329bc9b8f798046eefb22873396c819ba1e564662c828e06f002a9a1&scene=27)