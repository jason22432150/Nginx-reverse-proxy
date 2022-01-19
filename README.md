# Nginx-reverse-proxy
Nginx reverse proxy
使用NGINX反向代理多個網域    (( 搞了我一整天  ╰（‵□′）╯

## 1. 安裝nginx
``` sh
sudo apt-get install nginx
```
查看版本
``` sh
sudo nginx -v
```
我這邊版本是顯示 
``` sh
nginx version: nginx/1.18.0 (Ubuntu)
```


## 2. 配置網站配置檔
正常來說，nginx的網站配置檔會在「/etc/nginx/sites-available」之下，並建一個Symbolic link至「/etc/nginx/sites-enabled」下。

所以會看到「/etc/nginx/nginx.conf」有一句「 include /etc/nginx/sites-enabled/*;」，因此可以為每個網站建立不同的配置檔。
```nginxconf
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
        worker_connections 768;
        # multi_accept on;
}

http {
        ...
        include /etc/nginx/sites-enabled/*;
        ...
}
```
這時候我們可以在「/etc/nginx/sites-available」建立一個用來放網站配置的配置檔
```diff 
+ 有負數個網域的話就從這邊開始重複
```
```diff 
- (( 更改  your-domain.name  為你的網址 (1處)
```
```sh
sudo vim /etc/nginx/sites-available/your-domain.name
```
然後打上
```diff 
- (( 更改  your-domain.name  為你的網址 (1處)
- 修改 proxy_pass http://localhost:3000; 後面的端口
```
```nginxconf
server {
            listen 80 default_server;
            listen [::]:80 default_server;

            root /var/www/html;
            index index.html index.htm index.nginx-debian.html;
    
            server_name http://your-domain.name;
    
            location /{
                    proxy_pass http://localhost:2095;
                    proxy_redirect   off;
                    proxy_set_header Host $host;
                    proxy_set_header X-Forwarded-Host $server_name;
                    proxy_set_header X-Real-IP $remote_addr;
                    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                    proxy_set_header X-Forwarded-Proto $http_x_forwarded_proto;
            }
}
```
然後建立軟連結至「/etc/nginx/sites-enabled」之下

```diff 
- (( 更改  your-domain.name  為你的網址 (2處)
```

```nginxconf
sudo ln -sf /etc/nginx/sites-available/your-domain.name /etc/nginx/sites-enabled/your-domain.name
```
測試配置是否正常，並重新啟動Nginx伺服器
```sh
sudo nginx -t
sudo systemctl restart nginx
```
