# xtrabackup demo

## 环境准备

### 创建数据文件夹和备份文件夹

```shell
mkdir -p ~/GitCode/xtrabackup/mysql/data
mkdir -p ~/GitCode/xtrabackup/mysql/backup
```

### 制作MySQL:5.7运行环境

```shell

cd mysql-5.7

docker build -t mysqld-prod:5.7 .

```

### 制作xtrabackup-2.4.20运行环境

```shell

cd xtrabackup-2.4.20

docker build -t centos7.8_xtrabackup:v1 .

```

### 运行MySQL

```shell
docker run -d \
--name mysqld-prod \
--restart=always \
-e MYSQL_ROOT_PASSWORD=123456 \
-p 3307:3306 \
-v ~/GitCode/xtrabackup/mysql/data:/var/lib/mysql \
mysqld-prod:5.7 \
--character-set-server=utf8mb4 \
--collation-server=utf8mb4_unicode_ci
```

### 运行xtrabackup

```shell
docker run -it -d \
--name centos7.8_xtrabackup \
--restart=always \
-e TZ=Asia/Shanghai \
-v ~/GitCode/xtrabackup/mysql/data:/data \
-v ~/GitCode/xtrabackup/mysql/backup:/backup \
centos7.8_xtrabackup:v1
```

### 查看MySQL容器的IpAddress以及执行创建数据库插入/删除操作

执行插入或删除操作才能让MySQL生成bin-log文件

```shell
docker inspect mysqld-prod # 查看IPAddress
```


### 全量备份

> 启动xtrabackup备份

```shell

# 172.17.0.2 为上一步查看的MySQL容器IPAddress

innobackupex \
--user=root \
--password=123456 \
--port=3306 \
--host=172.17.0.2 \
--socket=/data/mysql.sock \
--datadir=/data /backup
```

> 执行prepare操作

```shell
xtrabackup --prepare --target-dir=/backup/2024-03-07_21-32-37
```

> 用于恢复MySQL

```shell
rsync -avrP /backup/2024-03-07_21-32-37 /var/lib/mysql/
```

> 修改MySQL文件夹的权限

```shell
chown -R mysql:mysql /var/lib/mysql
```