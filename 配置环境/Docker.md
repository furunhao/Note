# 安装 Oracle

```docker
docker search image_name -- 搜索镜像
docker pull image_name -- 拉取镜像
docker run -d -p 1521:1521 --name 别名 image_name -- 转换镜像为容器并使用
sudo docker exec -it image_id（通过docker ps -a获得）/bin/bash -- 启动容器
su - oracle -- 切换用户为Oracle
sqlplus system/oracle 进入Oracle
create user user_name identified by password -- 创建用户密码
grant connect,resource,dba to user_name -- 授权用户
alter user new_user_name identified by new_password -- 修改密码
```

- [错误：Sqlplus command not found]( https://www.cnblogs.com/qingmuchuanqi48/articles/15641505.html)

