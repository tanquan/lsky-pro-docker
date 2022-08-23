# LskyPro Docker 镜像

[![Build and push Docker images](https://github.com/cold-pig/LskyPro-Docker/actions/workflows/update-docker.yaml/badge.svg)](https://github.com/cold-pig/LskyPro-Docker/actions/workflows/update-docker.yaml)

每天自动拉取最新代码构建 Docker 镜像。

## 版本：

| 名称                   | 版本    | 说明  |
| ---------------------- | ------- | ----- |
| coldpig/lskypro-docker | v_2.1 | amd64 |
| coldpig/lskypro-docker | v_2.0.4 | amd64 |


## 使用方法

### 方法1: docker cli 运行容器

```docker
docker run -d \
    --name lskypro \
    --restart unless-stopped \
    -p 9080:80 \
    -v /path-to-data:/var/www/html \
    coldpig/lskypro-docker:latest
```

### 方法2: docker-compose 部署

使用 `MySQL` 来作为数据库的话可以参考原项目 [#256](https://github.com/lsky-org/lsky-pro/issues/256) 来创建 `docker-compose.yaml` ，参考内容如下：

> 说明：增加 adminer 和 redis: 
>
> 1. adminer： 用于方便管理 MySQL 数据库。
>
> 2. redis： 可用于配置缓存。 
>
>    [配置缓存的方法](https://docs.lsky.pro/docs/v2/advanced/cache.html#%E4%BD%BF%E7%94%A8-redis)
>
>    [redis.conf 简单的单实例配置文件](https://github.com/cold-pig/LskyPro-Docker/blob/master/redis.conf)



```yaml
---
version: '3'
services:
  lskypro:
    image: coldpig/lskypro-docker:latest
    restart: unless-stopped
    hostname: lskypro
    container_name: lskypro
    volumes:
      - /data/lsky_pro/html:/var/www/html
    ports:
      - "9080:80"

  lskypro_mysql:
    image: mysql:5.7.37
    restart: unless-stopped
    hostname: lskypro_mysql
    container_name: lskypro_mysql
    # 修改加密规则
    command: --default-authentication-plugin=mysql_native_password
    volumes:
      - /data/lsky_pro/mysql/data:/var/lib/mysql
      - /data/lsky_pro/mysql/conf:/etc/mysql
      - /data/lsky_pro/mysql/log:/var/log/mysql
    environment:
      MYSQL_ROOT_PASSWORD: MyPassword  # 数据库root用户密码（请自行修改密码）
      MYSQL_DATABASE: lskypro  # 给 lsky-pro 用的数据库名称

  lskypro_adminer:
    image: adminer:latest
    container_name: lskypro_adminer
    hostname: lskypro_adminer
    restart: unless-stopped
    ports:
      - 18080:8080
    environment:
      - ADMINER_DEFAULT_SERVER=lskypro_mysql

  lskypro_redis:
    image: redis:latest
    container_name: lskypro_redis
    hostname: lskypro_redis
    restart: unless-stopped
    ports:
      - "6379:6379"
    volumes:
      - /data/lsky_pro/redis/redis.conf:/etc/redis/redis.conf 
      - /data/lsky_pro/redis/data:/data
    command: redis-server /etc/redis/redis.conf
```


## 反代 HTTPS

如果使用了 Nginx 反代后，如果出现无法加载图片的问题，可以根据原项目 [#317](https://github.com/lsky-org/lsky-pro/issues/317) 执行以下指令来手动修改容器内`AppServiceProvider.php`文件对于 HTTPS 的支持。

***Tips：将 lskypro 改为自己容器的名字***

```bash
docker exec -it lskypro sed -i '32 a \\\Illuminate\\Support\\Facades\\URL::forceScheme('"'"'https'"'"');' /var/www/html/app/Providers/AppServiceProvider.php
```


------
原项目：[☁️兰空图床 (Lsky Pro) - Your photo album on the cloud.](https://github.com/lsky-org/lsky-pro)

根据 [HalcyonAzure/lsky-pro-docker](https://github.com/HalcyonAzure/lsky-pro-docker) 学习使用 Actions。 

