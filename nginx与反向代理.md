```
proxy_pass
```


```
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forward-For $proxy_add_x_forwarded_for;

location / {
  proxy_pass http://localhost:8000;

}
```


反向代理配置：
