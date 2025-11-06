### 安装hashicorp/consul

1、安装镜像，运行容器

```
docker pull hashicorp/consul
docker run -d -p 8500:8500 -p 8300:8300 -p 8301:8301 -p 8302:8302 -p 8600:8600/udp hashicorp/consul consul agent -dev -client=0.0.0.0
```

2、设置开机自启

```
docker container update --restart=always 容器id
```

访问地址：http://localhost:8500/ui/dc1/services

3、通过dig命令测试consul，默认端口8600

```
dig @192.168.0.122 -p 8600 hashicorp/consul.service.consul SRV
```

