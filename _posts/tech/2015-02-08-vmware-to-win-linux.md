---
author: hancel.lin
date: 2015-02-08
title: 如何用VMware在U盤上安裝Windows和Linux系統的
tags: VMware,Windows,Linux,操作系统
images: /blog/img/vmware-to-win-linux/01.jpg
category: tech
layout: default
---

前言
---
長久以來，我想做成一件事兒，就是在U盤或移動硬盤上安裝Window和linux，這樣我就可以到任何一台電腦上使用我的開發平臺了。這件事兒，終於在上週完全的實驗成功了。

基本原理
---
將硬盤分成3個部分，第一部分為FAT32格式的DOS盤，第二部分為Linux分區，最後為Windows分區。

我們將DOS盤作為啟動盤，啟動grub程式，加載menu.lst。[grub](https://zh.wikipedia.org/wiki/GNU_GRUB)是linux下的一支启动引导程序，在這裡我們使用他來引導各個系統。對linux和Windows引導方式各有不同。linux主要通過加載Linux的內核來進行引導，而Windows則有自己的啟動器，所以事實上我們是通過加載Windows的啟動器([bootmgr](https://zh.wikipedia.org/wiki/Windows_Boot_Manager))來進行引導。

![原理图](/img/vmware-to-win-linux/pic_01.jpg)

開始準備
---
硬體：

1. 移動硬盤（推薦）、U盤
2. 普通PC

軟體：

1. 虛擬機[VMware Player](https://my.vmware.com/web/vmware/downloads)
2. DiskGenius（使用U盤的話）
3. [grub4dos](http://pan.baidu.com/s/1bn1w8vP)
4. [bootice](http://pan.baidu.com/s/1bn1w8vP)
5. [ghost](http://pan.baidu.com/s/1bn1w8vP)

其他文件

1. Linux鏡像（如：[CentOS 7.0](http://mirror.neu.edu.cn/centos/7.0.1406/isos/x86_64/CentOS-7.0-1406-x86_64-DVD.torrent)）
2. Windows鏡像（如：Windows 7）
3. DOS系統（如：[DOS 7.1](http://pan.baidu.com/s/1i3zizGx)）

安裝DOS系統
---
網絡上有很多安裝DOS系統的工具，不過很多都是把整個硬盤變成DOS盤（或許是我找得不夠仔細），而我們需要的是將硬盤分出一部分空間安裝DOS系統。我使用的方法是比較特殊的，使用虛擬機進行DOS系統的安裝。

首先，**備份好U盤或移動硬盤的所有數據！**然後刪除所有分區，將硬盤分出大約100MB的空間格式化為FAT32的格式（不必太多，我的加上TC才14M）。移動硬盤只要用Windows的磁盤管理就行了，U盤就需要DiskGenius來分區了（請不要分配驅動號）。

![DOS分区](/img/vmware-to-win-linux/pic_02.jpg)

然後，啟動虛擬機，創建一個windows7虛擬系統。把移動硬盤添加到虛擬機裏面：

1.選擇“Edit virtual machine setting”，添加一個硬盤：

![edit setting](/img/vmware-to-win-linux/pic_03.jpg)

![add hardware](/img/vmware-to-win-linux/pic_04.jpg)

![hard disk](/img/vmware-to-win-linux/pic_05.jpg)

然後選擇“use a physical disk”，Device選擇你的移動硬盤，通常是最後一個（如果你只接入一個移動硬盤的話）。

![add physical disk](/img/vmware-to-win-linux/pic_06.jpg)

![choose Device](/img/vmware-to-win-linux/pic_07.jpg)

剩下的默認就好了。然後就可以在硬體列表中看到添加的磁盤，容量看對一下，應該就沒錯了。

![compare](/img/vmware-to-win-linux/pic_08.jpg)

然後，選擇“Floppy”，選擇下載的DOS img。記得勾上“connect at Power on”。**接著把創建虛擬機時創建的虛擬硬盤移除。**

![floppy setting](/img/vmware-to-win-linux/pic_09.jpg)

然後啟動虛擬機，就會進入DOS系統的安裝，如果提示要插入第二塊軟驅，就“Disconnect”掉Floppy，然後進入“Setting”選擇“DOS71_2.img”，再Connect上繼續就好了。其他沒啥可說的。

![dos second disk](/img/vmware-to-win-linux/pic_10.jpg)

按裝完後，如果不做任何修改，這樣的DOS盤也就只能在虛擬機裏面啟動而已了。在實際的電腦上啟動就會報錯，原因我還沒找到，不過可以做一些修改解決這個問題：

1. 先關閉虛擬機，這樣你才能對硬盤的數據做修改；
2. 去“文件夾和搜索選項”那裡把顯示系統文件和隱藏文件給打開；
3. 進入DOS盤，刪除裏面的config.sys（到這裡DOS盤就可以在PC上正常啟動了）
4. 新建ghost和grub文件夾；
5. 將grub4dos裏面的menu.lst和chinese文件夾裏的所有文件解壓到DOS盤裏的grub文件夾。
6. 將ghost解壓到ghost文件夾
7. 編輯autoexec.bat，在第二行前面插入一行，加上 `PATH=C:\;C:\DOS71\;C:\ghost\;C:\grub\` 最後保存。

至此，DOS系統就安裝完畢了。

安裝 Linux
---
在這一步，我們使用CentOS 7做為示例。

基本上都是Next…Next的事情，就不贅言了。說說特殊的地方。

1. 不要安裝引導，選擇自定義分區
2. 選擇標準分區
3. 記錄下根路徑的分区（如：sda5）

如下圖：

![no-install-grub](/img/vmware-to-win-linux/pic_11.jpg)

![std part](/img/vmware-to-win-linux/pic_12.jpg)

![sda5](/img/vmware-to-win-linux/pic_13.jpg)

安裝完成後，當然是進不去Linux系統的~~所以，我們就要——

引導Linux
---
這個時候啟動應該是進入DOS系統，

![dos_begin](/img/vmware-to-win-linux/pic_14.jpg)

啟動DOS後：

{% highlight bash %}
cd grub
grub.exe
{% endhighlight %}

這樣就可以啟動grub了（啟動grub後，切記不要移動鼠標，不然就會死機），啟動後畫面如下：

![grub view](/img/vmware-to-win-linux/pic_15.jpg)

選擇commandline，現在我們需要去加載Linux的內核，內核是位於boot目錄下，從我們剛才安裝的畫面可以看出，boot分區是另外獨立一塊空間。

先用cat指令查看硬盤分區（**cat後面有空格**）：

{% highlight bash %}
cat (hd0,
{% endhighlight %}

然後按下Tab鍵，可以看到有3個xfs分區，第一個通常就是boot分區。

![grub_01](/img/vmware-to-win-linux/pic_16.jpg)

{% highlight bash %}
cat (hd0,1)/
{% endhighlight %}

同樣按下Tab鍵，就可以看到這個分區下的文件，其中vmlinuz-3.10.0-123.el7.x86_64就是Linux的內核，不同版本的Linux應該會有所不同。這就說明了這個就是boot分區

![grub_02](/img/vmware-to-win-linux/pic_17.jpg)

在用同樣的方法查看其他分區，找到分區下包含dev文件夾的目錄。

![grub_03](/img/vmware-to-win-linux/pic_18.jpg)

然後將其設置為根目录

{% highlight bash %}
root (hd0,4)
{% endhighlight %}

然後用kernel命令加載Linux內核

{% highlight bash %}
kernel (hd0,1)/vmlinuz-3.10.0-123.e17.x86_64 ro root=/dev/sda5
{% endhighlight %}

按Tab可以補全文件名，上面指令中的sda5，就是最早我們在安裝時記錄根目錄的設備名。

接著設置initrd文件：

{% highlight bash %}
initrd (hd0,1)/initramfs-3.10.0-123.el7.x86_64.img
{% endhighlight %}

然後就可以引導了。

{% highlight bash %}
boot
{% endhighlight %}

至此，你就成功引導了linux系統~

![grub_04](/img/vmware-to-win-linux/pic_19.jpg)

可是！難道每次我們進入系統都有打這麼一長串嗎？當然不是。我們現將liunx關機，然後**將grub裏面的menu.lst文件拷貝到DOS盤的根目錄**，然後在文件最後面加入：

{% highlight bash %}
title CentOS
root (hd0,4)
kernel (hd0,1)/vmlinuz-3.10.0-123.e17.x86_64 ro root=/dev/sda5
initrd (hd0,1)/initramfs-3.10.0-123.el7.x86_64.img
boot
{% endhighlight %}

然後在autoexec.bat最後一行加上

{% highlight bash %}
call \grub\grub.exe
{% endhighlight %}

在啟動DOS時，就可以看到CentOS選項了。以後只要通過這個選項就可以啟動CentOS了。

安裝Windows
---
安裝Windows最糟糕的地方就是，他居然死都要裝引導，所以如果採用和上面linux的方法，就會把DOS的引導給覆蓋掉。所以……我們依然是藉助虛擬機的幫助，快速將Windows安裝到移動硬盤上。

還記得我們最開始時移除的那個虛擬機自動創建的虛擬硬盤嗎？這個虛擬系統叫Window7可不是沒有道理的~嘿嘿嘿~

現在我們要把他重新添加回來，和添加移動硬盤略有不同，選擇“Use an existing virtual disk”，然後選那個磁盤文件就可以了。

![win7_back](/img/vmware-to-win-linux/pic_20.jpg)

然後在CD/DVD選擇一個Windows7的鏡像就可以了。在那個虛擬系統裏面安裝一個Win7系統即可。

**注意：如果啟動時仍然從dos啟動，就把移動硬盤移除後再啟動，另外分區時要預留大約10幾G的空間。**

安裝完成後，進入系統，將預留的空間格式化，創建一個盤出來。

為了把在虛擬機安裝好的Windows7轉移到移動硬盤上，我們就需要用Ghost，把Windows7 Ghost下來，然後在移動硬盤上還原。但是，當前Windows7是運行中的，所以必須進入到DOS下再對Windows7進行Ghost，那麼我們可以在啟動虛擬機時，按下F2鍵，進入BIOS設定，在boot設定DOS盤先啟動：

![bios setting](/img/vmware-to-win-linux/pic_21.jpg)

進入dos後，運行Ghost文件夾裏的ghost11.exe

{% highlight bash %}
cd ghost
ghost11.exe
{% endhighlight %}

進入ghost，選擇local&gt;Partition&gt;To Image

![ghost01.jpg](/img/vmware-to-win-linux/pic_22.jpg)

再選擇對應硬盤，與磁盤，再選擇保存在剛才新建的盤即可。

![ghost06](/img/vmware-to-win-linux/pic_23.jpg)

**注意要把系統保留分區一併Ghost**

![ghost07](/img/vmware-to-win-linux/pic_24.jpg)

![ghost08](/img/vmware-to-win-linux/pic_25.jpg)

待Ghost完畢，在進入Windows，在移動硬盤新建一個分區原來存放Windows，啟動DOS盤中的Ghost32將Windows恢復到移動磁盤的分區中。

local>Partition>From Image

選擇恢復時，無需恢復系統保留區的文件，只要選擇系統分區即可：

![ghost03](/img/vmware-to-win-linux/pic_26.jpg)

但是！這樣恢復後，實際上Windows是沒有任何引導的。

所以，我們需要把系統保留區裏的啟動文件提取出來，使用Ghost裏面的Ghostexp.exe打開備份的Ghost文件。將引導文件提取出來，拷貝到恢復的Windows磁盤裏。

![ghost09](/img/vmware-to-win-linux/pic_27.jpg)

然後，我們還需要修改一下裏面的BCD文件，BCD文件記錄的是與Windows啟動相關的信息（當然不會是文本的）。我們要使用bootice來修改：

選擇BCD編輯，選擇其他BCD文件，瀏覽剛剛恢復的Windows盤，選擇其中Boot的BCD文件：

![BCD Edit](/img/vmware-to-win-linux/pic_28.jpg)

編輯“啟動磁盤”和“啟動分區”，選中BCD文件所在分區。保存即可。

![Setting boot disk](/img/vmware-to-win-linux/pic_29.jpg)

至此，我們的Windows就安裝好了~那麼，接下來就是——

引導Windows
---

bootmgr是引導Windows的入口，那麼啟動根目錄就要設置為bootmgr所在的分區，可以直接使用

{% highlight bash %}
root (hd0,6)
{% endhighlight %}

設置根目錄，但其實下面這樣會更合理

{% highlight bash %}
find --set-root /bootmgr
{% endhighlight %}

找到bootmgr然後把其所在分區設置為根目錄。

然後，將bootmgr裝載進來：

{% highlight bash %}
chainloader /bootmgr
{% endhighlight %}

然後就可以啦~

那麼在menu.lst裏就是添加：

{% highlight bash %}
title Windows 7
find --set-root /bootmgr
chainloader /bootmgr
{% endhighlight %}

至此，DOS+Linux+Windows移動系統硬盤就搭建完畢啦~\*:‧\\(￣▽￣)/‧:\*
最後，附上我的menu.lst文件

{% endhighlight %}bash  
# This is a sample menu.lst file. You should make some changes to it.  
# The old install method of booting via the stage-files has been removed.  
# Please install GRLDR boot strap code to MBR with the bootlace.com  
# utility under DOS/Win9x or Linux.  

color black/cyan yellow/cyan
timeout 15
default /default

title CentOS 7
root (hd0,4)
kernel (hd0,1)/vmlinuz-3.10.0-123.el7.x86_64 ro root=/dev/sda5
initrd (hd0,1)/initramfs-3.10.0-123.el7.x86_64.img
boot

title Windows 7
find --set-root /bootmgr
chainloader /bootmgr

title DOS 7.1
savedefault --wait=2
quit

title Command Line
savedefault --wait=2
commandline

title Reboot
savedefault --wait=2
reboot

title Halt
savedefault --wait=2
halt
{% endhighlight %}
