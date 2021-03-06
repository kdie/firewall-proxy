# 准备工作
你需要拥有一个自己的**域名**，并**已经将域名解析至你的服务器**   
# 配置环境
硬件 : 内存 ≧ 512M 储存 ≧ 5G | 64位系统      

软件 : Debian 9/10 && Ubuntu 16/18/20
# 配置内容
- 安装基础工具  
```bash
apt update && apt install -y socat wget git vim     
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```
- 安装证书生成脚本  
```bash
wget -qO- get.acme.sh | bash 
source ~/.bashrc
```
- 安装证书  (**your_domain.com** 改为你的域名）
```bash
acme.sh --issue --standalone -d your_domain.com -k ec-256
mkdir -p /etc/nginx/conf.d /etc/v2ray
acme.sh --installcert -d your_domain.com --fullchain-file /etc/v2ray/server.pem --key-file /etc/v2ray/server.key --ecc
```
- 安装 Docker && Nginx && V2ray 
```bash
wget -qO- get.docker.com | bash
docker pull nginx
docker pull teddysun/v2ray
docker pull containrrr/watchtower
```
- 编辑 v2ray 配置 
```bash
vim /etc/v2ray/config.json
```
- 复制配置  
```bash
{
  "inbounds": [
    {
      "port": 443,
      "protocol": "vless",
      "settings": {
        "clients": [
          {
            "id": "b831381d-6324-4d53-ad4f-8cda48b30866",  #更改id
            "level": 0,
            "email": "your@email.com"  #更改为你的邮箱
          }
        ],
        "decryption": "none",
        "fallback":{
        "addr": "127.0.0.1",
        "port": 80
        }
      },
      "streamSettings": {
        "network": "tcp",
        "security": "tls",
        "tcpSettings": {
        "type": "none"
        },
        "tlsSettings": {
          "serverName": "your_domain.com",  #改为你的域名
          "allowInsecure": false,
          "alpn": [
          "http/1.1"
          ],
          "certificates": [
            {
              "certificateFile": "/etc/v2ray/server.pem",
              "keyFile": "/etc/v2ray/server.key"
            }
          ]
       }
     }
   }
  ],
  "outbounds": [
    {
      "protocol": "freedom",
      "settings": {}
    }
  ]
}
```
- 修改 Nginx 配置 
```bash
vim /etc/nginx/conf.d/default.conf
```
- 复制配置  
```bash
server {
    listen 127.0.0.1:80;
    server_name your_domain.com;  #改为你的域名
    location / {
        proxy_pass https://proxy.com;  #改为你想伪装的网址
        proxy_redirect     off;
        proxy_buffer_size          64k; 
        proxy_buffers              32 32k; 
        proxy_busy_buffers_size    128k;  
    }
}
server {
    listen 127.0.0.1:80;
    server_name ip.ip.ip.ip; #改为你服务器的 IP 地址
    return 301 https://your_domain.com$request_uri;  #改为你的域名
}
server {
    listen 0.0.0.0:80;
    listen [::]:80;
    server_name _;
    return 301 https://$host$request_uri;
}
```
- 启动服务 
```bash 
docker run --network host --name v2ray -v /etc/v2ray:/etc/v2ray --restart=always -d teddysun/v2ray
docker run --network host --name nginx -v /etc/nginx/conf.d:/etc/nginx/conf.d --restart=always -d nginx
docker run --name watchtower -v /var/run/docker.sock:/var/run/docker.sock --restart unless-stopped -d containrrr/watchtower --cleanup
```
- 开启 BBR 加速 
```bash
bash -c 'echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf'
bash -c 'echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf'
sysctl -p
```
# 更新软件
使用这种配置方式后，**watchtower**会自动监测并更新软件，你无需手动更新

# 客户端
Android系统: [点击下载](https://github.com/2dust/v2rayNG/releases)    

Windows && Linux && MacOS : [Qv2ray 下载](https://github.com/Qv2ray/Qv2ray/releases)   

Qv2ray 用法 : [文档](https://qv2ray.net/getting-started/step2.html) 