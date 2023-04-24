**Docker - Notes**

* **作者：** Nicolas·Lemon
* **修改：** Nicolas·Lemon
* **创建时间：** 2022.01.01
* **修改时间：** 2023.04.24

# 常用命令

## 镜容相关

```bash
# 查看docker帮助文档
docker --help
docker ${command} --help

# 查看镜像
docker images -a
# 删除镜像
docker rmi ${image}

# 查看正在运行的容器
docker ps
# 查看所有的容器（包括已经停止运行的）
docker ps -a
# 停止容器
docker stop ${container}
# 停止所有正在运行的容器
docker stop $(docker ps -q)
# 删除容器
docker rm ${container}
# 进入容器控制台
docker exec -it ${container} ${/bin/bash}

# 将容器中的文件（夹）复制到本地
docker cp ${container_id}:${container_target_path} ${local_target_path}
# 将本地文件（夹）复制到容器中
docker cp ${local_target_path} ${container_id}:${container_target_path}

# 导出镜像（到本地）
docker save -o ${image_name}.tar ${image_name}
# （从本地）导入镜像
docker load -i ${image_name}.tar

# 实时查看最新10条容器运行日志
docker logs -f --tail=10 ${container_name}
```

### 网络相关

```bash
# 查看网络帮助文档
docker network --help
# 查看已有网络列表
docker network ls
# 创建网络
docker network create ${network_name}
# 查看网络详情
docker network inspect ${network_name}
# 删除网络
docker network rm ${network_name}
```

## 移除镜容步骤

**步骤【不可】随意调换**

```bash
# 1. 先查看容器是否在运行，如果在运行中，则先停止容器，否则删除容器会失败
docker ps
docker stop ${container}
# 2. 移除容器
docker rm ${container}
# 3. 移除镜像
docker rmi ${container}
```

# 创建公共网络

因为docker容器是相互隔离的，如果A容器想连通B容器的话，最简单的办法，就是共用一个公共网络，比如说Redis、MySQL、Redisinsight等有的时候是需要互通的，所以在此给规定一个公共的网络`localnet`

```bash
docker network create localnet
```

![](README.assets/2023-04-24-22-15-58-image.png)

# MySQL

## MySQL 5.7

```bash
docker run \
--name mysql \
--network localnet \
-p 3306:3306 \
-v /opt/docker-volume/mysql/conf:/etc/mysql \
-v /opt/docker-volume/mysql/data:/var/lib/mysql \
-v /opt/docker-volume/mysql/log:/var/log/mysql \
-e MYSQL_ROOT_PASSWORD=${password} \
-d mysql:5.7
```

## MySQL 8.0

1. 运行容器
   
   * **Linux**
     
     ```bash
     docker run \
     --name mysql8 \
     --network localnet \
     -p 3306:3306 \
     -v /opt/docker-volume/mysql/conf:/etc/mysql/conf.d \
     -v /opt/docker-volume/mysql/data:/var/lib/mysql \
     -v /opt/docker-volume/mysql/log:/var/log/mysql \
     -e MYSQL_ROOT_PASSWORD=${password} \
     -e TZ=Asia/Shanghai \
     -e default-time_zone='+8:00' \
     -d mysql:8.0
     ```
   
   * **Windows**
     
     ```bash
     docker run \
     --name mysql8 \
     --network localnet \
     -p 3306:3306 \
     -v /d/Daturm/DockerVolume/mysql/conf:/etc/mysql/conf.d \
     -v /d/Daturm/DockerVolume/mysql/data:/var/lib/mysql \
     -v /d/Daturm/DockerVolume/mysql/log:/var/log/mysql \
     -e MYSQL_ROOT_PASSWORD=root \
     -e TZ=Asia/Shanghai \
     -e default-time_zone='+8:00' \
     -d mysql:8.0
     ```

2. 配置权限
   
   ```mysql
   # 配置root连接权限
   GRANT ALL PRIVILEGES ON *.* TO 'root'@'%'WITH GRANT OPTION;
   
   #创建一个开发者账户，且指定只能访问的数据库
   create user 'developer'@'%' identified by '${devPassword}';
   # 查看用户权限
   show grants for 'developer'@'%';
   # 赋予数据库权限
   GRANT ALL PRIVILEGES ON ${database}.* TO 'developer'@'%' WITH GRANT OPTION;
   # 移除权限
   REVOKE ALL PRIVILEGES,GRANT OPTION FROM 'developer'@'%';
   # 刷新配置
   FLUSH PRIVILEGES;
   ```

## 字符集设置

### MySQL 5.7 & 8.0

1. 新增/修改配置文件
   
   ```bash
   # 编辑/新增my.cnf配置文件
   cd /opt/docker-volume/mysql/conf/
   vim my.cnf
   ```
   
   * **my.cnf**
     
     ```bash
     [client]
     default-character-set=utf8
     [mysql]
     default-character-set=utf8
     [mysqld]
     init_connect='SET collation_connection = utf8_unicode_ci'
     init_connect='SET NAMES utf8'
     character-set-server=utf8
     collation-server=utf8_unicode_ci
     skip-character-set-client-handshake
     skip-name-resolve
     bind-address=0.0.0.0
     ```

2. 重启MySQL
   
   ```bash
   docker restart mysql
   ```

3. 进入MySQL容器查看字符集
   
   ```bash
   # 进入Docker中的MySQL容器
   docker exec -it mysql /bin/bash
   # 登录MySQL
   mysql -uroot -p
   ```
   
   ```mysql
   # 查看字符集
   show variables like 'character_set_%';
   ```
   
   <img src="README.assets/image-20220102003954315.png" alt="image-20220102003954315" style="margin-left:30px;" />

4. 若在`Windows`下，发现上述配置无法生效，请进入容器内部配置`644`权限
   
   ```bash
   docker exec -it mysql8 /bin/bash
   cd etc/mysql/conf.d/
   chmod -R 644 my.cnf
   ```
   
   然后再重启MySQL容器即可
   
   ![](README.assets/2022-07-17-17-14-56-image.png)
   
   ![](README.assets/2022-07-17-17-20-56-image.png)

# Redis

```bash
docker run \
--name redis \
--network localnet \
-p 6379:6379 \
-v /opt/docker-volume/redis/data:/data \
-d redis redis-server \
--save 60 1 --loglevel warning
```

# Redisinsight

Redis可视化工具（Web）版

以Win10举例，注意笔者运行的Redis也是上述Docker容器，因此会存在容器间网络通信的问题，所以笔者就在此直接以redis的网络运行该容器（`--network reddis`）

```bash
docker run \
-itd \
--network localnet \
-v D:/Daturm/DockerVolume/redisinsight:/db \
-p 8001:8001 \
--name redisinsight \
redislabs/redisinsight:latest
```

## 连接Redis

上述端口设置的是8001，因此打开浏览器：http://localhost:8001/

此处需要注意的就是容器间网络的问题，查看redis网络，然后主机选择redis容器的ip就行（下面截图是没有共用公共网络的，改成下面的公共网络`localnet`即可）

```bash
docker network inspect localnet
```

![](README.assets/2023-04-12-17-17-41-image.png)

![](README.assets/2023-04-12-17-18-34-image.png)

# Jenkins

Jenkins目前好像不用有什么是需要上面几个容器互通网络的，因此自定义其他网络也是可以的

## 创建网络

```bash
# 创建网络
docker network create jenkins
```

## 运行容器

```bash
docker run \
--name jenkins \
--detach \
--privileged \
--network jenkins \
--network-alias docker \
--env DOCKER_HOST=tcp://docker:2376 \
--env DOCKER_TLS_CERTDIR=/certs \
--volume /opt/docker-volume/jenkins/jenkins-docker-certs:/certs/client \
--volume /opt/docker-volume/jenkins/jenkins-data:/var/jenkins_home \
--publish ${port}:8080 \
--publish 50000:50000 \
jenkins/jenkins 
```

# GitLab

## 创建网络

```bash
# 创建网络
docker network create gitlab
```

## 运行容器

```bash
docker run --detach \
--publish ${web_port}:${web_port} --publish ${ssh_port}:22 \
--name gitlab \
--network gitlab \
--volume /opt/docker-volume/gitlab/config:/etc/gitlab \
--volume /opt/docker-volume/gitlab/logs:/var/log/gitlab \
--volume /opt/docker-volume/gitlab/data:/var/opt/gitlab \
--shm-size 256m \
gitlab/gitlab-ce
```

## 配置端口

在上一步启动中，将GitLab的Web端口指向了自定义的`${web_port}`端口，将SSH连接的端口指向了自定义的`${ssh_port}` ，故需要进入GitLab容器中，更新它自身相应的端口。

1. 进入GitLab容器
   
   ```bash
   docker exec -it gitlab /bin/bash
   ```

2. 修改Web对应的端口配置
   
   ```bash
   vi /etc/gitlab/gitlab.rb
   
   # 在配置文件中加入下面一行话
   external_url "http://${ip}:${web_port}"
   ```
   
   <img src="README.assets/image-20220103192413724.png" alt="image-20220103192413724" style="margin-left:30px;" />
   
   保存并退出

3. 修改GitLab - SSH对应的端口配置
   
   ```bash
   gitlab_rails[gitlab_shell_ssh_port]=${ssh_port}
   ```

4. 使配置重新生效，并重启GitLab服务
   
   ```bash
   # 是配置重新生效
   gitlab-ctl reconfigure
   # 重启GitLab服务
   gitlab-ctl restart
   ```

## 初始密码

配置好GitLab后，会有个初始账号和密码

* **初始账号**
  
  root

* **初始密码**
  
  ```bash
  # 先进入容器
  docker exec -it gitlab /bin/bash
  # 查看初始密码
  cat /etc/gitlab/initial_root_password 
  ```
  
  <img src="README.assets/image-20220103193410364.png" alt="image-20220103193410364" style="margin-left:30px;" />

## Win10实例

这里的network是用上面已经创建好的

实例是在Win10上的本地Docker Desktop中本地部署的，并且新版的gitlab中的私有仓库是不允许`http`协议的，只能利用`ssh`协议，所以踩了一些坑，记录一下：

这里的`9732`是网站指向打开的端口，`9922`是指向ssh的端口

1. 运行容器
   
   ```bash
   docker run \
   -itd \
   -p 9732:80 -p 9922:22 \
   --network gitlab \
   -v  D:/Daturm/DockerVolume/gitlab/etc:/etc/gitlab  \
   -v  D:/Daturm/DockerVolume/gitlab/log:/var/log/gitlab \
   -v  D:/Daturm/DockerVolume/gitlab/opt:/var/opt/gitlab \
   --privileged=true \
   --name gitlab \
   gitlab/gitlab-ce
   ```

2. 配置容器
   
   成功运行容器后进入容器内部，配置网站访问ip和ssh的ip和端口
   
   ```bash
   #进容器内部
   docker exec -it gitlab /bin/bash
   
   #修改gitlab.rb
   vi /etc/gitlab/gitlab.rb
   
   #gitlab访问地址，可以写域名。如果端口不写的话默认为80端口
   external_url 'http://127.0.0.1'
   #ssh主机ip
   gitlab_rails['gitlab_ssh_host'] = '127.0.0.1'
   #ssh连接端口
   gitlab_rails['gitlab_shell_ssh_port'] = 9922
   ```
   
   此处只让配置重新生效，不要重启容器
   
   ```bash
   # 让配置生效
   gitlab-ctl reconfigure
   ```

3. 修改gitlab网站的指向端口
   
   ```bash
   vi /opt/gitlab/embedded/service/gitlab-rails/config/gitlab.yml
   ```
   
   ![](README.assets/2023-03-28-16-51-35-image.png)

4. 修改root密码
   
   此处也可以用上面的`初始密码`那步走，这里记录一下，是可以改root账户密码的
   
   ```bash
   # 进入控制台
   gitlab-rails console -e production
   
   # 查询id为1的用户，id为1的用户是超级管理员
   user = User.where(id:1).first
   # 修改密码为lhx123456
   user.password='lhx123456'
   # 保存
   user.save!
   # 退出控制台
   exit
   ```

5. 重启gitlab
   
   ```bash
   #重启gitlab 
   gitlab-ctl restart
   ```

至此先锋配置就完成了

下面讲下本地git配置ssh密钥的坑，之前配置了github的，所以此处是共用github的密钥，先把公钥（id_rsa.pub）中的内容复制添加到gitlab中，然后配置`~/.ssh`下的`config`，下面只看gitlab部分的就行，千万注意，User一定是`git`，之前我是填的gitlab中的用户名，导致就很坑，连不上gitlab

```bash
# ProxyCommand connect -S 127.0.0.1:10808 -a none %h %p

Host github.com
  User git
  Port 443
  Hostname ssh.github.com
  # 注意修改路径为你的路径
  IdentityFile "C:\Users\NicolasLemon\.ssh\id_rsa"
  TCPKeepAlive yes

Host gitlab
  User git
  Port 9922
  HostName 127.0.0.1
  PreferredAuthentications publickey
  # 注意修改路径为你的路径
  IdentityFile "C:\Users\NicolasLemon\.ssh\id_rsa"
  TCPKeepAlive yes
```

# Nginx

如果需要挂载配置文件到本地的话，直接进行挂载，可能会报找不到相应配置文件的错误，因此，在这，就先准备好配置文件后，再进行挂载启动和运行容器。

## 配置文件准备

思路：**直接将Nginx容器中的配置文件复制出来**

1. 先启动一个不挂载的Nginx容器
   
   ```bash
   docker run \
   --name nginx \
   --network localnet \
   -p 8080:80 \
   -d nginx:latest
   ```

2. 进入启动好的Nginx容器，查找配置文件的目录
   
   ```bash
   # 进入Nginx容器
   docker exec -it nginx /bin/bash
   
   ### 不出意外的话，是需要拷贝以下这几个地方的配置文件(夹)
   # 首页资源文件夹
   /usr/share/nginx/html
   # 配置文件
   /etc/nginx/nginx.conf
   # 配置文件夹
   /etc/nginx/conf.d
   # 日志文件夹
   /var/log/nginx
   ```
   
   ![](README.assets/2023-04-24-22-36-23-image.png)

3. 将容器中的配置文件复制到本地
   
   外部挂载的目录不见得非得同，按照自己的喜好规定即可
   
   ```bash
   docker cp nginx:/usr/share/nginx/html /opt/docker-volume/nginx/html
   docker cp nginx:/etc/nginx/nginx.conf /opt/docker-volume/nginx/conf/nginx.conf
   docker cp nginx:/etc/nginx/conf.d /opt/docker-volume/nginx/conf/conf.d
   docker cp nginx:/var/log/nginx /opt/docker-volume/nginx/logs
   ```

不出意外的话，复制出来的配置文件在本地的 **/opt/docker-volume/nginx/** 目录下了

![](README.assets/2023-04-24-22-47-12-image.png)

## 正式运行容器

在上述操作中，已经启动了一个未进行挂载的Nginx容器，现在需要删除它

```bash
docker stop nginx
docker rm nginx
```

接下来就可以已挂载启动的方式运行一个Nginx容器了

```bash
docker run -d -p 8080:80  \
--name nginx \
--network localnet \
-v /opt/docker-volume/nginx/html:/usr/share/nginx/html \
-v /opt/docker-volume/nginx/conf/nginx.conf:/etc/nginx/nginx.conf \
-v /opt/docker-volume/nginx/conf/conf.d:/etc/nginx/conf.d \
-v /opt/docker-volume/nginx/logs:/var/log/nginx \
nginx:latest
```

## 配置文件基础转发

```bash
location /${proxy_alias}/ {
    # root   /usr/share/nginx/html;
    # index  index.html index.htm;
    # 代理地址
    proxy_pass http://${ip}:${port}/;
    # 允许cross跨域访问
    add_header 'Access-Control-Allow-Origin' '*';
}
```

## 重新加载配置文件

**重新加载配置文件** 和 **重启Docker Nginx容器** 二选一

```bash
# 重新加载配置文件
docker exec -it nginx service nginx reload

# 重启Docker Nginx
docker restart nginx
```
