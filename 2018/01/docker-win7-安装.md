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


#### elk全套安装
- [doc](http://elk-docker.readthedocs.io)
- docker run --name pc0-elk-1 -d -p 15601:5601 -p 19200:9200 -p 19300:9300 -p 15044:5044 -it  sebp/elk
- 注意登录虚拟机修改：sysctl -w vm.max_map_count=262144

#### etcd v2
- [doc](https://coreos.com/etcd/docs/latest/v2/docker_guide.html)
- 注意 -name 和 initial-cluster 中的节点名称保持一致
```
docker run -d -p 14001:4001 -p 12380:2380 -p 12379:2379 \
 --name pc0-etcd-1 quay.io/coreos/etcd:v2.3.8 \
 -name etcd1 \
 -advertise-client-urls http://${HostIP}:2379,http://${HostIP}:4001 \
 -listen-client-urls http://0.0.0.0:2379,http://0.0.0.0:4001 \
 -initial-advertise-peer-urls http://${HostIP}:2380 \
 -listen-peer-urls http://0.0.0.0:2380 \
 -initial-cluster-token etcd-cluster-1 \
 -initial-cluster etcd1=http://${HostIP}:2380 \
 -initial-cluster-state new
```
```
docker run --name pc0-etcd-1 -d -p 12379:2379 quay.io/coreos/etcd:v2.3.8 -lis
ten-client-urls http://0.0.0.0:2379 -advertise-client-urls http://0.0.0.0:2379
```
