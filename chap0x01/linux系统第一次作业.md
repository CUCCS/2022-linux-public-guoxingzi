# linux系统第一次作业

- ## 调查并记录实验环境的如下信息：

  - ### 当前 Linux 发行版基本信息

  - ### 当前 Linux 内核版本信息

    ### 1.在ubuntu系统中

    应用到的语句:

    1.`cat /etc/os-release`
    2.`cat /etc/issue`
    3.`lsb_release -a`
    4.`less /etc/os-release`
    5.`cat /etc/*release*`
    6.`ls /etc/*release* #  *是通配符，查找带有release的语句`

    ![第一张，ubuntu中查看版本信息](image-20220310183300084.png)

    ![第二张，ubuntu中查看版本信息](image-20220310183416711.png)

    ### 2.在阿里云终端，centos环境中,对比可以发现，在不同的环境中，查看发行版本的信息具有差异化。

    应用到的语句:

    1.`cat /etc/*release*`
    2.`cat /etc/centos-release`
    3.`cat /etc/redhat-release`
    4.`ls -l /etc/*release*`

    ![第三张，centos环境中查看版本信息](image-20220310183509811.png)

    ![第四张，centos环境中查看版本信息](C:\Users\86187\AppData\Roaming\Typora\typora-user-images\image-20220310183552364.png)

- ## Virtualbox 安装完 Ubuntu 之后新添加的网卡如何实现系统开机自动启用和自动获取 IP？

  1.在Ubuntu环境下是有目录的，可以通过语句`ls /etc/netplan/`来查看

  ![第五张，Ubuntu中查看netplan](image-20220310183632863.png)

  2.在阿里云终端，centos环境中，运用语句可以发现，不存在目录。

  ![第六张，centos环境中查看netplan](image-20220310183722607.png)

  3. **先利用语句`ip a`查看网卡，了解基本信息**，了解到第一块是虚拟网卡，第二块是`net`网卡，第三块是`host only`网卡。接下来，通过手动在虚拟机的设置中开启关闭网线来观察命令行中网卡状态的变化。

  **接入网线时，网卡状态一切正常：**

  ![第七张，网卡正常时的状态](image-20220310184309171.png)

  **断开第一块网卡，即`net`网卡的网线，可见`enpos3`处显示`NO-CARRIER`，说明断开了连接：**

  ![第八张，网卡断开时的状态](image-20220310183830084.png)

  4.手动在虚拟机设置中检查网卡与服务器的状态

  ![第九张，虚拟机设置中检查网卡与服务器的状态](image-20220310184133848.png)

  5.使用vim打开文件`sudo vim /etc/netplan/00-installer-config.yaml`来查看配置信息。

  为什么配置信息存在于这个文件中，查找资料发现：

  > ubuntu20网络的配置信息将不在保存在`/etc/network/interfaces`文件中，虽然该文件依然存在，但是     内容是空的。新系统已经使用netplan管理网络，对于配置信息，使用vim打开文件`sudo vim /etc/netplan/00-installer-config.yaml`

  这一步的操作参考资料来源：[网卡自动启用和自动获取ip](https://blog.csdn.net/xiongyangg/article/details/110206220)

  ![第十张，命令行查看配置信息](image-20220310184555434.png)

  可以发现网卡`enp0s3`和网卡`enp0s8`下面都有参数`dhcp4: true`，说明该两张网卡都开启了`dhch`地址分配

  ![第十一张，网卡状态显示](image-20220310184830882.png)

  若有网卡没有出现在检查结果中，那么就手动添加进去，例如：

  `enp0s8：` `dhcp4: true`

  最后执行语句`sudo netplan apply`生效

- ## 如何使用 `scp` 在「虚拟机和宿主机之间」、「本机和远程 Linux 系统之间」传输文件？

  ### 1.首先在在阿里云终端，centos环境中创建一个文件

  创建文件所用到的语句:

  1.`touch test # 用于创建空文件`
  2.`ls # 检查现有文件`
  3.`pwd #打印工作目录,它打印工作目录的路径，从根目录开始。`
  4.`cat test # 创建带有内容的文件`

  5.`which scp`

  ![第十二张，centos环境中创建一个文件](image-20220310184937559.png)

  **由于在实验过程中，遇到很多不明白意思的语句，因此查找资料了解到它的更多内容，各语句的详细内容与资料出处如下：**

  1. [**touch:**  ](https://www.geeksforgeeks.org/touch-command-in-linux-with-examples/)

     > The ***touch*** command is a standard command used in UNIX/Linux operating system which is used to create, change and modify timestamps of a file. Basically, there are two different commands to create a file in the Linux system which is as follows:
   >
     > - [**cat command**](https://www.geeksforgeeks.org/cat-command-linux-examples/): It is used to create the file with content.
     > - **touch command:** It is used to create a file without any content. The file created using touch command is empty. This command can be used when the user doesn’t have data to store at the time of file creation.
  
  2. [**ls&ll&ls -l**](https://www.geeksforgeeks.org/touch-command-in-linux-with-examples/)

     > we are in home directory and this can be checked using the *[pwd ](https://www.geeksforgeeks.org/pwd-command-in-linux-with-examples/)*command. Checking the existing files using command **ls** and then long listing command(ll) is used to gather more details about existing files.
   >
     > The file which is created can be viewed by **ls** command and to get more details about the file you can use **long listing command ll or ls -l** command .
  
  3. > [**pwd** ](https://www.geeksforgeeks.org/pwd-command-in-linux-with-examples/)
   >
     > **pwd** stands for **P**rint **W**orking **D**irectory. It prints the path of the working directory, starting from the root.
     > pwd is shell built-in command(pwd) or an actual binary(/bin/pwd).
     > $PWD is an [environment variable](https://www.geeksforgeeks.org/environment-variables-in-linux-unix/) which stores the path of the current directory.
     > This command has two flags.
     >
     > **pwd -L**: Prints the symbolic path.
     >
     > **pwd -P**: Prints the actual path.
  
     ### 2.在gitbash的桌面环境中使用scp进行文件的复制与传输

     #### 一. 将远程主机上的文件传输到本地主机，即从阿里云实验室centos环境中创建的文件传输到我gitbash所控制的桌面环境中

     应用到的语句：

     1.传输单个文件：`scp root@47.103.153.176:/root/test ./ #结构为scp 远程主机名@远程主机地址:/远程主机名/文件名 ./`

     2.传输文件夹及其目录：`scp -r root@47.103.153.176:/root/test-dir ./ #结构为scp -r 远程主机名@远程主机地址:/远程主机名/文件夹./  `该**-r**标志可用于递归复制文件夹及其内容而不是单个文件

  ![第十三张，桌面环境中使用scp进行文件的复制](image-20220310185118640.png)

  在gitbash所控制的本机桌面上出现了一个文件，正是在阿里云实验室centos环境中所创建的文件，说明远程传入本地成功

  ![第十四张，本机桌面出现图片](image-20220310185225980.png)

  #### 二.将本地主机上的文件传输到远程主机上，即把gitbash所控制的本机桌面上的一个文件传输到阿里云实验室centos环境中

  使用到的语句：

  1.`ls # 先用ls语句查看当前本机电脑桌面上所存在的文件,发现存在文件desktop.ini`

  2.`scp desktop.ini root@47.103.153.176:/root/ # 结构为scp 文件名 远程主机名@远程主机地址:/远程主机名/  `

  ![第十五张，从本地传输到云端](image-20220310185320375.png)

  最后，登陆阿里云实验室，进入centos环境，使用语句`ls -la desktop.ini`查看传输的目标文件desktop.ini，结果显示desktop.ini存在，故从本机上传到远程成功。

  ![第十六张](image-20220308202440774.png)

  

  **在使用scp过程中也遇到了非常多的问题，解决途径与使用资料如下**：

  [**scp**](https://codepre.com/en/como-usar-el-comando-scp-en-linux.html)

  

- ## 如何配置 SSH 免密登录？

  1.使用语句`ssh-keygen.exe`配置公私钥

  2.`ssh-copy-id -i ~/.ssh/id_rsa.pub cuc@192.168.56.101`

  3.尝试再次登陆，成功！

  遇到的问题：

  刚开始输入第一个语句后，命令行显示Enter file in which to save the key (/c/Users/86187/.ssh/id_rsa): 由于我从未配置过公私钥文件，因此我点击了回车，接着弹出询问/c/Users/86187/.ssh/id_rsa already exists.
  Overwrite (y/n)? y
  Enter passphrase (empty for no passphrase):
  Enter same passphrase again:

  我输入yes后，看到它问密码，我不知道是问什么密码，所以直接输入虚拟机的登陆密码cuc。

  接着输入第二个语句后，它继续向我询问虚拟机的密码，我仍旧输入cuc后进程完成，让我重新登陆试试看。然而我重新登陆后发现依然需要密码。

  我怀疑是我之前输入的那几个密码不对，便返回去仔细读它的要求。经过多次尝试后发现它所要求输入的密码是不同的，一个是rsa密码，一个是虚拟机密码。我尝试自定义rsa密码，并且将其与虚拟机密码区分开来。再次登录时发现，它先询问我rsa密码，多次输入不正确后询问我虚拟机密码。我忽然反应过来，我若是将rsa密码置为空值，则再次登录即不需要密码。故当命令行再次询问rsa密码时，我直接回车。

  最后重新登陆，显示成功！

  ps:还是要多读命令行，多试几次啊。

  ![第十七张，免密配置](image-20220310185455063.png)

