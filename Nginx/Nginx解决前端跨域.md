#### 如何通过Nginx解决前端跨域问题？

##### 一，安装Nginx

直接下载windows版本的Nginx安装包，启动`nginx`,



##### 二，修改Nginx配置文件

配置文件如下：监听本地开发端口，转发给服务器

```nginx
server {
        listen       22222;
        server_name  localhost;
        location  / {
            if ($request_method = 'OPTIONS') {
                add_header Access-Control-Allow-Origin 'http://localhost:8080';
                add_header Access-Control-Allow-Headers '*';
                add_header Access-Control-Allow-Methods '*';
                add_header Access-Control-Allow-Credentials 'true';
                return 204;
            }
            if ($request_method != 'OPTIONS') {
                add_header Access-Control-Allow-Origin 'http://localhost:8080' always;
                add_header Access-Control-Allow-Credentials 'true';
            }
            proxy_pass  http://localhost:59200; 
        }
    }
```

