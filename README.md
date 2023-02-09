### 介绍
通过docker快速内网部署YApi Mock服务平台

GitHub仓库: https://github.com/OnlyBrownAnt/docker-for-yapi

### 准备资料
1. 已安装过docker
2. yapi官网: https://hellosean1025.github.io/yapi/index.html
3. mongosh: https://www.mongodb.com/docs/mongodb-shell/
4. mock.js: https://github.com/nuysoft/Mock
5. mongoose: http://mongoosejs.net/docs/connections.html
6. docker-mongo: https://hub.docker.com/_/mongo/#!
### 默认设置

```shell
mongoDB
默认数据库: yapi
默认初始密码: root
默认初始密码: password

YApi:
管理员账号: root@email.com
管理员密码: ymfe.org

YApi访问地址: http://127.0.0.1:3000
mongoDB数据库管理(express): http://127.0.0.1:8081
```

### 部署步骤
> 可以通过以下命令查看容器列表: docker ps -a
> 
> 假设mongoDB docker容器ID(CONTAINER ID)是ca49b6c498cb
> 
> 假设node docker容器ID(CONTAINER ID)是5c1d09ec15c3

#### 下载YApi Git仓库
```shell
git clone https://github.com/YMFE/yapi.git vendors 
// 或者下载 zip 包解压到 vendors 目录（clone 整个仓库大概 140+ M，可以通过 `git clone --depth=1 https://github.com/YMFE/yapi.git vendors` 命令减少，大概 10+ M）
```

#### 创建docker网络
```shell
docker network create yapi_default
```
#### 执行docker.yml
> 部署mongoDB、mongo-express、node
> 
> docker-compose -f docker-compose-1.yml up -d
> 
> 运行http://127.0.0.1:8081能查看到数据库信息，即安装成功

#### 进入mongoDB配置访问用户
1. 查看当前docker列表，获取当前mongoDB docker的容器ID(CONTAINER ID)
```shell
docker ps -a
```
2. 进入容器
```shell
docker exec -it ca49b6c498cb /bin/bash
```
3. 配置访问用户
```shell
mongosh "mongodb://root:password@127.0.0.1:27017"

use yapi 

db.createUser({user:"root",pwd:"password",roles:[{"role":"readWrite","db":"yapi"}]})
```
#### YApi配置文件
1. 查看当前docker局部网络的ip情况，获取mongoDB docker的IP
```shell
docker network inspect yapi_default
```
> 假设得到mongoDB docker所分配的是192.168.80.2/20
2. YApi配置文件config.json，设置mongoDB地址为步骤1中得到的192.168.80.2
3. 设置config.json中db.authSource="admin"
> 因为yapi连接mongoDB的默认管理员密码是mongoDB的admin库中的用户。
#### 部署启动YApi
1. 查看当前docker列表，获取当前node docker的容器ID(CONTAINER ID)
```shell
docker ps -a
```
3. 进入容器
```shell
docker exec -it 5c1d09ec15c3 /bin/bash
```
4. 执行以下命令
```shell
// 配置YApi
cd vendors && npm install --production --registry https://registry.npm.taobao.org && npm run install-server

// 并会提示'初始化管理员账号成功,账号名："root@email.com"，密码："ymfe.org"'

// 启动服务
node server/app.js
```
> 当提示"log: 服务已启动，请打开下面链接访问:"时，就意味着安装完成。
### 常见问题处理
1. 在第一次安装完成之后，node docker突然关闭该如何启动
```shell
// 启动docker
docker restart ca49b6c498cb

// 进入docker
docker exec -it 5c1d09ec15c3 /bin/bash

// 重启服务
node server/app.js
```
2. mongoDB docker突然关闭了该如何启动
```shell
// 启动docker
docker restart ca49b6c498cb
```
3. 在执行过node server/app.js的情况下再执行命令时会报错"已有init.lock文件"
> 再删掉该文件即可