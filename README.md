# 这个配置实际上是我部署`graylog-server`多节点时使用的方案，不过每个节点是分摊到多个节点服务器上的。此处记录合并到一起  

# 1. 初始化环境
```bash
$> mkdir -p /data/docker/mongodb/.keyfile 
$> openssl rand -base64 743 >  /data/docker/mongodb/.keyfile/mongodb-keyfile 
$> chmod 600 /data/docker/mongodb/.keyfile/mongodb-keyfile
$> chown -R 999.999 /data/docker/mongodb/.keyfile 
$> cd /data/docker/mongodb 
$> git clone https://github.com/0x5c0f/docker-mongodb.git . 
```

# 2. 启动 
```bash
$> cd /data/docker/mongodb 
$> docker-compose up -d
```

# 3. mongo initiate 初始分片
```bash
$> docker exec -it mongd01 mongo -uroot -p<passwd>
rs0:SECONDARY> rs.initiate({_id : 'rs0',members: [{ _id : 0, host : "mongo01:27017" },{ _id : 1, host : "mongo02:27017" },{ _id : 2,host : "mongo03:27017" }]})      # 仅在一个节点执行即可 
# rs.status() # 查看分片配置状态，可以在每个节点查看下，可以全部查看到了  
```

# 4. 账号管理
```bash
rs0:SECONDARY> use graylog
rs0:PRIMARY> db.createUser( { user: "graylog", pwd: "<gray_passwd>", roles: [ { role: "readWrite", db: "graylog" } ]});  
rs0:PRIMARY> db.grantRolesToUser( "graylog" , [ { role: "dbAdmin", db: "graylog" } ])  
rs0:PRIMARY> db.auth("graylog","<gray_passwd>")  
```

#  以上算是一个完整的吧，`mongodb`如果不是`graylog-server`要使用到，我都没有机会去了解这个，不知道这块的概念对不对 。 

# 最后在接入我部署`graylog-server`的一个完整过程吧  
> [https://blog.cxd115.me/115/95.html](https://blog.cxd115.me/115/95.html)