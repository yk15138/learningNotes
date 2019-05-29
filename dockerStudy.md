#docker 学习笔记


##镜像命令
### 查看所有镜像
> docker images {options} 

    -a 查看所有  
    -q 只有id
    
   ==
### 从dockerHub上找镜像
> docker search {options} ${image} 

    -s 点赞数(docker search -s 30 tomcat)
     
   ==

### 拉取镜像 --
> docker pull ${image}  

    可以换阿里云的地址 能快很多  
    [aliyun镜像加速地址](https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors)
    
### 删除镜像 --
> docker rmi {options} ${image} 
 
    -f 强制删除
    删除全部  docker rmi -f $(docker images -qa)
    
    
##容器命令 --
### 新建并启动容器 --
> docker run {options} ${image}

    -i 交互
    -t 为容器分配一个伪输入终端
    -d 后台运行
    -p 端口映射
    --name 设置名字
    -P 随机端口映射
    交互式启动一个tomcat容器  docker run -it -p 8080:8080 tomcat:latest
    后台启动一个Tomcat容器  docker run -itd -p 8080:8080 tomcat:latest
    
### 列出当前的容器
> docker ps {options}
    
    -a 所有的容器，包括为运行的
    -l 显示最近创建的容器
    -n 显示最近几个容器
    -q 静默模式，只显示容器编号

### 退出容器

> exit  退出容器并关闭  
`ctrl` + `P` + `Q`   退出但不关闭容器
    
### 启动容器
> docker start ${containerId}

### 重启容器
> docker restart ${containerId}

### 停止容器
> docker stop ${containerId}

### 强制停止容器
> docker kill ${containerId}

### 删除容器
> docker rm {options} ${containerId}
     
     -f 强制删除，没关闭的也删除
     docker rm -f $(docker ps -qa) 删除所有
     
### 查看容器日志
> docker logs {options} ${containerId}
    
    -t 显示时间戳
    -f 跟随最新的日志打印
    --tail n 显示最后n条

### 查看容器运行的程序
> docker top    
    
### 查看容器内部细节 --
> docker inspect ${containerId}

### 进入正在运行的容器
> docker attach ${containerId}

### 在容器中打开新的终端，启动新进程 --
> docker exec -it  ${containerId} (/bin/bash)

### 容器内的数据拷贝到机器上
> docker cp ${containerId}:${filePath} ${filePath}

## 镜像commit
> docker commit {options} ${containerId} {namespace}/{name}:{version}

    -m 描述信息
    -a 作者
    
   
## 容器数据卷
### 命令添加
> docker run {options} -v {宿主机目录}:{容器目录}
    
    -v /etc/timezone:/etc/timezone -v /etc/localtime:/etc/localtime
    同步宿主机时间
    
### dockerFile添加
> VOLUME \["/data1","/data2"\]

    默认实体机位置
    /var/lib/docker/volumes/${random}/_data
    
### 数据卷容器
> docker run --volume-from  数据卷继承
    


## dockerFile 学习
- 1.手动编写一个dockerfile文件（必须要符合file的规范）  
- 2.直接docker build命令  
- 3.run

### 保留字指令
> FROM 从... 继承
   
    FROM scratch 从基类继承
    
> MAINTAINER  镜像维护者的姓名和邮箱地址

    MAINTAINER yuakang<15589523189@163.com>

> RUN 镜像构建时需要运行的命令
    
    RUN yum install -y vim net-toole

> EXPOSE 当前容器对外暴露的端口

    EXPOSE 80

> WORKDIR 默认登录进来的目录

    WORKDIR /usr/local

> ENV 设置环境变量

    ENV JAVA_HOME /home/java

> ADD 将宿主机文件拷贝到容器中，zip会自动解压

    ADD dockerfile /usr/local/java
   
> COPY 类似add，不会解压
    
    ADD dockerfile /usr/local/java

> VOLUME 容器数据卷
    
    VOLUME ["/data1", "data2"]

> CMD 指定容器启动是进行的指令，多个时只有最有一个生效，会被docker后面的指令覆盖掉
    
    CMD ["curl", "-s", "http://ip.taobao.com/service/getIpInfo2.php?ip=myip"]

> ENTRYPOINT 类似CMD
    
    ENTRYPOINT ["curl", "-s", "http://ip.taobao.com/service/getIpInfo2.php?ip=myip"]
    
> ONBUILD 当构建一个被继承的dockerfile时运行命令，会被触发

    ONBUILD RUN echo "hello world"
    
## docker 常用安装
### tomcat 安装
    docker search tomcat 查询镜像  
    docker pull tomcat 拉取镜像  
    docker images 查看所有的镜像  
    docker run -itd -p 8080:8080 --name tomcat1 tomcat 创建并启动tomcat  
    docker stop ${containerId} 停止容器
    docker restart ${containerId} 重启容器
    docker start ${containerId} 启动容器
    docker logs -t -f --tail 10 ${containerId} 从后面10条跟踪日志
    docker exec -it ${containerId} /bin/bash 操作容器

### mysql安装
> docker run --name mysql1 -p 3306:3306 -v /opt/mysql/conf:/etc/mysql/conf.d -v /opt/mysql/logs:/logs -v /opt/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=12345 -d mysql:5.6

    docker run --name mysql1 -p 3306:3306 
    -v /opt/mysql/conf:/etc/mysql/conf.d
    -v /opt/mysql/logs:/logs
    -v /opt/mysql/data:/var/lib/mysql
    -e MYSQL_ROOT_PASSWORLD=12345
    -d mysql:5.6

### redis安装
> docker run --name redis1 -p 6379:6379 -v /opt/redis/data:/data -d redis redis-server  --appendonly yes

    docker run --name redis1 -p 6379:6379
    -v /opt/redis/data:/data
    -v /opt/redis/conf/redis.conf:/usr/local/etc/redis/redis.conf
    -d redis redis-server /usr/local/etc/redis/redis.conf
    --appendonly yes
    
    
### rabbitmq 安装
> docker run -d --hostname rabbitmq -p15672:15672 -p5672:5672 --name rabbitmq -e RABBITMQ_DEFAULT_VHOST=/ -e RABBITMQ_DEFAULT_USER=user -e RABBITMQ_DEFAULT_PASS=password rabbitmq:3-management



## docker 文档
[docker 文档](docs.docker.com)
## docker hub
[docker hub](hub.docker.com)

###### 勿在浮沙上面筑高台，出老混迟早是要还的。
    

   
     
    
