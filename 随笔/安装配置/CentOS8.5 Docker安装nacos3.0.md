###dokcer安装nacos

1、下载镜像

```
docker pull nacos/nacos-server
```

2、创建存储nacos数据信息的目录

```
mkdir /data/docker/nacos -p
```

>-p 作用是在创建多级文件时，不存在某一级文件就会自动创建，存在就使用原文件

3、nacos配置文件获取

 - 1 提前运行一个nacos容器。

   ```
   docker  run -d  --name nacos  \
   -p 8848:8848  \
   -p 9848:9848  \
   -p 9849:9849  \
   --privileged=true  \
   --restart=always  \
   -v /etc/localtime:/etc/localtime:ro  \
   -e TZ=Asia/Shanghai \
   -e LANG=en_US.UTF-8 \
   -e JVM_XMS=256m  \
   -e JVM_XMX=256m  \
   -e MODE=standalone  \
   nacos/nacos-server:latest
   ```

- 2 把nacos容器中的配置文件拷贝到宿主机中对应的目录下

  ```
  docker cp nacos:/home/nacos/conf /data/docker/nacos/conf
  ```

- 3 停止&删除容器

  ```
  docker rm -f nacos
  ```

- 4 修改 `application.properties` 配置文件

  ```
  vim /data/docker/nacos/conf/application.properties
  ```

  修改以下内容

  ```
  # 开启鉴权功能
  nacos.core.auth.enabled=true
  # 关闭使用user-agent判断服务端请求并放行鉴权的功能
  nacos.core.auth.enable.userAgentAuthWhite=false
  # 配置自定义身份识别的key和value，这两个属性是auth的白名单，用于标识来自其它服务器的请求。
  nacos.core.auth.server.identity.key=authKey
  nacos.core.auth.server.identity.value=shigzh
  # 自定义用于生成JWT令牌的密钥，注意：原始密钥长度不得低于32字符，且一定要进行Base64编码，否则无法启动节点。
  nacos.core.auth.plugin.nacos.token.secret.key=bmFjb3NfMjAyNDAxMTBfc2hpZ3poX25hY29zX3Rva2Vu
  # 权限缓存开关，开启后权限缓存的更新默认有15秒的延迟，默认 : false
  nacos.core.auth.caching.enabled=true
  # 数据库相关配置
  db.url.0=jdbc:mysql://${MYSQL_SERVICE_HOST:sgz.wz}:${MYSQL_SERVICE_PORT:3306}/${MYSQL_SERVICE_DB_NAME}?${MYSQL_SERVICE_DB_PARAM:characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useSSL=false}
  db.user.0=${MYSQL_SERVICE_USER:root}
  db.password.0=${MYSQL_SERVICE_PASSWORD:root@12345678}
  ```

4、初始化nacos数据库

		- 1 用navicat创建数据库，库名为nacos
  - 2 获取数据库脚本文件，用以下方式都可以
    - 通过nacos容器目录`/home/nacos/conf`下获取数据库脚本文件mysql-schema.sql。
    - 通过宿主机目录`/data/docker/nacos/conf`下获取数据库脚本文件mysql-schema.sql。
    - 直接通过 https://www.cnblogs.com/shigzh/p/17941250 获取数据库脚本文件。
- 3 执行数据库脚本文件

5、运行容器并挂载nacos配置信息到宿主机

```
docker run -d --name nacos \
    -e MODE=standalone \
    -e TZ=Asia/Shanghai \
    -e LANG=en_US.UTF-8 \
    -e JVM_XMS=256m \
    -e JVM_XMX=256m \
    -v /data/docker/nacos/logs/:/home/nacos/logs \
    -v /data/docker/nacos/data/:/home/nacos/data \
    -v /data/docker/nacos/conf/:/home/nacos/conf \
    -v /etc/localtime:/etc/localtime:ro \
    -e NACOS_AUTH_TOKEN=bmFjb3NfMjAyNTA4MDVfc2V0a2luZ19uYWNvc190b2tlbg== \
    -e NACOS_AUTH_IDENTITY_KEY=identity_key \
    -e NACOS_AUTH_IDENTITY_VALUE=setking \
    -p 8080:8080 \
    -p 8848:8848 \
    -p 9848:9848 \
    -d nacos/nacos-server:latest
```

3、部署访问地址：http://localhost:8080/index.html 

>参考：https://nacos.io/docs/v3.0/quickstart/quick-start-docker/?spm=5238cd80.6a33be36.0.0.10651e5dFZrwzM