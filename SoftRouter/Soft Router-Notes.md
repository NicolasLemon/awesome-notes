# 软路由折腾笔记

* **作者：** Nicolas·Lemon
* **修改：** Nicolas·Lemon
* **创建时间：** 2021.11.07
* **修改时间：** 2022.04.11



## OpenWrt

### 笔记本电脑改造

* 笔记本电脑基本信息：

  **型号：** 联想s40-70

  **网口：** 单网口，100Mps全双工

  **处理器：** i5

  **硬盘：** 三星sata固态硬盘（三星870evo **500G**）

  **内存：** DDR3 8G

* 软路由系统选择

  这里的软路由固件选择的是**eSir**大神编译好的**lede**系统

  附上**eSir**大神**谷歌**网盘链接（需科学上网）：https://drive.google.com/drive/folders/1dqNUrMf9n7i3y1aSh68U5Yf44WQ3KCuh

#### 镜像选择

1. 在eSir大神的网盘里选择第一个目录

   <img src="Soft Router-Notes.assets/image-20211107223232856.png" alt="image-20211107223232856" style="margin-left:30px;" />

   在此目录下，可以根据自己的喜好去，选择是**高大全版**还是**精品小包**，作者是选的**高大全版**

2. 在**高大全版**里选择一个自己想要下载的镜像下载即可

   <img src="Soft Router-Notes.assets/image-20211107223916559.png" alt="image-20211107223916559" style="margin-left:30px;" />

3. 下载完成后选择带有**legacy**字样的**img.gz**压缩包

   **legacy**版的兼容性更好

   <img src="Soft Router-Notes.assets/image-20211107224142086.png" alt="image-20211107224142086" style="margin-left:30px;" />

#### 写盘工具

常见的写盘工具有：**img写盘工具**、**physdiskwrite**、**rufus**等，rufus是写入u盘的工具，此处需要通过u盘将镜像img写入到硬盘中，故此处不考虑

* **physdiskwrite**工具是用命令行来写入的
* **img写盘工具**是界面化的工具

<img src="Soft Router-Notes.assets/image-20211107225945820.png" alt="image-20211107225945820" style="margin-left:30px;" />

#### 组网方案

软路由接入组网的方案主要有三种：**主路由**模式、**旁路由**模式与**单臂路由**模式，因为作者笔记本电脑是单网口的，只能用**旁路由**或者**单臂路由**模式接入到网络中（作者采用的旁路由模式）

##### 主路由模式

由**软路由器**拨号，**硬路由器**发送WIFI（硬路由器需设置成**AP**模式，只利用其能发送WIFI的性能，网络数据的处理交给软路由来）

<img src="Soft Router-Notes.assets/image-20211108012940141.png" alt="image-20211108012940141" style="margin-left:30px;zoom:50%;" />

##### 旁路由模式

<img src="Soft Router-Notes.assets/image-20211108013625439.png" alt="image-20211108013625439" style="margin-left:30px;zoom:50%;" />

<img src="Soft Router-Notes.assets/image-20211108013116605.png" alt="image-20211108013116605" style="margin-left:30px;zoom:50%;" />

<img src="Soft Router-Notes.assets/image-20211108012627069.png" alt="image-20211108012627069" style="margin-left:30px;zoom:50%;" />

##### 单臂路由模式

<img src="Soft Router-Notes.assets/image-20211108014200776.png" alt="image-20211108014200776" style="margin-left:30px;zoom:50%;" />

####  镜像安装

1. 先备份好原电脑的**所有**硬盘上的数据

   此处与刷Windows系统不同，刷windows系统，只要刷系统盘就可以了，但是刷OpenWrt，写盘工具是会针对**整个物理磁盘**的，而不是**某个分区**，所以整个硬盘上的数据都会被抹除掉

2. 利用 **WePE** U盘启动进入需要刷入的笔记本电脑

   没制作U盘启动的先制作U盘启动，作者选择的是WePE

3. 用磁盘分区工具，直接**清除所有分区**即可

   如前面所讲，刷入OpenWrt是针对整个磁盘的，所以要分区也没有用，作者也尝试过，单独刷某个分区，比如说刷C盘，最后的结果是img镜像刷入不成功

   <img src="Soft Router-Notes.assets/image-20211107233421519.png" alt="image-20211107233421519" style="margin-left:30px;zoom:80%" />

4. 然后利用写盘工具将img文件写入到**物理磁盘**中

   **不要选分区，选物理磁盘**

   <img src="Soft Router-Notes.assets/image-20211107225945820.png" alt="image-20211107225945820" style="margin-left:30px;" />

   刷入成功后，此时分区就变成两个了

   <img src="Soft Router-Notes.assets/image-20211107233736861.png" alt="image-20211107233736861" style="margin-left:30px;zoom:80%" />

   引导分区大小是**16MB**，系统分区大小是**500MB**，其他未用的空间会显示在**未使用状态**下

   （此处系统分区显示的是465.75GB是因为作者**已经扩展**了系统空间了）

5. 扩展OpenWrt系统空间

   初次装好后，500GB的硬盘空间并不会全部利用上，只使用了**16MB的引导分区**和**500MB的系统分区**，此处再利用**DiskGenius**将**未使用的空间**给配给**系统分区**，是分给**500MB**的那个分区哦，空间分配好了，就只剩下了两个空间

   <img src="Soft Router-Notes.assets/1.jpg" alt="1" style="margin-left:30px;zoom:80%;" />

   **注意，此时的磁盘格式是MBR的，是写盘时强制刷成MBR的，也就是说后续装双系统时，磁盘只能选择MBR的**

6. 关机，拔掉u盘

#### 初始配置

1. 开机后，等待系统加载完成，界面上会显示**按下任意键进入控制台**，那就如他所愿，进入控制台呗

2. 修改网络配置文件

   ```shell
   vi etc/config/network
   ```

   找到**lan**网口配置，将**ipaddr**改为**自己路由器网段中未使用的ip**（此处就是**软路由后台的地址**）

   <img src="Soft Router-Notes.assets/image-20211107235750681.png" alt="image-20211107235750681" style="margin-left:30px;" />

   记得 **:wq!** 保存退出哦

3. 重启系统，使配置生效

   ```shell
   reboot
   ```

   此处，对于**旧笔记本电脑**的操作就完成了，可以**转入另一台电脑**了

4. 登录软路由后台

   输入刚刚配置好的ip

   <img src="Soft Router-Notes.assets/image-20211108000432566.png" alt="image-20211108000432566" style="margin-left:30px;" />

   初次登录后，会提示你创建个新密码，按照提示跳转过去操作即可（**需要设置密码，以便后面利用SSH连接**）

5. 在 **网络** -> **接口** 中找到**LAN**配置，点击**修改**

   <img src="Soft Router-Notes.assets/image-20211108000917459.png" alt="image-20211108000917459" style="margin-left:30px;" />

6. 配置**基本设置**

   <img src="Soft Router-Notes.assets/image-20211108001654055.png" alt="image-20211108001654055" style="margin-left:30px;" />

   **保存&应用**

7. 配置**高级设置**和**物理设置**

   <img src="Soft Router-Notes.assets/image-20211108002056908.png" alt="image-20211108002056908" style="margin-left:30px;" />

   **保存&应用**

8. 配置**防火墙**

   在 **网络** -> **防火墙** -> **自定义规则** 中加入一行命令，避免某些设备连上后会上不了网

   ```shell
   iptables -t nat -I POSTROUTING -o eth0 -j MASQUERADE
   ```

   <img src="Soft Router-Notes.assets/image-20211108002506290.png" alt="image-20211108002506290" style="margin-left:30px;" />

   此时，软路由的配置就配好了，此时**软路由**与**主路由**是通的了，并且是可以**通过软路由上网了**

#### 设备配置

将设备的**IP网关**和**DNS网关**全指向软路由的网关即可（电脑端、移动端、网络电视端等）

* **Windows**

  <img src="Soft Router-Notes.assets/image-20211108003326193.png" alt="image-20211108003326193" style="margin-left:30px;zoom:80%;" />

* **Android**

  <img src="Soft Router-Notes.assets/image-20211108003726440.png" alt="image-20211108003726440" style="margin-left:30px;zoom:80%;" />

这样设备的网络数据就可以通过软路由了，实现了软路由作**旁路由**模式

#### 进阶玩法

##### PassWall

**顾名思义，不解释了**

* 在**节点列表**中添加节点后，如下配置，即可科学上网了

  <img src="Soft Router-Notes.assets/image-20211108004504331.png" alt="image-20211108004504331" style="margin-left:30px;zoom:80%;" />

* **节点列表**

  <img src="Soft Router-Notes.assets/image-20211108004656317.png" alt="image-20211108004656317" style="margin-left:30px;zoom:80%;" />

* **节点订阅**

  <img src="Soft Router-Notes.assets/image-20211108005221860.png" alt="image-20211108005221860" style="margin-left:30px;zoom:80%;" />

* **自动切换**

  自动切换**并不会**去切换到最快的那个节点，只是当主节点失效时，会**从备用节点中**去找寻能连的上的节点，最大程度确保能科学上网

  <img src="Soft Router-Notes.assets/image-20211108010715465.png" alt="image-20211108010715465" style="margin-left:30px;" />
  
  配置了**自动切换**后的效果日志
  
  <img src="Soft Router-Notes.assets/image-20211108011650934.png" alt="image-20211108011650934" style="margin-left:30px;zoom:80%" />

##### SmartDNS

**自动选择最快DNS**的插件

##### GodProxy

**广告过滤**插件

##### 网易云灰色歌曲

替换掉网易云中的歌曲播放链接，达到能**在网易云中听任意歌曲**的效果

#### 双系统

##### OpenWrt + Win

```shell
ls
ls (hd0,msdos5)
set root=(hd0,msdos5)
```

