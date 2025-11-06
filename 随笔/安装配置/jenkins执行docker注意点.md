1. 添加Jenkins用户到docker用户组中

   ```
   grep docker /etc/group
   sudo usermod -a -G docker jenkins
   systemctl restart docker
   重启Jenkins
   ```

   

2. 在docker进行login时还会失败，执行以下命令

   ```
   /etc/docker/daemon.json
   # 添加这一行， IP地址和端口号是harbor中配置的
   "insecure-registries":["192.168.194.101:30002"]
   ```

   