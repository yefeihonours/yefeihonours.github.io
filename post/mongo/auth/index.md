### 启动MongoDb实例

> 本文以`Docker`启动`MongoDB`为例，省略服务器上MongoDB安装过程

```
# Docker启动MongoDB实例，使用前确保电脑上已经安装docker

$ docker pull mongo:3.0
...

# --auth 参数表示以权限验证模式启动
$ docker run --name mongo_instance -p 0.0.0.0:27017:27017 -d mongo --auth
d18d1d4fd831...a5bc378178

```

<!--more-->

此时，mongo实例已经启动了，可以通过`docker ps`命令查看启动状态。

```
$ docker ps
CONTAINER ID    IMAGE       ...   PORTS                      NAMES
d18d1d4fd831    mongo:3.0   ...   0.0.0.0:27017->27017/tcp   mongo_instance
```

### 连接MongoDB，创建管理账号

```
$ docker exec -it mongo_instance mongo admin
MongoDB shell version: 3.0.15
connecting to: admin
Welcome to the MongoDB shell.
For interactive help, type "help".
For more comprehensive documentation, see
	http://docs.mongodb.org/
Questions? Try the support group
	http://groups.google.com/group/mongodb-user
>
"""
# 创建管理员账户，拥有对Mongo所有数据库的管理权限
# userAdminAnyDatabase 允许用户授予任何用户（包括他们自己）的任何权限

> db.createUser({
    user: 'root_user',
    pwd: 'password',
    roles: [{ role: "userAdminAnyDatabase", db: "admin" }]
});
Successfully added user: {
	"user" : "root_user",
	"roles" : [
		{
			"role" : "userAdminAnyDatabase",
			"db" : "admin"
		}
	]
}
"""
```

### 创建数据库，添加用户并配置权限

```
# 添加新的数据库[选择数据库]
> use mydb
# 创建新用户，赋予"读写"权限
> db.createUser({
    user: 'user',
    pwd: 'password',
    roles: [ { role: "readWrite", db: "mydb" } ]
});
Successfully added user: {
	"user" : "user",
	"roles" : [
		{
			"role" : "readWrite",
			"db" : "mydb"
		}
	]
# 验证用户权限
> db.auth('dunwei', 'password')
1
```

验证用户通过，此时就可以通过图形化工具连接MongoDB的实例了。

图形化工具推荐使用 `Studio 3T MongoDB Chef` ([点此下载](https://studio3t.com/download/))。

### 常用Mongo用户权限
```
# 数据库用户权限
read                提供对当前数据库所有"读取"数据的权限
readWrite           提供对当前数据库所有"读写"数据的权限

# 数据库管理权限
dbAdmin             提供对当前数据库"管理"的权限，与模式相关的任务，索引，收集统计信息
dbOwner             提供对当前数据库执行"任何管理操作"的能力[readWrite,dbAdmin,userAdmin]
userAdmin           提供在数据库上"创建和修改角色和用户"的能力

# 所有数据库权限
readAnyDatabase         提供对"全部"数据库"read"的权限
readWriteAnyDatabase    提供对"全部"数据库"readWrite"的权限
userAdminAnyDatabase    提供对"全部"数据库"userAdmin"的权限
```
