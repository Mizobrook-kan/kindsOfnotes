先安装[Docker](https://docs.docker.com/engine/install/)和[Docker Compose](https://docs.docker.com/compose/install/compose-plugin)，


创建安装目录

```
mkdir -p /root/data/docker_data/lsky-pro
cd /root/data/docker_data/lsky-pro
```

查看端口号是否被占用，输入：

```
lsof -i:7791
```

运行：

```
docker compose up -d
```


## 配置域名解析

在控制台添加Nginx Proxy Manager地址


查看docker容器内部地址：

```
ip addr show docker0
```



## 反向代理

```
location / {
   proxy_pass http://178.18.249.61:8123/;
   rewrite ^/(.*)$ /$1 break;
   proxy_redirect off;
   proxy_set_header Host $host;
   proxy_set_header X-Forwarded-Proto $scheme;
   proxy_set_header X-Real-IP $remote_addr;
   proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
   proxy_set_header Upgrade-Insecure-Requests 1;
   proxy_set_header X-Forwarded-Proto https;
}
```
