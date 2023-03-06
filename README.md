# about-vps-deploy
> 免责声明：该项目所涉及软件及代码均为网上搜集整理，仅为个人笔记，若涉及您的版权，请联系本人删除。

# 安装x-ui：

原版安装：
```
bash <(curl -Ls https://raw.githubusercontent.com/vaxilu/x-ui/master/install.sh)
```
多功能版安装：
```
bash <(curl -Ls https://raw.githubusercontent.com/FranzKafkaYu/x-ui/master/install.sh)
```

# 安装nginx：
apt install nginx

# 安装acme：
curl https://get.acme.sh | sh

# 添加软链接：
ln -s  /root/.acme.sh/acme.sh /usr/local/bin/acme.sh

# 切换CA机构：
acme.sh --set-default-ca --server letsencrypt

# 申请证书： 
acme.sh  --issue -d cloudfreeosk.cf -k ec-256 --webroot  /var/www/html

# 安装证书：
acme.sh --install-cert -d example.com --ecc --key-file       /etc/x-ui/server.key  --fullchain-file /etc/x-ui/server.crt --reloadcmd     "systemctl force-reload nginx"

# 安装cloudreve：
cd /usr/bin

wget https://github.com/cloudreve/Cloudreve/releases/download/3.5.3/cloudreve_3.5.3_linux_amd64.tar.gz

# 解压获取到的主程序：
tar -zxvf cloudreve_3.5.3_linux_amd64.tar.gz

# 赋予执行权限：
chmod +x ./cloudreve

# 启动 Cloudreve
./cloudreve

#结束Cloudreve安装
Ctrl+C

# 编辑配置文件
vim /usr/lib/systemd/system/cloudreve.service    #输入i开始编辑

# 配置文件中内容:
```
[Unit]
Description=Cloudreve
Documentation=https://docs.cloudreve.org
After=network.target
After=mysqld.service
Wants=network.target

[Service]
WorkingDirectory=/usr/bin
ExecStart=/usr/bin/cloudreve
Restart=on-abnormal
RestartSec=5s
KillMode=mixed

StandardOutput=null
StandardError=syslog

[Install]
WantedBy=multi-user.target
```

# 保存配置文件并退出
Esc :wq回车

# 更新配置
systemctl daemon-reload

# 启动服务
systemctl start cloudreve

# 设置开机启动
systemctl enable cloudreve

# 设置XUI：
输入域名:9999登录，XRAY切换最新版本。入站列表添加节点，监听IP：127.0.0.1，端口10000,传输:ws。添加。
面板设置：监听IP：127.0.0.1，面板 url 根路径：/UUID-xui
保存重启。

# 配置nginx
vim /etc/nginx/nginx.conf#输入i开始编辑

# 配置文件内容：
```
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
    worker_connections 1024;
}

http {
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;

    include /etc/nginx/mime.types;
    default_type application/octet-stream;
    gzip on;

    server {
        listen 443 ssl;
        
        server_name example.com;  #你的域名
        ssl_certificate       /etc/x-ui/server.crt;  #证书位置
        ssl_certificate_key   /etc/x-ui/server.key; #私钥位置
        
        ssl_session_timeout 1d;
        ssl_session_cache shared:MozSSL:10m;
        ssl_session_tickets off;
        ssl_protocols    TLSv1.2 TLSv1.3;
        ssl_prefer_server_ciphers off;

        location / {
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	    proxy_set_header Host $http_host;
	    proxy_redirect off;
	    proxy_pass http://127.0.0.1:5212;
		
		# 如果您要使用本地存储策略，请将下一行注释符删除，并更改大小为理论最大文件尺寸
	     client_max_body_size 20000m;
        }


        location /UUID {   #分流路径
            proxy_redirect off;
            proxy_pass http://127.0.0.1:10000; #Xray端口
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
        
        location /UUID-xui {   #xui路径
            proxy_redirect off;
            proxy_pass http://127.0.0.1:9999;  #xui监听端口
            proxy_http_version 1.1;
            proxy_set_header Host $host;
        }
    }

    server {
        listen 80;
        location /.well-known/ {
               root /var/www/html;
            }
        location / {
                rewrite ^(.*)$ https://$host$1 permanent;
            }
    }
}
```

# 保存配置文件并退出
Esc :wq回车


# 重新加载nginx（每次编辑后都要进行）
systemctl reload nginx

# 设置开机启动：
systemctl enable nginx

# 查验运行状态：
systemctl status nginx

# 致谢
- [vaxilu/x-ui](https://github.com/vaxilu/x-ui)
- [FranzKafkaYu/x-ui](https://github.com/FranzKafkaYu/x-ui)
- [cloudreve/docs](https://github.com/cloudreve/docs)
- [不良林](https://bulianglin.com)
