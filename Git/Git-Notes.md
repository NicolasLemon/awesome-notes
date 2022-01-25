# Git 初玩

**在使用Git中碰到的常见玩法**

* **作者：** Nicolas·Lemon
* **修改：** Nicolas·Lemon
* **创建日期：** 2021.12.24
* **修改日期：** 2022.01.24



## 创建项目

### 啥也没有

* Create a new repository

  ```shell
  # 将空白远程仓库里的Git项目文件夹拉取到本地
  git clone http://${remote_address}/${project}.git
  # 进入这个项目的文件夹的目录
  cd ${project}
  # 选择master分支 &&
  # 创建一个README文件 &&
  # 添加所有文件变化至暂存空间 &&
  # 提交并添加提交记录 && 推送master分支至远程仓库
  git switch -c master && \
  touch README.md && \
  git add . && \
  git commit -m "Add README file" && \
  git push -u origin master
  ```

### 文件夹式

* Push an existing folder

  ```shell
  # 进入到文件夹中
  cd ${existing_folder}
  # 初始化分支为master
  git init --initial-branch=master
  # 将分支关联到远程仓库
  git remote add origin http://${remote_address}/${project}.git
  # 添加所有文件变化至暂存空间 &&
  # 提交并添加提交记录 &&
  # 更新推送至远程仓库
  git add . && \
  git commit -m "Initial commit" && \
  git push -u origin master
  ```

### 本地仓库

* Push an existing Git repository（未测试）

  ```shell
  # 进入到仓库中
  cd ${existing_repo}
  # 重命名分支
  git remote rename origin ${old_origin}
  # 将分支关联到远程仓库
  git remote add origin http://${remote_address}/${project}.git
  # 更新所有分支
  git push -u origin --all
  git push -u origin --tags
  ```



## 命令连接符

### `&&` 连接

命令连接符 `&&` ：前一条命令执行成功才执行后一条命令

### `;;`连接

命令连接符 `;;` ：不论前一条命令是否执行成功都继续执行后一条命令



## 常用玩法

### 全局设置

* Git全局设置

  ```shell
  # 设置Git用户名
  git config --global user.name "${yourname}"
  # 设置Git用户邮箱
  git config --global user.email "${youremail}"
  # 设置Git-HTTP代理
  git config --global http.proxy 'socks5://127.0.0.1:10808'
  # 取消HTTP代理，让Git-HTTP走系统代理
  git config --global --unset http.proxy
  # 设置Git-HTTPS代理
  git config --global https.proxy 'socks5://127.0.0.1:10808'
  # 取消HTTPS代理，让Git-HTTPS走系统代理
  git config --global --unset https.proxy
  
  # 配置全局.gitignore文件，并将macOS的配置文件给忽略掉
  echo "*.DS_Store" >> ~/.gitignore
  git config --global core.excludesfile ~/.gitignore
  ```

### 强制推送

* 强制推送

  ```shell
  # 强制推送master分支
  git push origin master --force
  ```

### 清除冗余

* 清除远程仓库中不需要的文件

  ```shell
  # 先pull远程代码，保持同步
  git pull
  # 删除单个文件
  git rm ${file} --cached
  # 删除文件夹
  git rm -r ${floder} --cached
  # 提交
  git commit -m "清除冗余"
  # 推送
  git push
  ```

### 覆盖本地

* 将远程仓库强制覆盖到本地

  ```shell
  # 拉取所有更新，不同步 && 本地代码同步线上最新版本（会覆盖本地所有与远程仓库上同名的文件） && 再更新一次
  git fetch --all &&  git reset --hard origin/master && git pull origin master
  ```

### 抹除记录

* 抹除Git的提交记录

  ```shell
  # 基于当前分支创建一个独立的分支new_branch &&
  # 添加所有文件变化至暂存空间 && 
  # 提交并添加提交记录 &&
  # 删除master分支 && 
  # 重新命名当前独立分支为master && 
  # 强制推送到远程仓库
  git checkout --orphan new_branch && \
  git add . && \
  git commit -m "Initial commit" && \
  git branch -D master && \
  git branch -m master && \
  git push -f origin master
  ```

### 修改仓库指向

* 在Git远程仓库中修改了项目名后，远程仓库地址也会发生变化，所以需要将本地的仓库也同步一下

  ```sh
  # 查看远程仓库名称
  git remote
  ```

  <img src="Git-Notes.assets/image-20220104122456104.png" alt="image-20220104122456104" style="margin-left:30px;" />

  ```sh
  # 更新指向的远程仓库
  git remote set-url ${origin} ${new_remote_address}
  ```
  

### 生成ssh密钥

macOS系统下默认的`.ssh`的目录是 **~/.ssh** ，基本上也不用去改它

1. 先设置Git的 **全局用户名** 和 **全局用户邮箱**（如果没有配置过的话）

   ```sh
   # 设置Git用户名
   git config --global user.name "${yourname}"
   # 设置Git用户邮箱
   git config --global user.email "${youremail}"
   ```

2. 查看当前电脑下是否存在`.ssh`目录，如果存在，则先备份，然后再删除

   ```sh
   # 进入目录，看是否存在一个已有的.ssh目录
   cd ~/.ssh
   ```

   <img src="Git-Notes.assets/image-20220112205030288.png" alt="image-20220112205030288" style="margin-left:30px;" />

3. 生成密钥

   ```sh
   ssh-keygen -t rsa -C "${youremail}"
   ```

4. 根据提示，一路三个回车即可，当然需要设置密码的，就设置

   <img src="Git-Notes.assets/image-20220112205804489.png" alt="image-20220112205804489" style="margin-left:30px;" />

5. 然后就可以在 **~/.ssh** 目录下见到两个文件

   `id_rsa.pub`是公钥，`id.rsa`是私钥（这个要自己 **保密** 哦）

   <img src="Git-Notes.assets/image-20220112205935113.png" alt="image-20220112205935113" style="margin-left:30px;" />
   
6. 测试连接

   ```sh
   # 测试连接（有提示输入yes即可）
   ssh -T git@github.com
   # ssh连接debugx
   ssh git@github.com -vT
   ssh git@github.com -vv
   ```

   
