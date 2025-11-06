安装指南：

### 1、 准备工作

#### 创建本地挂载目录

```
sudo mkdir -p /data/elasticsearch/{data,config,plugins}
sudo chmod -R 777 /opt/elasticsearch  # 确保权限（生产环境建议精细化权限控制）
```

#### 开放防火墙端口

```
sudo firewall-cmd --add-port={9200,9300}/tcp --permanent
sudo firewall-cmd --reload
```

### 2、安装Elasticsearch8.19

#### 设置网络

```
docker network create elastic
```



#### 获取镜像

```
docker pull docker.elastic.co/elasticsearch/elasticsearch:8.19.0
```



#### 首次运行容器（生成配置文件）

```
docker run  --name es-temp --net elastic -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -it  docker.elastic.co/elasticsearch/elasticsearch:8.19.0
```

#### 复制配置文件到本地

```
docker cp es-temp:/usr/share/elasticsearch/config /data/elasticsearch/
docker stop es-temp && docker rm es-temp
```

#### 正式启动容器挂载目录

```
docker run --name es-temp --net elastic -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -v /data/elasticsearch/config:/usr/share/elasticsearch/config -v /data/elasticsearch/data:/usr/share/elasticsearch/data -v /data/elasticsearch/plugins:/usr/share/elasticsearch/plugins -d docker.elastic.co/elasticsearch/elasticsearch:8.19.0
```

#### 启动容器后设置密码

```
docker exec -it es-temp sh #进入容器
bin/elasticsearch-setup-passwords auto #设置密码
bin/elasticsearch-reset-password -u elastic #获取elastic账号密码
bin/elasticsearch-create-enrollment-token --scope kibana #获取kibana的token
```

### 3、安装及运行 Kibana

#### 获取镜像

```
docker pull docker.elastic.co/kibana/kibana:8.19.0
```

#### 运行容器

```
docker run --name kib-01 --net elastic -p 5601:5601 docker.elastic.co/kibana/kibana:8.19.0
```

#### 后记

**设置容器开机自启动 **

```azure
docker container update --restart=always 容器id
```

**Linux系统需设置（重启会恢复）**

```
sudo sysctl -w vm.max_map_count=262144
```

**永久设置 `vm.max_map_count`可通过`/etc/sysctl.conf` 文件**

```
sudo vim /etc/sysctl.conf
#末尾写入以下内容
vm.max_map_count=262144 
```