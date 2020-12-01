# LuckysheetServerStarter

简体中文 | [English](./README.md)

## 介绍
💻[LuckysheetServer](https://gitee.com/mengshukeji/LuckysheetServer/) docker部署启动模板。

## 演示
- [协同编辑Demo](http://luckysheet.lashuju.com/demo/)（注意：请大家别操作频繁，防止搞崩服务器）

## 教程

### 文件夹结构

```shell
C:\Users\Administrator\Desktop\lsheet>tree /f
│ 
│  docker-compose.yml   #部署文件
│
├─java-server
│      application-dev.yml           #项目配置
│      application.yml                  #项目配置 
│      web-lockysheet-server.jar   #项目
│
├─nginx
│  │  nginx.conf  #nginx配置
│  │
│  ├─html         #nginx静态文件夹
|  |  luckysheet_demo.html # Luckysheet demo
|  |
│  └─logs          #nginx日志文件夹
├─postgres
│  │  init.sql      #postgre初始化文件
│  │
│  └─data          #postgres数据文件夹
└─redis
    │  redis.conf   #redis配置文件
    │ 
    ├─data          #redis数据文件夹
    └─logs           #redis日志文件夹
```
### 准备工作

- 安装 [docker](https://docs.docker.com/get-docker/)

- 安装 [docker-compose](https://docs.docker.com/compose/)

- 安装 curl
    ```shell
    yum install curl
    ```

#### 1. 进入文件夹
![cd dir](https://img2020.cnblogs.com/blog/120439/202011/120439-20201130215031237-1786358850.png)

#### 2. 设置redis/logs目录权限
```shell
chmod a+rwx ./redis/logs/
```
![redis/logs](https://img2020.cnblogs.com/blog/120439/202011/120439-20201130215234833-651444535.png)

#### 3. 开始构建生成镜像
```shell
docker-compose build
```
![build](https://img2020.cnblogs.com/blog/120439/202011/120439-20201130215315502-317406069.png)

#### 4. 后台启动容器
```shell
docker-compose up -d 
```
![up](https://img2020.cnblogs.com/blog/120439/202011/120439-20201130215402429-317738862.png)

#### 5. 查看镜像
```shell
docker ps
```
![ps](https://img2020.cnblogs.com/blog/120439/202011/120439-20201130215652620-1451330894.png)

#### 6. 验证

- redis验证
```shell
docker exec -it  容器ID  redis-cli -a '123456'
```
![redis](https://img2020.cnblogs.com/blog/120439/202011/120439-20201130215842628-1770102098.png)

- postgres验证

```shell
#进入容器
docker exec -ti postgres /bin/bash
#登陆postgres
psql -U postgres
#列出全部数据库
\l
#切换数据库
\c luckysheetdb
#列出全部表名
\dt
#查看表数据
select * from luckysheet;
```
![postgres](https://img2020.cnblogs.com/blog/120439/202011/120439-20201130220215958-665241524.png)
![select](https://img2020.cnblogs.com/blog/120439/202011/120439-20201130220237119-760074998.png)


- 验证java应用（使用测试url）
```shell
curl http://172.19.0.4:9004/luckysheet/test/constant?param=123
```
![java](https://img2020.cnblogs.com/blog/120439/202011/120439-20201130220452697-2141439848.png)

- 验证nginx访问java应用
```shell
curl http://172.19.0.101/luckysheet/test/constant?param=123
```
![java nginx](https://img2020.cnblogs.com/blog/120439/202011/120439-20201130220550581-323083313.png)

- 本示例是安装在云服务器上，通过浏览器测试
```shell
http://xx.100.104.9/luckysheet/test/constant?param=123
```
![browser](https://img2020.cnblogs.com/blog/120439/202011/120439-20201130220737225-1998494518.png)

访问Luckysheet demo
```shell
＃ 静态demo 
http://xx.100.104.9 
＃ 协同编辑模式
http://xx.100.104.9?share
```

#### 7. 配置文件

- nginx配置

```shell
#运行用户
#user  nobody;

#开启进程数 <=CPU数
worker_processes  1;

#错误日志保存位置
error_log  /var/log/nginx/error.log;

#进程号保存文件
pid        /var/log/nginx/nginx.pid;

#等待事件
events {
    #Linux下打开提高性能
    #use epoll;
    #每个进程最大连接数（最大连接=连接数x进程数）
    worker_connections  1024;
}


http {
    #文件扩展名与文件类型映射表
    include       mime.types;
    
    #默认文件类型
    default_type  application/octet-stream;

    #日志文件输出格式 这个位置相于全局设置
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    #请求日志保存位置
    access_log  /var/log/nginx/access.log  main;

    #打开发送文件
    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #打开gzip压缩
    #gzip  on;
    gzip  on;
    gzip_min_length 1k;
    gzip_buffers 16 64k;
    gzip_http_version 1.0;
    gzip_comp_level 7;
    #DO NOT zip pics
    gzip_types text/plain application/x-javascript text/javascript application/x-httpd-php text/css text/xml text/jsp application/eot application/ttf application/otf application/svg application/woff;
    gzip_vary on;
    gzip_disable "MSIE [1-6].";

    #websocket
    map $http_upgrade $connection_upgrade {
        default upgrade;
        ''      close;
    }
    
    #设定负载均衡的服务器列表
    upstream luckysheetserver {
        server 172.19.0.4:9004 weight=1;  
    }
    
    #第一个虚拟主机
    server {
        #监听IP端口
        listen       80;
        
        #主机名
        server_name  localhost;
        
        #设置字符集
        #charset koi8-r;

        #本虚拟server的访问日志 相当于局部变量 
        #access_log  logs/host.access.log  main;
        
        proxy_set_header X-Forwarded-Host $host;
        proxy_set_header X-Forwarded-Server $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        location /luckysheet/websocket/luckysheet {
            proxy_pass http://luckysheetserver/luckysheet/websocket/luckysheet;

            proxy_set_header Host $host;
            proxy_set_header X-real-ip $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_connect_timeout 1800s;
            proxy_read_timeout 600s;
            proxy_send_timeout 600s;
            #websocket
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
        }
        location /luckysheet/ {
            proxy_pass http://luckysheetserver;

            proxy_connect_timeout 1800;
            proxy_read_timeout 600;
        }
         
        location / {
            root   /usr/share/nginx/html/;
            index  index.html index.htm luckysheet_demo;            
            
            proxy_connect_timeout 1800;
            proxy_read_timeout 600;
            #websocket
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
        }

        #动静分离
        location ~ .*\.(html|js|css|jpg|txt)?$ {
           root  /usr/share/nginx/html/;
           #expires 3d;
        }
        
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   /usr/share/nginx/html/;
        }

    }
}
```
- redis配置

```shell
bind *
protected-mode no
port 6379
tcp-backlog 511
timeout 0
tcp-keepalive 300
supervised no
pidfile /usr/local/redis/redis.pid
loglevel notice
logfile /usr/local/redis/logs.log
databases 16
save 900 1
save 300 10
save 60 10000
stop-writes-on-bgsave-error yes
rdbcompression yes
rdbchecksum yes
dbfilename dump.rdb
dir /usr/local/redis/data/
slave-serve-stale-data yes
repl-diskless-sync no
repl-diskless-sync-delay 5
repl-disable-tcp-nodelay no
slave-priority 100
maxmemory 500mb
maxmemory-policy noeviction
appendonly no
appendfilename "appendonly.aof"
appendfsync everysec
no-appendfsync-on-rewrite no
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
aof-load-truncated yes
lua-time-limit 5000
slowlog-log-slower-than 10000
slowlog-max-len 128
latency-monitor-threshold 0
notify-keyspace-events ""
hash-max-ziplist-entries 512
hash-max-ziplist-value 64
list-max-ziplist-size -2
list-compress-depth 0
set-max-intset-entries 512
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
hll-sparse-max-bytes 3000
activerehashing yes
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit slave 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60
hz 10
aof-rewrite-incremental-fsync yes
requirepass 123456
```
- postgres初始化脚本

```shell
CREATE DATABASE luckysheetdb;
\c luckysheetdb;

DROP SEQUENCE IF EXISTS "public"."luckysheet_id_seq";
CREATE SEQUENCE "public"."luckysheet_id_seq"
INCREMENT 1
MINVALUE  1
MAXVALUE 9999999999999
START 1
CACHE 10;

DROP TABLE IF EXISTS "public"."luckysheet";
CREATE TABLE "luckysheet" (
  "id" int8 NOT NULL,
  "block_id" varchar(200) COLLATE "pg_catalog"."default" NOT NULL,
  "index" varchar(200) COLLATE "pg_catalog"."default" NOT NULL,
  "list_id" varchar(200) COLLATE "pg_catalog"."default" NOT NULL,
  "status" int2 NOT NULL,
  "json_data" jsonb,
  "order" int2,
  "is_delete" int2
);
CREATE INDEX "block_id" ON "public"."luckysheet" USING btree (
  "block_id" COLLATE "pg_catalog"."default" "pg_catalog"."text_ops" ASC NULLS LAST,
  "list_id" COLLATE "pg_catalog"."default" "pg_catalog"."text_ops" ASC NULLS LAST
);
CREATE INDEX "index" ON "public"."luckysheet" USING btree (
  "index" COLLATE "pg_catalog"."default" "pg_catalog"."text_ops" ASC NULLS LAST,
  "list_id" COLLATE "pg_catalog"."default" "pg_catalog"."text_ops" ASC NULLS LAST
);
CREATE INDEX "is_delete" ON "public"."luckysheet" USING btree (
  "is_delete" "pg_catalog"."int2_ops" ASC NULLS LAST
);
CREATE INDEX "list_id" ON "public"."luckysheet" USING btree (
  "list_id" COLLATE "pg_catalog"."default" "pg_catalog"."text_ops" ASC NULLS LAST
);
CREATE INDEX "order" ON "public"."luckysheet" USING btree (
  "list_id" COLLATE "pg_catalog"."default" "pg_catalog"."text_ops" ASC NULLS LAST,
  "order" "pg_catalog"."int2_ops" ASC NULLS LAST
);
CREATE INDEX "status" ON "public"."luckysheet" USING btree (
  "list_id" COLLATE "pg_catalog"."default" "pg_catalog"."text_ops" ASC NULLS LAST,
  "status" "pg_catalog"."int2_ops" ASC NULLS LAST
);
ALTER TABLE "public"."luckysheet" ADD CONSTRAINT "luckysheet_pkey" PRIMARY KEY ("id");

INSERT INTO "public"."luckysheet" VALUES (nextval('luckysheet_id_seq'), 'fblock', '1', '1079500#-8803#7c45f52b7d01486d88bc53cb17dcd2c3', 1, '{"row":84,"name":"Sheet1","chart":[],"color":"","index":"1","order":0,"column":60,"config":{},"status":0,"celldata":[],"ch_width":4748,"rowsplit":[],"rh_height":1790,"scrollTop":0,"scrollLeft":0,"visibledatarow":[],"visibledatacolumn":[],"jfgird_select_save":[],"jfgrid_selection_range":{}}', 0, 0);
INSERT INTO "public"."luckysheet" VALUES (nextval('luckysheet_id_seq'), 'fblock', '2', '1079500#-8803#7c45f52b7d01486d88bc53cb17dcd2c3', 0, '{"row":84,"name":"Sheet2","chart":[],"color":"","index":"2","order":1,"column":60,"config":{},"status":0,"celldata":[],"ch_width":4748,"rowsplit":[],"rh_height":1790,"scrollTop":0,"scrollLeft":0,"visibledatarow":[],"visibledatacolumn":[],"jfgird_select_save":[],"jfgrid_selection_range":{}}', 1, 0);
INSERT INTO "public"."luckysheet" VALUES (nextval('luckysheet_id_seq'), 'fblock', '3', '1079500#-8803#7c45f52b7d01486d88bc53cb17dcd2c3', 0, '{"row":84,"name":"Sheet3","chart":[],"color":"","index":"3","order":2,"column":60,"config":{},"status":0,"celldata":[],"ch_width":4748,"rowsplit":[],"rh_height":1790,"scrollTop":0,"scrollLeft":0,"visibledatarow":[],"visibledatacolumn":[],"jfgird_select_save":[],"jfgrid_selection_range":{}}', 2, 0);
```
- docker-compose文件

```shell
version: "2"
services:

  nginx:
    image: nginx:latest
    #restart: always
    container_name: nginx
    environment:
      - TZ=Asia/Shanghai
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
      - ./nginx/logs:/var/log/nginx/
      - ./nginx/html:/usr/share/nginx/html/
      - /etc/localtime:/etc/localtime
    networks:
      extnetwork:
        ipv4_address: 172.19.0.101
  
  
  redis:
    image: redis:latest
    container_name: redis
    #restart: always
    environment:
      - TZ=Asia/Shanghai
    command: redis-server /usr/local/etc/redis/redis.conf --requirepass 123456
    ports:
      - "6379:6379"
    volumes:
      - ./redis/data:/usr/local/redis/data/
      - ./redis/logs:/usr/local/redis/
      - ./redis/redis.conf:/usr/local/etc/redis/redis.conf
      - /etc/localtime:/etc/localtime
    networks:
      extnetwork:
        ipv4_address: 172.19.0.2
      

  postgres:
    image: postgres:12
    #restart: always
    privileged: true 
    container_name: postgres 
    ports:
      - 5432:5432
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: 123456
      PGDATA: /var/lib/postgresql/data/pgdata
    volumes:
      - ./postgres/data:/var/lib/postgresql/data/pgdata
      - ./postgres/init.sql:/docker-entrypoint-initdb.d/init.sql
      - /etc/localtime:/etc/localtime
    networks:
      extnetwork:
        ipv4_address: 172.19.0.3  
      

  web-server:
    image: java:8
    #restart: always
    privileged: true  
    ports:
      - 9004:9004
    volumes:
      - ./java-server/web-lockysheet-server.jar:/usr/local/luckysheet-server/app.jar
      - ./java-server/application.yml:/usr/local/luckysheet-server/application.yml
      - ./java-server/application-dev.yml:/usr/local/luckysheet-server/application-dev.yml
      - /etc/localtime:/etc/localtime
    command: [
      'java',
      '-Xmx200m',
      '-jar',
      '/usr/local/luckysheet-server/app.jar',
      '--spring.config.location=/usr/local/luckysheet-server/application.yml,/usr/local/luckysheet-server/application-dev.yml'
    ]  
    networks:
      extnetwork:
        ipv4_address: 172.19.0.4

networks:
   extnetwork:
      ipam:
         config:
         - subnet: 172.19.0.0/16
           gateway: 172.19.0.1    
            
```


**注：**

```shell
# 进入容器
docker exec -ti 容器ID /bin/bash
# 进入容器内redis
docker exec -it 容器ID  redis-cli -a '123456'
# 登陆docker查看日志
docker logs --tail 300 -f 容器ID
# 设置文件夹权限
chmod a+rwx 目标
```

## 相关链接
- [Luckysheet官方文档](https://mengshukeji.gitee.io/LuckysheetDocs/)
- [Luckysheet如何把表格里的数据保存到数据库](https://www.cnblogs.com/DuShuSir/p/13857874.html)
- [docker-compose 部署 Nginx、Postgres、redis、java应用](https://www.cnblogs.com/xuchen0117/p/14064109.html)

## 贡献者和感谢

### 团队
- [@iamxuchen800117](https://github.com/iamxuchen800117)
- [@wpxp123456](https://github.com/wpxp123456)

## 版权信息
有关详细信息，请查阅附件的[LICENSE](./LICENSE)文件。原始作者保留Apache 2.0许可未明确授予的所有权利。
