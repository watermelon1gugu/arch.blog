---
title: 浅析systemd
date: 2019-09-09 12:19:12
tags: systemd
---

## 参考文献

《鸟哥的Linux私房菜》

## daemon与service

+ service是系统为了实现某些功能而必须要提供的一些服务。

+ service的提供总是需要程序的运行。所以达成这个service的程序称为daemon。

+ 比如达成循环型例行性工作调度服务（service）的程序 就是 crond这个daemon

+ 达成某个服务需要一支daemon在背景中运行，没有这支daemon就不会有service
+ daemon是一只程序(program)执行后的程序(process),通常daemon所处的原本程序(program)命名为{xxx}d.例如atd和crond

## systemd的好处

+ 平行处理所有服务，相较于init。systemd可以平行启动所有没有相互依赖的服务。
+ 一经要求就回应(on-demand)(按需启动)的启动方式：systemd全部就是仅有一只systemd服务搭配systemctl指令来处理。此外，systemd常驻内存，当有任何需要（on-demand)时，都可以立即处理后续的daemon启动服务。
+ 服务相依性自我检查：由于systemd可以自订服务相依性的检查，因此如果B服务是架构在A服务上启动的，那么在没有启动服务A的情况下启动服务B，systemd会自动启动服务A。
+ 依daemon功能分类：systemd管理的服务非常多，为了清晰服务的功能，首先systemd先定义所有的服务为一个服务单位(unit)，并将该unit归类到不同的服务类型去(type)，systemd将服务单位区分为service,socket,target,path,snapshot,timer等不同的类型(type)。
+ 将多个daemons集合成一个群组。systemd可以将许多个功能集合为一个所谓的target项目，这个项目只要在设计操作环境的创建，所以是集合了许多的daemons，亦即是执行某个target就是执行好多个daemon的意思
+ 向下兼容旧的init服务脚本：基本上，systemd是可以相容于init的服务脚本的。因此，就的init启动脚本也能通过systemd来管理。

## system配置文件目录

基本上，systemd将过去所谓的daemon执行脚本通通称为一个服务单位(unit),而每种服务依据功能分为了不同的类型(type)。基本的类型有系统服务、数据监听和交换的插槽档服务(socket)、储存系统状态的快照类型、提供不同类似执行等级分类的操作环境(target)等等。所有的配置文件都放置于下面的目录中：

+ /usr/lib/systemd/system/：每个服务最主要的启动脚本设置，类似于/etc/init.d下面的文件
+ /run/systemd/system/：系统执行过程中所产生的服务脚本，这些脚本的优先级要高于/usr/lib/systemd/system/
+ /etc/systemd/system/：管理员依据主机系统的需求所创建的执行脚本，优先级高于/run/systemd/system/

所以，系统开机时会不会执行某些服务，是根据/etc/systemd/system/下的设置（优先级高），所以该目录下是一大堆链接文件，而实际执行的systemd启动脚本配置文件都是放置在/usr/lib/systemd/system/下面的。因此，如果需要修改某个服务启动的设置，应该要去/usr/lib/systemd/system/下面修改才对，/etc/systemd/system/仅是链接到正确的执行脚本配置文件而已。

## systemd的unit分类说明

/usr/lib/systemd/system/以下的数据根据扩展名进行区分type。

| 扩展名            | 主要服务功能                                                 |
| ----------------- | ------------------------------------------------------------ |
| .service          | 一般服务类型(service unit):主要是系统服务，包括服务器本身所需要的本机服务以及网络服务都是，比较经常使用到的服务大多是这种类型，所以也是最常见的类型！ |
| .socket           | 内部程序数据交换的插槽服务 （socket unit）：主要是 IPC （Interprocess communication） 的传输讯息插槽档 （socket file） 功能。 这种 类型的服务通常在监控讯息传递的插槽档，当有通过此插槽档传递讯息来 说要链接服务时，就依据当时的状态将该用户的要求传送到对应的 daemon， 若 daemon 尚未启动，则启动该 daemon 后再传送用户的要 求。使用 socket 类型的服务一般是比较不会被用到的服务，因此在开机时 通常会稍微延迟启动的时间 （因为比较没有这么常用嘛！）。一般用于本 机服务比较多，例如我们的图形界面很多的软件都是通过 socket 来进行本 机程序数据交换的行为。 （这与早期的 xinetd 这个 super daemon 有部份 的相似喔！） |
| .target           | 执行环境类型(target unit)：其实是一群unit的集合，例如上面表格中提到的multi-user.target其实就是一堆服务的集合,选择执行multi-user.target就是执行一堆其他.service或.socket之类的服务 |
| .mount/.automount | 文件系统挂载相关服务(automount unit/mount unit)：例如来自网络的自动挂载、NFS文件系统挂载等与文件系统相关性较高的程序管理。 |
| .path             | 侦测特定文件或目录类型(path unit)：某些服务需要侦测某些特定的目录来提供队列服务，例如最常见的打印服务，就需要通过侦测打印队列目录来启动打印功能！这时候就得要.path的服务类型支持了！ |
| .timer            | 循环执行的服务(timer unit)：这个东西有点累anacrontab.不过是由systemd主动提供的，比anacrontab更有弹性！ |

## 通过systemctl管理单一服务(service unit)的启动/开机启动与观察状态

![image](https://user-images.githubusercontent.com/25349066/64608306-ee047680-d3fc-11e9-8fc3-260862126f1d.png)

使用systemctl status atd.service 查看atd这个服务的状态

Loaded: 这行在说明，开机时这个unit会不会启动，enabled为开机启动，disable开机不会启动

Active:现在这个unit的状态为正在执行(running)或没有执行(dead)

后面几行则是在说明这个unit程序的PID状态以及最后一行显示这个服务的的登录文件信息

文件信息格式为:"时间" "讯息发送主机" "哪一个服务的讯息" "实际讯息内容"

所以上面显示的讯息是： 这个atd服务已启动延迟执行计划的意思。

![image](https://user-images.githubusercontent.com/25349066/64608795-00cb7b00-d3fe-11e9-8b81-6e6a32b6abb0.png)

使用sytemctl stop atd.service关闭atd服务。不应该使用kill的方式来关掉一个正常的服务，否则systemctl会无法继续监控该服务。最后两行为新增加的登录讯息，告诉我们目前的系统状态。

第二行中为enabled 表示它在重新开机后仍会启动

Active几种基本状态：

+ active （running）：正有一只或多只程序正在系统中执行的意思，举例来说，正在执行 中的 vsftpd 就是这种模式。 
+ active （exited）：仅执行一次就正常结束的服务，目前并没有任何程序在系统中执行，举例来说，开机或者是挂载时才会进行一次的 quotaon 功能，就是这种模式！ quotaon 不须一直执行～只须执行一次之后，就交给文件系统去自行处理啰！通常用 bash shell 写的小型服务，大多是属于这种类型 （无须常驻内存）。
+ active （waiting）：正在执行当中，不过还再等待其他的事件才能继续处理。举例来 说，打印的伫列相关服务就是这种状态！ 虽然正在启动中，不过，也需要真的有伫列进 来 （打印工作） 这样他才会继续唤醒打印机服务来进行下一步打印的功能。
+ inactive：这个服务目前没有运行的意思。

daemon几种默认状态：

+ enabled：这个daemon将在开机时被执行
+ disabled：这个daemon在开机时不会被执行
+ static：这个daemon不可以自己启动(enable不可)，不过可能被其他的enabled的服务唤醒(相互依赖)
+ mask：这个daemon无论如何都不能被启动，因为已经被强制注销，可通过systemctl unmask方式改回原本状态

![image](https://user-images.githubusercontent.com/25349066/64609988-c9120280-d400-11e9-9907-c48ca70d7d12.png)

使用systemctl list-units查看所有启动的unit

![image](https://user-images.githubusercontent.com/25349066/64611188-74bc5200-d403-11e9-8c60-a24e554171cf.png)

使用systemctl list-unit-files查看已安装的所有unit

## systemctl针对service类型的配置文件

### 配置文件目录相关

systemd的配置文件大部分放置于/usr/lib/systemd/system/目录内。但是该目录下的文件是原本软件所提供的位置，1建议不要修改。而要修改的位置应该放置于/etc/systemd/system/目录内。举例来说，如果你想要额外修改vsftpd.service的话，可以放置到:

+ /usr/lib/systemd/system/vsftpd.service ：官方的默认配置文件
+ /etc/systemd/system/vsftpd.service.d/custom.conf：在/etc/systemd/system下面创建与配置文件相同文件名的目录，但要加上.d扩展名。然后在该目录下创建配置文件即可。另外，配置文件最好附文件名.conf最佳，在这个目录下的文件会“累加其他设置”进入/usr/lib/systemd/system/vsftpd.service内。
+ /etc/systemd/system/vsfrpd.service.wants/*：此目录内的文件为链接文件，设置相依服务的链接，意思是启动了vsftpd.service之后，最好再加上这个目录下建议的服务。
+ /etc/systemd/system/vsftpd.service.requires：此目录内的文件为链接文件，设置相依服务的链接。意思是启动了vsftpd.service之后，需要事先启动哪些服务的意思。

### systemctl配置文件设置项目

