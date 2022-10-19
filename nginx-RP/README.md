# docker-nginx

port mapping

#### Description:

使用nginx做反向代理、负载均衡

**How it really works:**

This docker uses the official Nginx docker image based on Alpine Linux. I decided to use Alpine because it packs everything that you need on just 18MB. :sunglasses:
When `docker-compose up` is called via command line, it creates two volumes that is shared between the Docker container and host machine. 
The fist volume make possible you customize the Nginx config. That being said, it points `./config/default.conf` to `/usr/share/nginx/html` 
The second volume make possible you customize the Nginx index. That being said, it points `./nginx-html` to `/usr/share/nginx/html` 
The 3rd volume make possible you customize the Nginx logs. That being said, it points `./nginx-log` to `/var/log/nginx` 
The 4rd volume make possible you customize the Nginx logs. That being said, it points `./nginx-log` to `/var/log/nginx` 


```nginx
//默认域名
server {
    listen       80;
    server_name  localhost;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
        autoindex on;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}

//其他域名
server {
    listen       80;
    server_name  www.xxx.com;
    charset utf-8;

    location / {
        proxy_pass  http://172.18.0.1:5005; # 转发规则
        proxy_set_header Host $proxy_host; # 修改转发请求头，让8080端口的应用可以受到真实的请求
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        # proxy_pass http://192.168.10.50:9101; # 如果是转发到其它主机
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```

`./docker-compose.yml`

```yaml
version: '2'

services:
    web:
        build: .
        volumes:
            - ./config/default.conf:/etc/nginx/conf.d/default.conf
            - ./shared-folder:/usr/share/nginx/html
        ports:
- 8080:80
```

##### Lets see this working:

1) cd /home

2) `git clone https://github.com/alishwang/DevOps-Docker.git`

3) `cd nginx-RP`

4）修改nginx-config目录中的default.conf配置

5) 确认所需要反向代理服务的ip地址和端口，推荐使用docker自带网桥，docker network ls 查询，然后通过docker network inspect <name> 查询实际分配的ip地址；填写到xxx.conf配置文件中。 

2) `docker-compose build`

3) `docker-compose up -d`

Open your browser and try the address: `http://localhost:80`. You must see something like this:

![index of](config/img.png)

