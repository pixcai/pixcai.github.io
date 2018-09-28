---
title: Nginx部署单页应用
date: 2018-09-27 15:22:36
tags: [nginx, react, vue]
---
**React**或**Vue**应用编译为静态文件后用**Nginx**访问的话，需要给**Nginx**添加以下配置：
```
server {
  listen      80;
  server_name localhost;

  root /usr/share/nginx/html;

  location / {
    try_files $uri $uri/ @router;
  }

  location ^~ /api/ {
    proxy_pass http://localhost:8080;
  }

  location @router {
      rewrite ^.*$ /index.html break;
  }

  error_page 500 502 503 504  /50x.html;
  location = /50x.html {
    root html;
  }
}
```
