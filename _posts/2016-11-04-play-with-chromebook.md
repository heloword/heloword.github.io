---                                                                        
layout: post
title: Play With ChromeBook
date: 2016-11-04
categories:
- os
- hardware
tags: chromeos
---
##### **引子**
就在前天，自己之前辛辛苦苦海淘回来的Acer C720的屏幕裂了，虽然这个本配置不高，2955U的CPU，2G内存加32G的SSD，但是苦于目前手头又没有其它的电脑，于是只能“缝缝补补”了，希望再扛一段时间。淘宝了一圈，只发现了一家卖这款电脑屏幕的，于是简单跟店主了解之后下单。
等了两天终于拿到屏幕，二话不说开始屏幕的拆换，Acer C720拆机比较简单，毕竟只是一台廉价的ChromeBook。拆机参考了油管的视频[Acer Chromebook C720 Cracked Screen Replacement Procedure](https://www.youtube.com/watch?v=yOjkMZR47AY)，只要手能动的基本拆换起来都没有什么问题。插上屏幕接口之后开机，屏幕不亮，还以为是电脑有毛病了，经过一番“置换变量”，最后发现是接口插的不够紧，又使劲插插解决问题。然后网上找了点纯色图片，检查了一下没有亮点和坏点，于是确认收货，大功告成。
不过鉴于这个本不知道什么时候就会光荣牺牲，所以还是把以前配置的过程记录下来，以便于以后参考（说不定啥时候脑子一抽就买了个ChromeBook Pixel了呢）。

##### **安装**
ChromeBook，内置Chrome OS，我觉得还可以称之为“自带硬件的Chrome”，不过当你进入开发者模式之后，新世界的大门将向你打开：

1. 按住 Esc+F3 (Refresh)，然后按 Power，接着就会进入恢复模式。
2. 接着，按 Ctrl+D，它会提示您确认进入开发者模式，您的数据会被清除。
3. 再次按  Ctrl+D，或者等待 30 秒左右，系统会引导您进入开发者模式。

上述步骤参考[Arch的wiki](https://wiki.archlinux.org/index.php/Acer_C720_Chromebook)（这篇wiki可以让你实现在ChromeBook上完全安装一个ArchLinux，感兴趣的可以自己研究）。

进入开发者模式之后，我们就可以干很多事情了，比如说可以在Chrome中CTRL-ALT-T调出来终端，还可以安装开发者工具包来使用一些基础的GNU工具（在翻墙的情况下也压根没有完全安装成功过，好坑），不过最重要的是我们可以用[chroot](https://en.wikipedia.org/wiki/Chroot)来安装其他Linux发行版了，与Chrome OS共用内核，目前已有大神写出了成熟的工具[crouton](https://github.com/dnschneid/crouton)以及其衍生版[chroagh](https://github.com/drinkcat/chroagh)。关于其使用方法可以自行研究readme解决，我主要记录一下使用crouton安装kali-rolling的技巧，防止在没有全局VPN的情况下抓瞎：

1. 已发布的最新的crouton脚本的地址在[这里](https://goo.gl/fd3zc)，下载之后进入脚本所在目录并运行：` sudo sh ./crouton ` 你会发现其需要联网去下载很多东西，原来这个脚本只是一个壳，如果想查看具体源码，执行` sudo sh ./crouton -x `就会把所有的脚本解压到当前目录的一个文件夹中了，然后就可以把源码翻个底朝天了。
2. 由于在安装桌面环境时，需要在chroot的系统中连接google服务器下载CRAS的驱动进行编译安装，为了防止没有全局VPN翻墙会失败这种情况，我们先安装一个只有CLI版本的kali，后续再加装桌面（这里为了下载加速使用了中科大的kali-rolling源）。
`sudo sh ./crouton -r kali-rolling -t cli-extra -n kali -m http://mirrors.ustc.edu.cn/kali `
3. 安装完成之后，如果不考虑桌面，我们就拥有了一个可以完整执行的Linux系统，可以通过` sudo startcli`进入。但是想安装桌面怎么办呢？由于必须在Chrome OS中用全局VPN才可以实现chroot内部的翻墙，所以这里对chroot采用hosts大法。通过替换已安装的Linux系统内的hosts文件，实现其对google的访问，也就避免了使用全局VPN。这里推荐一个[hosts](https://raw.githubusercontent.com/racaljk/hosts/master/hosts)，下载之后可以直接在Chrome OS中直接替换`/usr/local/chroots/kali/etc/hosts`文件，然后在kali中就可以直接翻墙了。由于我需要安装i3wm，所以此时只需要执行`sudo sh ./crouton -u -n kali -t x11`安装x11服务就行了，不带有任何桌面，后续过程参考[Using i3 in crouton](https://github.com/dnschneid/crouton/wiki/i3) 。（如果想把chroot中的hosts换回来，那就在替换之前备份，之后再替换回原来的就行了）

##### **配置（Linux）**
这里主要备份一下使用Crouton安装好的新系统（基于Debian的Kali发行版）中可能遇到的问题以及解决方法：

1. 中文乱码。
   * 安装locales：` sudo apt-get install locales `
   * 配置locales：` sudo dpkg-reconfigure locales `
   * 在弹出的界面中选择所有以en_US和zh_CN开头的locales，然后选择默认的locales为en_US.UTF-8。（因为我默认使用英文系统）
   * 安装中文字体：` sudo apt-get install fonts-wqy-microhei fonts-wqy-zenhei xfonts-wqy `
   * 安装一些好看的英文字体（可选）：` sudo apt-get install fonts-inconsolata xfonts-terminus `
   * 刷新字体缓存（可选）：fc-cache
   * 查看已安装字体（可选）：fc-list
   * 参考[ Kali Rolling 解决中文乱码问题](http://blog.csdn.net/bleachswh/article/details/51419670)

2. 安装中文输入法：
   * 安装fcitx框架：` sudo apt-get install fcitx `
   * 在~/.config/i3/config中增加fcitx的启动：`exec --no-startup-id fcitx -d`
   * 在~/.xinitrc头部增加：
   `export GTK_IM_MODULE=fcitx `
   `export QT_IM_MODULE=fcitx `
   `export XMODIFIERS="@im=fcitx"`
   * 若在终端执行fctix出现**/bin/dbus-launch terminated abnormally without any error message**的错误，则说明需要安装D-Bus，执行`sudo apt-get install dbus-x11`，这样fctix框架就可以启动了。
   * 安装google拼音：`sudo apt-get install fcitx-googlepinyin`

3. 64位系统上安装32位库：
   * 在源中增加i386架构：`dpkg --add-architecture i386`
   * 更新源：`sudo apt-get update;sudo apt-get upgrade`
   * 之后在相应的库名字后面增加:i386即可安装对应的32位版本

4. IDA运行的问题：
   * 查找缺失的依赖属于哪个库：`sudo dpkg -S libgthread-2.0.so.0`
   * 安装对应的32位版本：`sudo apt-get install libglib2.0-0:i386`
   * 可以使用以上两部解决所有的库依赖问题。
   * 遇到QT的xcb错误：` sudo apt-get install libqt5gui5:i386`
   * 参考[Error while loading shared libraries: libgthread-2.0.so.0](http://askubuntu.com/questions/427496/error-while-loading-shared-libraries-libgthread-2-0-so-0) 
 
5. Java程序字体虚化问题：
   * 可以通过设置环境变量解决，在**/etc/environment**中增加`export _JAVA_OPTIONS='-Dawt.useSystemAAFontSettings=lcd'`
   * 参考[Java Runtime Environment fonts](https://wiki.archlinux.org/index.php/Java_Runtime_Environment_fonts)

主要花时间解决的问题大概就这么多，还有一些应用的配置，比如i3wm和xterm什么的不在此列。

##### **小结**
这篇文章的目的主要是备份自己的使用经历，方便下次使用时解决问题，如果恰巧也能给其他拥有chromebook的小伙伴一些帮助那就更好啦。
