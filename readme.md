## Install Dependency
``` sh
sudo apt-get update -y
sudo apt-get install --no-install-recommends -y ruby rubygems gettext make g++ build-essential libc6-dev git
sudo apt-get clean -y
sudo gem install redis
```

## Install Redis
``` sh
git clone -b 4.0.0 https://github.com/antirez/redis.git
cd redis
sudo make install
sudo REDIS_PORT=7000 \
 		 REDIS_CONFIG_FILE=/etc/redis/7000.conf \
 		 REDIS_LOG_FILE=/var/log/redis_7000.log \
 		 REDIS_DATA_DIR=/var/lib/redis/7000 \
 		 REDIS_EXECUTABLE=`command -v redis-server` ./utils/install_server.sh
```

## Edit Config
/etc/redis/7000.conf
* change follownings
```
bind 10.222.0.6
protected-mode no
appendonly yes
```
* uncomment and change followings
```
cluster-enabled yes
cluster-config-file nodes_7000.conf
cluster-node-timeout 15000
```

## Service
``` sh
sudo service redis_7000 start
sudo service redis_7001 start
sudo service redis_7002 start
```

## Cluster Command
* Create Cluster
``` sh
cd redis/src
./redis-trib.rb create --replicas 0 10.222.0.6:7000 10.222.0.6:7001 10.222.0.6:7002
```
* Add Master Node
``` sh
cd redis/src
./redis-trib.rb add-node 10.222.0.6:7001 10.222.0.6:7000
./redis-trib.rb add-node 10.222.0.6:7002 10.222.0.6:7000
```
* Add Slave Node
``` sh
./redis-trib.rb add-node --slave --master-id 3c3a0c74aae0b56170ccb03a76b60cfe7dc1912e 10.222.0.6:7006 10.222.0.6:7000
```
* Fix Slot
``` sh
./redis-trib.rb fix 10.222.0.6:7000
```
* Reshard
``` sh
./redis-trib.rb reshard 127.0.0.1:7000
./redis-trib.rb reshard --from <node-id> --to <node-id> --slots <number of slots> --yes <host>:<port>
```
* Remove Node
remove slave
``` sh
./redis-trib del-node 127.0.0.1:7000 `<node-id>`
```
remove master
``` sh
./redis-trib.rb reshard --from <node-id> --to <node-id> --slots <number of slots> --yes 
./redis-trib del-node 127.0.0.1:7000 `<node-id>`
```

## Redis cli
``` sh
cd redis/src
./redis-cli -c -h 127.0.0.1 -p 7000
cluster nodes
```

## Trouble Shoot
* /var/run/redis_7000.pid exists, process is already running or crashed
``` sh
sudo rm -rf /var/run/redis_7000.pid
```
* clear node
``` sh
ps -ef | grep redis
kill -9
service stop redis_7000
sudo rm -rf /var/run/redis_7000.pid
sudo rm -rf /var/lib/redis/7000/*
```