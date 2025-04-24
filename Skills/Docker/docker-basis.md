# Docker 基础

目录：

- [Docker 基础](#docker-基础)
  - [参考链接](#参考链接)


docker

docker ps [-a] # 查看运行中的容器、查看所有容器（包括暂停的） [-a]
docker pull [image]
docker run  [image] 
    [-d] 
    [--net] = docker pull + docker start 
    [-p*（本机）:*（容器）] --name ** # 创建新的（包括额外的参数）
    [-v] volume 挂载 持久化
docker stop [ID of contartiner]
docker start [ID of contartiner] # 启动新的
docker images
docker logs [container id/name]

docker exec -it [continer id / name] /bin/bash
exit

docker network create network-name

docker network ls

docker-compose -f **.yaml up

DockerFile 

docker build -t my-app:1.0(tag) .(目录)
docker rmi [id 或者 name]（删除 image）
docker rm [id 或者 name] （删除 container）

注意：当同一个 id 绑定多个 tag 时，需要指定 tag 进行删除
 



k8s



docker run --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=root -d  mysql:5.7 --lower_case_table_names=1

docker run -id --name=mysql_5.7 -v mysql-config:/etc/mysql/conf.d -v mysql-log:/logs -v mysql-data:/var/lib/mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -e LANG=C.UTF-8 mysql:5.7

docker run -id --name=mysql -v mysql-config:/etc/mysql/conf.d -v mysql-log:/logs -v mysql-data:/var/lib/mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -e LANG=C.UTF-8 mysql

## 参考链接
