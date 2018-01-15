### 关于安装
- win10之前只能用docker tool box
- 修改虚拟机目录需要在quick start之前修改start.sh文件：加入` export MACHINE_STORAGE_PATH="E:\VMs\docker-machine" `配置
- 第一次下载boot2docker.iso失败，直接将安装目录下的boot2docker.iso复制到cache目录即可

### Containers
#### Mysql
- [doc](https://store.docker.com/images/mysql)
-  docker run --name pc0-mysql-1 -e MYSQL_ROOT_PASSWORD=123456 -p 13306:3306 -d mysql:5.7

#### Redis
- [doc](https://store.docker.com/images/redis)
- docker run --name pc0-redis-1 -p 16379:6379 -d redis
