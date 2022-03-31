# Linux第三次实验

- ## 如何添加一个用户并使其具备sudo执行程序的权限？

  ```shell
  sudo adduser username ## 添加用户
  cat /etc/passwd ##查看passwd目录下是否多了一个用户
  id username ##查看用户的基本信息
  id ##查看当前用户自己的信息
  id root ##查看管理员
  sudo usermod -G sudo username ##添加sudo权限
  ```

  [![添加用户](https://asciinema.org/a/ldPdDnnm6rGHyIPIBe3WankED.svg)

  [](https://asciinema.org/a/ldPdDnnm6rGHyIPIBe3WankED)

  **注：本题中添加权限时误将用户名写成root，故新添加用户test授权失败。下一题中将给出作答。**

- ## 如何将一个用户添加到一个用户组？

  ```shell
  usermod -a -G group username ##添加用户到某个用户组
  id username ##查看用户在哪个用户组中，验证添加结果
  ```

  [![添加用户到用户组](https://asciinema.org/a/SFLUc6AuGd7O8qThBq37BHnud.svg)](https://asciinema.org/a/SFLUc6AuGd7O8qThBq37BHnud)

- ## 如何查看当前系统的分区表和文件系统详细信息？

  ```shell
  lsblk ##列举查看当前系统中的快设备
  sudo fdisk /dev/硬盘名 ##对其中一个硬盘进行分区
  进入磁盘管理交互式命令行：
  m ##获取帮助
  p ##打印查看分区表
  ```

  [![查看文件系统及其分区表](https://asciinema.org/a/lsr9afTYd116rT9U20RIHH77r.svg)](https://asciinema.org/a/lsr9afTYd116rT9U20RIHH77r)

- ## 如何实现开机自动挂载Virtualbox的共享目录分区？

  ```shell
  mkdir /mnt/share ##创建挂载点
  mount -t vboxsf shareubuntu（windows中的文件夹名）/mnt/share（ubuntu中的挂载点名）##进行挂载
  df -h ##查看是否挂载成功
  cd /mnt/share ##进入此目录下查找share中是否有windows系统共享文件夹下的text.txt文件
  cd /etc/systemd/system ##进入systemd文件夹
  sudo touch mnt-share.mount ##建立mount文件
  vi mnt-share.mount ##修改文件
  systemctl enable mnt-share.mount ##执行命令，使得文件生效
  ```

  **本地windows系统中的操作：**

  1. 在虚拟机设备中安装增强功能。

  2. 在C盘中建立文件夹Ubuntushare并且填入文本。

  3. 在虚拟机的共享文件夹中将Ubuntushare添加。

  4. 选定固定分配。

  5. 最后去终端进行操作。

     [![将共享文件进行挂载](https://asciinema.org/a/rDI60MWcBnZgGKOvkYKYidxJc.svg)](https://asciinema.org/a/rDI60MWcBnZgGKOvkYKYidxJc)

     [![检查是否挂载成功，尝试自动挂载](https://asciinema.org/a/3X7g821LWpqDyggV2NG0zK1rL.svg)](https://asciinema.org/a/3X7g821LWpqDyggV2NG0zK1rL)

- ## 基于LVM（逻辑分卷管理）的分区如何实现动态扩容和缩减容量？

  **实验前准备**：在虚拟机上创建新磁盘，创建两块，在新磁盘上进行实验。

  ```shell
  sudo fdisk /dev/sdb ##对sdb进行磁盘分区
  n ##创建分区
  +5G ##确定分区大小
  p ##查看分区
  w ##写入分区，使其生效
  sudo gdisk /dev/sdc ##对sdc进行磁盘分区 
  ? ##获取帮助
  lsblk ##最后查看分区是否创建成功
  sudo su - ##切换到root用户下
  id ##返回0说明你在root用户下，返回1000说明不在root下
  pvcreate /dev/sdb{1,2} ##创建物理卷PVsdb
  pvs&pvscan ##查看pv详细信息
  vgcreate demo-vg /dev/sdb{1,2} ##创建卷组VG，命名为demo-vg
  vgs&vgscan ##查看vg详细信息
  pvcreate /dev/sdc{1,2} ##继续创建物理卷PVsdc
  vgextend demo-vg /dev/sdc1 ##把物理卷sdc1添加到卷组demo-vg中，实现卷组的扩容
  vgdisplay ##查看卷组信息
  lvcreate -L 10G(-l 2000) -n demo-lv-1 demo-vg ##创建逻辑卷LV，-l参数为定PE数来设定逻辑分区大小，也可以使用-L参数直接指定逻辑分区大小，-n参数指定逻辑分区名称
  lvdisplay ##查看逻辑卷信息
  lvcreate -l 100%FREE -n demo-lv-2 demo-vg ##使用vgdisplay查看卷组剩余空间，把剩余空间全部用于创建逻辑卷demo-lv-2 
  vgs ##发现卷组free剩余空间为0，已全部创建逻辑卷
  mkfs.ext4 /dev/demo-vg/demo-lv-1(2) ##在指定分区上创建文件系统
  mkdir /mnt/demo-lv-1 ##创建挂载点
  mount /dev/demo-vg/demo-lv-1 /mnt/demo-lv-1 ##将分区挂载到指定目录 
  df -h ##查看分区是否成功挂载到指定目录
  cd /mnt/demo-lv-1 & touch test ##进入目录创建文件
  lvresize --size +(-)10G --resizefs demo-vg/demo-lv-1 ##分区扩容和缩减容量 注意：当卷组中的剩余空间不足时，当然不能为逻辑卷扩容，上面有一步中已经耗尽了卷组的剩余空间，故最后再使用此句进行扩容不能成功。
  ```

  [![增加新硬盘sdb与sdc并对其进行分区](https://asciinema.org/a/tcTv7LjjuKqOeTkAAgVG3m9LP.svg)](https://asciinema.org/a/tcTv7LjjuKqOeTkAAgVG3m9LP)

  [![LVM逻辑分卷管理创建各层并进行扩容与缩减](https://asciinema.org/a/AFjwas9ayzFc2oC98164coxwi.svg)](https://asciinema.org/a/AFjwas9ayzFc2oC98164coxwi)

- ## 如何通过systemd设置实现在网络连通时运行一个指定脚本，在网络断开时运行另一个脚本？

  一 . 创建两个不同的脚本，并且使用`cat`来查看脚本

  二 . 为脚本提供可执行权限

  `chmod u+x 脚本`

  三 . 创建一个新的服务单元

  ```shell
  [root@centos-8 ~]# cat /etc/systemd/system/run-at-startup.service ##运行脚本的服务
  [Unit]
  Description=Run script at startup after network becomes reachable
  After=network.target ##在网络服务启动后运行主程序ExecStart
  
  [Service]
  Type=oneshot
  RemainAfterExit=yes ##进程退出之后，服务依然执行
  ExecStart=/tmp/startup_script1.sh
  ExecStop=/tmp/startup_script2.sh ##停止主进程服务后去运行第二个脚本
  
  [Install]
  WantedBy=multi-user.target
  ```

  四 . 修改配置文件以后，需要重新加载配置文件，然后重新启动相关服务

  ```shell
   sudo systemctl daemon-reload ##重新加载配置文件
   sudo systemctl restart foobar ##重启相关服务
  ```

- ## 如何通过systemd设置实现一个脚本在任何情况下被杀死之后会立即重新启动？实现杀不死？

  若要实现总是重启，要修改运行该脚本的服务单元的配置文件，去修改service区块的`Restart`字段，以`sshd.service`**文件为例**：

  ```shell
  systemctl cat sshd.service
  
  [Unit]
  Description=OpenSSH server daemon
  Documentation=man:sshd(8) man:sshd_config(5)
  After=network.target sshd-keygen.service
  Wants=sshd-keygen.service
  
  [Service]
  EnvironmentFile=/etc/sysconfig/sshd
  ExecStart=/usr/sbin/sshd -D $OPTIONS
  ExecReload=/bin/kill -HUP $MAINPID
  Type=simple
  KillMode=process
  Restart=always ##不管是什么退出原因，总是重启,即是杀不死
  RestartSec=0s ##Systemd重启服务之前，需要等待的秒数，要立即重新启动，则设置等待的秒数为0秒
  
  [Install]
  WantedBy=multi-user.target
  ```

  ## 阮一峰老师systemd教程跟练

  ### 系统管理与unit相关代码教程跟练

  [![系统管理与unit相关代码阮一峰老师代码教程跟练](https://asciinema.org/a/n3nVJBuQ1HYvMAgmvAPzt5xyK.svg)](https://asciinema.org/a/n3nVJBuQ1HYvMAgmvAPzt5xyK)

  ### unit相关指令跟练

  [![unit相关指令跟练](https://asciinema.org/a/K4Jq3SSYj4kDjg7RjHOEfuISw.svg)](https://asciinema.org/a/K4Jq3SSYj4kDjg7RjHOEfuISw)

  ### unit的配置文件查看与管理命令跟练

  [![unit的配置文件查看与管理命令跟练](https://asciinema.org/a/SNe9JbAMEVTGtfLQ5ZgXb82kx.svg)](https://asciinema.org/a/SNe9JbAMEVTGtfLQ5ZgXb82kx)

### 一些服务单元启动与停止的指令以及查看配置文件相关指令的跟练

[![一些服务单元启动与停止的指令以及查看配置文件相关指令的跟练](https://asciinema.org/a/INAGE8XON6Hq3lg5EBO3C0j4l.svg)](https://asciinema.org/a/INAGE8XON6Hq3lg5EBO3C0j4l)

## 参考资料：

1. [添加用户并且赋予sudo权限]: https://blog.csdn.net/breeze5428/article/details/52837768

2. [添加用户到用户组]: 、https://linux.cn/article-10768-1.html#:~:text=%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8%20usermod%20%E5%91%BD%E4%BB%A4%E5%B0%86%E7%8E%B0%E6%9C%89%E7%9A%84%E7%94%A8%E6%88%B7%E6%B7%BB%E5%8A%A0%E5%88%B0%E6%AC%A1%E8%A6%81%E7%BB%84%E6%88%96%E9%99%84%E5%8A%A0%E7%BB%84%EF%BC%9F.%20%E8%A6%81%E5%B0%86%E7%8E%B0%E6%9C%89%E7%94%A8%E6%88%B7%E6%B7%BB%E5%8A%A0%E5%88%B0%E8%BE%85%E5%8A%A9%E7%BB%84%EF%BC%8C%E8%AF%B7%E4%BD%BF%E7%94%A8%E5%B8%A6%E6%9C%89%20-G%20%E9%80%89%E9%A1%B9%E5%92%8C%E7%BB%84%E5%90%8D%E7%A7%B0%E7%9A%84%20usermod%20%E5%91%BD%E4%BB%A4%E3%80%82.,-a%20-G%20mygroup%20user1.%20%E8%AE%A9%E6%88%91%E4%BD%BF%E7%94%A8%20id%20%E5%91%BD%E4%BB%A4%E6%9F%A5%E7%9C%8B%E8%BE%93%E5%87%BA%E3%80%82.%20%E6%98%AF%E7%9A%84%EF%BC%8C%E6%B7%BB%E5%8A%A0%E6%88%90%E5%8A%9F%E3%80%82.

3. [畅课讨论区同学提供的自动挂载的经验]: http://courses.cuc.edu.cn/course/82669/learning-activity/full-screen#/248350/topics/251929?show_sidebar=false&amp;scrollTo=topic-251929&amp;pageIndex=1&amp;pageCount=1&amp;topicIds=251929,251926,247732,243484,239623&amp;predicate=lastUpdatedDate&amp;reverse

4. [手动挂载Virtualbox的共享目录分区]: https://blog.csdn.net/skylake_/article/details/53132499

5. [网络上自动挂载的方法]: https://cloud.tencent.com/developer/article/1467688

6. [vim的使用方法]: https://blog.csdn.net/weixin_38208741/article/details/78862368

7. [进入/etc/systemd/system目录使用open-vm-tools配置ubuntu共享文件夹]: https://www.jianshu.com/p/04b47cb172e1

8. [Linux中利用LVM实现分区动态扩容]: https://blog.csdn.net/seteor/article/details/6708025

9. [通过systemd设置实现在网络连通时运行一个指定脚本]: https://www.golinuxcloud.com/run-script-at-startup-boot-without-cron-linux/

10. [systemd入门篇]: https://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-commands.html

11. [systemd实战篇]: https://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-part-two.html

## 遇到的问题：

1. 在为用户添加了用户组和权限之后不知如何查看与验证结果，通过参考**资料2**了解到，使用`id`可查看此用户存在于哪些用户组。

2. 不会使用`vim`，困在`vim`里面出不来。查找**资料7**去补了vim编辑的知识。
3. 使用语句`lvresize --size +10G --resizefs demo-vg/demo-lv-1`，收到报错：`Insufficient free space: 2560 extents needed, but only 0 available`，反复检查思考原因后发现，当卷组中的剩余空间不足时，当然不能为逻辑卷扩容，上面有一步中已经耗尽了卷组的剩余空间，free vg=0，故最后再使用此句进行扩容不能成功，但可以缩减。由此可见`lvextend`和`lvresize`，均可以扩展逻辑卷，实现分区扩容。
4. 经过尝试发现，某分区处于挂载状态是不能进行LVM操作的，故不要在把分区挂载之后对其进行相关操作，否则先`unmount`移除掉才可以。