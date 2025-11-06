### 创建docker的kong网络

```
 docker network create kong-net
```



### 安装postgresql和migrations

```
 docker run -d --name kong-database \
  --network=kong-net \
  -p 5432:5432 \
  -e "POSTGRES_USER=kong" \
  -e "POSTGRES_DB=kong" \
  -e "POSTGRES_PASSWORD=kongpass" \
  postgres:13
  
  
  
  
  docker run --rm --network=kong-net \
 -e "KONG_DATABASE=postgres" \
 -e "KONG_PG_HOST=kong-database" \
 -e "KONG_PG_PASSWORD=kongpass" \
 -e "KONG_PASSWORD=test" \
kong/kong-gateway:3.8.0.0 kong migrations bootstrap
```

### 安装kong

官方安装地址：https://docs.konghq.com/gateway/3.8.x/install/linux/rhel/?install=oss

通过yum安装示例

```
下载 Kong YUM 存储库：
curl -1sLf "https://packages.konghq.com/public/gateway-38/config.rpm.txt?distro=el&codename=$(rpm --eval '%{rhel}')" | sudo tee /etc/yum.repos.d/kong-gateway-38.repo
sudo yum -q makecache -y --disablerepo='*' --enablerepo='kong-gateway-38'
安装Kong：
sudo yum install -y kong-3.8.0
```

### 配置kong

关闭防火墙，重启docker -- 重要

```
systemctl stop firewalld.service
systemctl restart docker
```

```
cp /etc/kong/kong.conf.default /etc/kong/kong.conf
vi /etc/kong/kong.conf
# 修改如下内容
database = postgres
pg_host = 192.168.194.100 #这里是对外地址，填写部署所在服务器地址
pg_port = 5432
pg_timeout = 5000


pg_user = kong
pg_password = kongpass
pg_database = kong

dns_resolver = 192.168.194.100:8600 #配置consul的dns端口
admin_gui_listen = 192.168.194.100:8002, 192.168.194.100:8445 ssl
admin_listen = 192.168.194.100:8001 reuseport backlog=16384, 192.168.194.100:8444 http2 ssl reuseport backlog=16384
proxy_listen = 0.0.0.0:8000 reuseport backlog=16384, 0.0.0.0:8443 http2 ssl reuseport backlog=16384

```





### 初始化kong的数据库并启动

```
kong migrations bootstrap up -c /etc/kong/kong.conf  #初始化生成数据库
kong start -c /etc/kong/kong.conf

#添加防火墙规则 -- 如果开启了防火墙
firewall-cmd --zone=public --add-port=8002/tcp --permanent
firewall-cmd --zone=public --add-port=8000/tcp --permanent
sudo firewall-cmd --reload
```

看到这个代表启动成功

![](.\img\kong.jpg)

浏览器访问：http://192.168.194.100:8002/



### kong集成consul

前面已经配置了dns_resolver接下来如下配置即可

在**Gateway Service**里面将`host`改成`consul`里的`[Service Name].service.consul`将`port`改为`80`端口

![](F:\studyNote\随笔\安装配置\img\kong integration with consul.jpg)