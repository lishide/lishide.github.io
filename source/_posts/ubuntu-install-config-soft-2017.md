---
title: 一只小白的 Linux 之旅|Ubuntu 安装配置及软件安装（持续更新）
date: 2018-04-08 09:36:05
tags: [Ubuntu, OS, ]
---

记录 Ubuntu 系统安装配置及其软件安装的整个过程，同时分享给有需要的人。

![Ubuntu](http://p6wpxhpqt.bkt.clouddn.com/img_uos_ubuntu1604.png)
<!--more-->

## 1. U 盘安装 Ubuntu 和 Windows 双系统
### 1.1 下载 Ubuntu 的 `.iso` 文件
可以在 [Ubuntu 官网](https://www.ubuntu.com/download)或者 [Ubuntu 中文官网](https://cn.ubuntu.com/)上下载系统镜像文件。

### 1.2 制作 U 盘启动盘

 - 在电脑中插入一个空白的 U 盘（大小至少 2GB）
 - 使用 **Universal USB Installer** 烧写入 U 盘
 官方网站是 http://www.pendrivelinux.com/
 我们下载它：http://www.pendrivelinux.com/universal-usb-installer-easy-as-1-2-3/ 并且安装。

### 1.3 安装位置准备
- 打开硬盘管理器，在想要用来安装 Ubuntu 的盘符上右击选择“压缩卷”
- 输入压缩空间量（建议 40 G 以上，这里的单位是 M，自己换算），之后点击“压缩”
- 完成之后可以看到刚点击压缩后得来的硬盘分区已经处于了**未分配**的状态，这样安装位置的准备工作就完成了。
> 特别提醒：如果你的电脑是多硬盘，如固态硬盘和机械硬盘的组合，请一定在 BIOS 所在的磁盘上分配出 200~300 M 的空间，用于在后面的 Linux 分区时作为引导分区，否则会出现无法启动 Linux 系统的问题。
一般地，固态硬盘都用作了 Windows 系统的安装盘，则 BIOS 也在此，当然并不绝对，根据你电脑的实际情况确定。
当然了，若 BIOS 所在的磁盘容量足够大，并想将 Linux 系统安装到此磁盘，则不存在上面说的问题了，按照步骤设置分区大小即可。
而如果将系统安装在剩余空间较大却并非有 BIOS 的另一块磁盘上时，就要注意一下这个问题了。

### 1.4 重启电脑，选择 U 盘启动
一般都是按 F1 ~ F12 里面的一个键，具体是哪一个键请自行百度对应自己电脑牌子型号的按键。

### 1.5 正式安装 Ubuntu

 - 双击桌面上的“安装Ubuntu”图标，就进入了安装界面。
 - 在左边的栏目里选择安装语言，可以选择简体中文，点击下一步。
 - 选择安装 Ubuntu 过程中是否下载更新，可以不选；选择是否安装第三方软件，可以选择不安装，点击下一步。
 > 我基本都选择的是安装，导致安装过程特别长。
 - 这个页面选择在哪里安装 Ubuntu，**我们要安装双系统，所以选择“其他”**，点击下一步按钮。
 > 居然提示我，我的电脑没有安装任何操作系统，mdzz，于是选择“其他”。

 - 接下来进行分区，这一步很关键，决定了 Ubuntu 是否能成功启动。
首先我们看到有一个空闲分区（白色的那条），这个就是一开始我们在 Windows 下面给 Ubuntu 分出来的空间。操作这个“空闲”分区，鼠标选中它，点击下面的“+”号，会弹出对话框，按照下表中的挂载点名称、大小及格式进行分区。分区大小只是建议，可根据实际情况进行调整。

| 挂载点|分区大小|格式|
| --------   | -----:        | :----:      |
| swap       | 内存大小（8 G） |  交换空间     |
| ~~/boot~~  | ~~300 M~~     |  ~~引导分区~~ |
| /          | 20 - 30 G     | 逻辑分区 ext4 |
| /home      | 60 - 80 G     | 逻辑分区 ext4 |

~~将“安装启动引导器的设备”选择为之前分配`/boot`的那个分区名。如下图中，是`sda5`，选择你的电脑对应的分区名。~~

![我的 Ubuntu 分区大小设置](http://p6wpxhpqt.bkt.clouddn.com/img_uos_show_diskpart.jpg)

==更新==
在分区过程中，建议不划分 `/boot`分区，在使用中发现，在更新几次内核之后，就会提示空间不足，因此划分上面表格中其他三个分区即可，“安装启动引导器的设备”保持默认不变。
系统安装完成后，电脑使用 Linux 的引导，有 Linux 和 Windows 的引导选项，如需修改启动项，详见下文中“4.4 双系统引导项设置”。如此分区后，下面的在 Windows 中使用 EasyBCD 修改引导的步骤不用再做了。

> 分区操作参考下面教程：
http://www.jianshu.com/p/53b8b76439d0

 - 接下来根据提示选择时区，然后点击下一步。
 - 选择键盘布局，我们选择汉语，当然你也可以选择其他语言，点击继续，下一步
 - 设置用户名、计算机名、密码等。点击”继续”，下一步。

开始安装，请耐心等待。

安装完成，点击重启。

---------------分割线---------------

> 如果为 Ubuntu 系统划分了 `/boot` 分区，重启之后发现启动选项没有 Ubuntu，而是直接进了 Windows 系统，那就需要在 Windows 中，用 EasyBCD 软件来添加启动引导项了。

在 Windows 中安装 EasyBCD 后打开，点击“添加新条目（Add New Entry）” ，选择 `Linux/BSD`，具体设置如下图。类型（Type）选择 GRUB(Legacy)；名称（Name）自己随便写，小编写的是 Ubuntu 作为标识；驱动器（Drive）选取我们设置的 /boot 分区，有 Linux 标记。设置完成后点击“添加条目（Add Entry）”。

![EeayBCD](http://p6wpxhpqt.bkt.clouddn.com/img_uos_ease_bcd.jpg)

电脑再次启动时，就可以看到多了 Ubuntu 的启动选项了。

## 2. 系统更新与美化

### 2.1 系统更新
安装完系统之后，需要更新一些补丁。CTRL+ALT+T 打开终端，输入下面代码
``` bash
sudo apt-get update
sudo apt-get upgrade
```
### 2.2 系统美化

#### 2.2.1 调整 Unity 桌面环境：Unity Tweak Tool
``` bash
sudo apt-get install unity-tweak-tool
```
#### 2.2.2 主题 & 图标
Flatabulous 主题（扁平化主题）
``` bash
sudo add-apt-repository ppa:noobslab/themes
sudo apt-get update
sudo apt-get install flatabulous-theme
```
该主题有配套的图标，安装方式如下：
``` bash
sudo add-apt-repository ppa:noobslab/icons
sudo apt-get update
sudo apt-get install ultra-flat-icons
```
安装完成后，打开 unity-tweak-tool 软件，修改主题和图标。
![Unity Tweak Tool_主题](http://p6wpxhpqt.bkt.clouddn.com/img_uos_utt_theme.png)
![Unity Tweak Tool_图标](http://p6wpxhpqt.bkt.clouddn.com/img_uos_utt_icon.png)

#### 2.2.3 字体：文泉译微米黑字体
``` bash
sudo apt-get install fonts-wqy-microhei
```
然后通过 unity-tweak-tool 来更换字体，按照自己的喜好选择字体和字号。
![Unity Tweak Tool_字体](http://p6wpxhpqt.bkt.clouddn.com/img_uos_utt_font.png)

...
主题美化参考自：http://www.jianshu.com/p/71b60921972b

## 3. 软件安装
### 3.0 软件中心
3.0.1 gdebi——deb 安装器（`*.deb`文件右键选择打开方式）
``` bash
sudo apt install gdebi
```
> 这个软件是我安装搜狗输入法的时候发现的，刚上手不会装软件，系统总是提示不支持第三方balabala，在网上找到解决方法是安装此软件。

3.0.2 ubuntukylin 软件中心
3.0.3 ubuntu 软件中心

### 3.1 chrome 浏览器

谷歌浏览器，官网下载安装包。

### 3.2 搜狗输入法
首先去[搜狗拼音输入法官网](http://pinyin.sogou.com/linux/?r=pinyin)下载。
安装命令如下：
``` bash
sudo dpkg -i sogoupinyin_2.1.0.0082_amd64.deb
```
如果提示依赖有问题，执行下面命令后重新安装：
``` bash
sudo apt-get install -f
```
或者右键安装包，*打开方式* 选择 **GDebi 软件包安装程序** 打开

### 3.3 科学上网
#### 3.3.1 lantern
蓝灯，[GitHub repo 地址](https://github.com/getlantern/lantern)。

#### 3.3.2 shadowsocks
Shadowsocks-qt5 是一款支持 Windows 和 Linux 的 Shadowsocks 客户端，通过 PPA 源安装，仅支持 Ubuntu 14.04 或更高版本。
``` bash
sudo add-apt-repository ppa:hzwhuang/ss-qt5
sudo apt-get update
sudo apt-get install shadowsocks-qt5
```
安装完之后查找 shadowsocks 软件，点击运行，然后配置你的信息即可。

### 3.4 Git
Git，程序员必备软件，安装：
``` bash
sudo apt-get install git
```
配置用户名和邮箱：
``` bash
git config --global user.name "Your Name"
git config --global user.email "email@example.com"
```
配置SSH
``` bash
ssh-keygen -C 'you email address@gmail.com' -t rsa
```
此时在 **目录：~/.ssh/** 下面会建立相应的密钥文件。
创建完公钥后，需要上传到 GitHub、码云或者自己配置的 GitLab 上。
使用命令进入 **~/.ssh** 文件夹：
``` bash
cd ~/.ssh
```
打开 **id_rsa.pub** 文件：
``` bash
gedit id_rsa.pub
```
，复制其中所有内容，配置到相应的 SSH 配置中。

测试连接是否畅通：
``` bash
// GitHub
ssh -T git@github.com
// 码云
ssh -T git@gitee.com
// ...
```

### 3.5 JDK
[JDK 官网](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html) 下载 Linux 的 JDK 版本。

- 在 `/usr/lib` 目录下创建一个文件夹 **jvm**
``` bash
sudo mkdir /usr/lib/jvm
```

- 进入默认的下载目录下 `home/下载/` 进行解压文件
``` bash
sudo tar -zxvf jdk-8u112-linux-x64.tar.gz
```

- 将解压后的 jdk 移动到 `/usr/lib/jvm`目录下
``` bash
sudo mv jdk1.8.0_121/ /usr/lib/jvm/
```

- 接下来配置系统环境变量，这里是将环境变量配置在 `etc/profile`，即为所有用户配置 JDK 环境，使用命令：
``` bash
sudo gedit /etc/profile
```
在文件末尾追加：
``` bash
#set java environment
export JAVA_HOME=/usr/lib/jvm/jdk1.8.0_121
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
```

- 执行下面命令使当前 bash 环境生效
``` bash
source /etc/profile
```

- 查看是否安装成功
运行命令：
``` bash
java -version
```
显示版本信息，则说明安装和配置成功。

### 3.6 Android Studio
下载地址：[Android Studio Linux版](https://developer.android.google.cn/studio/index.html)

- 安装及运行命令：
``` bash
unzip android-studio-ide-145.3360264-linux.zip  //解压
sudo mv android-studio /opt/  //移动
cd /opt/android-studio/bin  //切换文件夹
sh studio.h（sudo ./studio.sh）  //运行
```
按照提示进行安装。

- 如果运行的是 64 位版本 Ubuntu，则需要使用以下命令安装一些 32 位库：
 ``` bash
sudo apt-get install lib32z1 lib32ncurses5 lib32bz2-1.0 lib32stdc++6
```

> 部分说明来自于官网，更多安装过程细节可参考[官方安装 Android Studio 说明](https://developer.android.google.cn/studio/install.html)。


- 创建 AS 在 Ubuntu 中的快捷方式：
点击`Tools-->create Desktop entry` 创建

### 3.7 VScode
下载地址：[Visual Studio Code](https://code.visualstudio.com/Download)

### 3.8 CPU、内存、网速监控
GitHub 地址：[https://github.com/fossfreedom/indicator-sysmonitor](https://github.com/fossfreedom/indicator-sysmonitor)
- 安装
``` bash
sudo add-apt-repository ppa:fossfreedom/indicator-sysmonitor
sudo apt update
sudo apt install indicator-sysmonitor
```
安装完成之后输入`indicator-sysmonitor &`回车，之后按`Ctrl+C`
，继续输入`exit`退出控制台，这时此工具就在后台运行了。
- 开机启动
鼠标左击标题栏该工具图标，弹出下拉菜单选择`Preferences`，勾选上`Run on startup`，此时该工具就会开机自动启动了。
- 配置
切换到`Advanced`选项，可以对要显示的信息的格式进行设置，如：我想知道当前网络的上下行速度，就加上`网速:{net}`即可。

### 3.9 RapidSVN
SVN 客户端
``` bash
apt-get install rapidsvn
```
使用教程，[点我](http://jingyan.baidu.com/article/647f01159232ee7f2048a85d.html)。

### 3.10 Shutter
Ubuntu下很强大的一款截图软件。
``` bash
sudo apt-get install shutter
```
**设置 Shutter 快捷键：**
打开系统设置-->键盘-->快捷键-->自定义快捷键：

|  功能 | 命令 | 我的快捷键组合 |
| --------   | -----:  | :----:  |
| 选中区域截取图片（Shutter Select）    | shutter -s | Shift + Alt + s |
| 截取活动窗口（Shutter Active）           | shutter -a | Shift + Alt + a |
| 截取整个屏幕（Shutter Full）               | shutter -f | Shift + Alt + f |
| 选择要捕获的窗口（Shutter Window） | shutter -w | Shift + Alt + w |

更多功能请运行 `shutter -h` 查看帮助。

### 3.11 WPS
WPS，办公套件。
#### 3.11.1 下载安装软件
软件中心下载或去[官网](http://www.wps.cn/)下载 `.deb` 安装包进行安装。

#### 3.11.2 安装字体
第一次启动的时候会报错，说你有很多字体没有安装，这时，你可以去网上自己找或者下面这个链接[必备字体库](http://download.csdn.net/detail/wl1524520/6333049)下载安装字体。

字体安装方法：

- 创建目录
``` bash
sudo mkdir /usr/share/fonts/wps-office
```

- 将下载的字体复制到创建的目录
``` bash
sudo cp -r wps_symbol_fonts.zip /usr/share/fonts/wps-office
```

- 解压字体包
``` bash
sudo unzip wps_symbol_fonts.zip
```

- 解压后删除字体包
``` bash
sudo rm -r wps_symbol_fonts.zip
```

### 3.12 Haroopad
Markdown 编辑器，[官网](http://pad.haroopress.com/user.html)下载 `.deb` 安装包，右键安装。

### 3.13 Beyond Compare
文件及文件夹（目录）对比工具，[官网](http://scootersoftware.com/)下载 `.deb` 安装包，右键安装。

### 3.14 PhpStorm
PHP 开发工具，[官网](http://www.jetbrains.com/phpstorm/)下载 `.tar.gz` 压缩包，按照如下步骤安装。

- 从官网下载压缩包后，进入下载的文件夹，并解压 PhpStorm
``` bash
tar -xvf PhpStorm-2017.2.tar.gz
```
- 移动到目标文件夹（我将其移到了  `opt` 文件夹里）
``` bash
sudo mv PhpStorm-172.3317.83/ /opt/
```
- 进入文件夹并打开软件
``` bash
cd /opt/PhpStorm-172.3317.83/bin/
./phpstorm.sh
```
按照提示进行安装。

更多 Linux 安装 PhpStorm 的步骤请参考：http://www.linuxidc.com/Linux/2016-05/131373.htm

### 3.15 搭建 LAMP 环境
最简单的安装方式，注：别忘了最后这个脱字符号`^`，否则终端会报无法定位软件包的错误提示。
``` bash
sudo apt-get install lamp-server^
```
安装 phpMyAdmin
``` bash
sudo apt-get phpmyadmin
```
更多关于测试是否安装成功及修改一些配置的地方，参考这两篇文章：
1、[在Ubuntu上安装LAMP服务器系统的方法](http://www.jianshu.com/p/48c1a892f90b)
2、[Ubuntu16.04搭建LAMP环境](https://blog.luckyw.cn/2016/08/21/ubuntu-lamp/)

### 3.16 福昕 PDF 阅读器
PDF 阅读器，[下载地址](https://www.foxitsoftware.cn/downloads/)。
- 下载完成后，切换到文件所在目录

- 解压可执行文件
``` bash
gzip -d 'FoxitReader2.4.1.0609_Server_x64_enu_Setup.run.tar.gz'
```
- 对`.tar`文件进行解包
``` bash
tar xvf 'FoxitReader2.4.1.0609_Server_x64_enu_Setup.run.tar'
```
- 移动到目标文件夹（可选，我将其移到了 `opt` 文件夹里）
``` bash
sudo mv FoxitReader.enu.setup.2.4.1.0609\(r08f07f8\).x64.run /opt/
```
- 切换到 root 用户
比官方给的多这一步，为了有权限将软件安装到`opt/` 目录，否则直接进行下一步只能安装到用户目录下了。
``` bash
su root
```
- 运行安装程序
``` bash
./'FoxitReader.enu.setup.2.4.1.0609(r08f07f8).x64.run'
```
根据屏幕提示完成安装，同 Windows 系统安装软件过程。
> 具体的版本号，以下载的实际情况为准。

### 3.17 Postman
API 调试工具，[官网](https://www.getpostman.com/postman)下载 `.tar.gz` 压缩包，按照如下步骤安装。

- 从官网下载压缩包后，进入下载的文件夹，并解压 Postman
``` bash
tar -xvf Postman-linux-x64-5.3.2.tar.gz
```
- 移动到目标文件夹（我将其移到了 `opt` 文件夹里）
``` bash
sudo mv Postman/ /opt/
```
- 进入文件夹并打开软件
``` bash
cd /opt/Postman/
./Postman
```
到此软件就安装完成了，但是 Postman 没有为用户生成启动项，这样每次都要进入这个目录，且启动器没有该应用。

- 创建全局变量，也就是在任何地方都可以执行 postman，不用去到安装目录，执行
``` bash
sudo ln -s /opt/Postman/Postman /usr/bin/postman
```
- 添加启动器应用图标，也就是可以从启动器快速启动，执行
``` bash
sudo gedit /usr/share/applications/postman.desktop
```
添加如下内容：
``` bash
[Desktop Entry]
Encoding=UTF-8
Name=Postman
Exec=postman
Icon=/opt/Postman/resources/app/assets/icon.png
Terminal=false
Type=Application
Categories=Development;
```
到启动器搜索，应该可以找到该应用了。

---
### 小结

#### 软件安装与卸载命令
* apt-get 安装方法
``` bash
~$ sudo apt-get install 软件名
```
如果出现依赖问题只需要键入命令更新一下依赖：
``` bash
~$ sudo apt-get -f install
```

* dpkg -i 安装方法
``` bash
~$ sudo dpkg -i xxx.deb
```
xxx 是具体的 deb 软件包的包名。
或者右键安装包，*打开方式* 选择 **GDebi 软件包安装程序** 打开。

* 卸载软件
``` bash
sudo apt-get autoremove --purge xxx
```
sudo：获取 root 权限；
apt-get：执行安装卸载功能的软件；
autoremove：告诉 apt-get 我们所要做的操作是移除软件；
--purge：注意这前面是两个短划线，这个参数是告诉他们要完整的干净的彻底的移除；
不清楚软件的具体包名，可以在输入开头几个字母之后按 Tab 键提示。

## 4. 问题解决
### 4.1 双系统时间修正

在 Ubuntu16.04 之前的版本中，可以在终端中输入`sudo gedit /etc/default/rcS`，将`UTC=yes` 改为 `UTC=no`
而在 Ubuntu16.04 和 16.10 版本中，在终端中输入`timedatectl set-local-rtc 1 --adjust-system-clock`即可关闭 UTC。

### 4.2 开机开启数字小键盘
- 安装 numlockx
``` bash
sudo apt-get install numlockx
```
- 完成后修改
``` bash
sudo gedit /etc/rc.local
```
在最后一行 `exit 0` 前增加如下内容
``` bash
if [-x /usr/bin/numlockx ]; then

numlockx on

fi
```
重启或者注销即可。

### 4.3 Ubuntu 设置 root 密码
设置 root 密码
``` bash
sudo passwd
```
输入两次密码。
切换root用户
``` bash
su root
```

### 4.4 双系统引导项设置
默认情况是 Linux 系统在引导的第一个条目，Windows 系统在最后一个，默认启动项停留在第一个。如果你想设置不用手动选择的情况下默认进入的系统，则可以进行如下的修改。

- 打开配置文件
```bash
sudo gedit /etc/default/grub
```

- 修改默认启动项
GRUB_DEFAULT=0 设置成 GRUB_DEFAULT=4。
> 索引值是从 0 开始的，第一个是 Ubuntu，第 5 个是 Windows，这样就可以设置开机默认启动 Windows。

- 修改开机默认等待时间
GRUB_HIDDEN_TIMEOUT= 秒数

- 更新设置，使其有效
```bash
sudo update-grub
```

## 5. 结尾
文章持续更新中，遇到好的应用或美化相关的，会更新上来；文章中有不完美的地方，也请大佬指点出来，将做出修改和优化。希望这篇文章能帮到有需要的人，点滴积累，点滴分享。
