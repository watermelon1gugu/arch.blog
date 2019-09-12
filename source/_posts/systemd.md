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

例如sshd.service 

```
[Unit] # 这个项目与此unit的解释、执行服相依性有关
Description=OpenBSD Secure Shell server
After=network.target auditd.service # 在此之后启动
ConditionPathExists=!/etc/ssh/sshd_not_to_be_run

[Service] # 这个项目与实际执行的指令参数有关
EnvironmentFile=-/etc/default/ssh
ExecStartPre=/usr/sbin/sshd -t
ExecStart=/usr/sbin/sshd -D $SSHD_OPTS
ExecReload=/usr/sbin/sshd -t
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
Restart=on-failure
RestartPreventExitStatus=255
Type=notify
RuntimeDirectory=sshd
RuntimeDirectoryMode=0755

[Install] # 这个项目说明此unit要挂载哪个target下面
WantedBy=multi-user.target
Alias=sshd.service
```



+ [Unit] : unit本身的说明，以及与其他相依daemon的设置，包括在什么服务之后才启动此unit之类的设置值。
+ [Service], [Socket], [Timer], [Mount], [Path]..：不同的 unit type 就得要使用相对应的设置项目。比如sshd.service就是service。这个项目主要在规范服务启动的脚本、环境配置文件文件名、重新启动的方式等等。
+ [Install] : 这个项目就是将此unit安装到哪个target里面去的意思.

至于配置文件内有些设置规则还是得要说明一下： 

设置项目通常是可以重复的，例如我可以重复设置两个 After 在配置文件中，不过，后面 的设置会取代前面的喔！因此，如果你想要将设置值归零， 可以使用类似“ After= ”的设 置，亦即该项目的等号后面什么都没有，就将该设置归零了 （reset）。 如果设置参数需要有“是/否”的项目 （布林值, boolean），你可以使用 1, yes, true, on 代 表启动，用 0, no, false, off 代表关闭！随你喜好选择啰！ 空白行、开头为 # 或 ; 的那一行，都代表注解！

**Unit部分：**

| RestartSec    | 与 Restart 有点相关性，如果这个服务被关闭，然后需要重新启动 时，大概要 sleep 多少时间再重新启动的意思。默认是 100ms （毫 秒）。 |
| ------------- | ------------------------------------------------------------ |
| Description   | 就是当我们使用 systemctl list-units 时，会输出给管理员看的简易说明！当然，使用 systemctl status 输出的此服务的说明，也是这个项目！ |
| Documentation | 这个项目在提供管理员能够进行进一步的文件查询的功能！提供的文 件可以是如下的数据： Documentation=http://www.... Documentation=man:sshd（8） Documentation=file:/etc/ssh/sshd_config |
| After         | 说明此unit是在哪个daemon启动之后才启动的意思。基本上仅是说 明服务启动的顺序而已，并没有强制要求里头的服务一定要启动后此 unit 才能启动。 以 sshd.service 的内容为例，该文件提到 After 后面 有 network.target 以及 sshd-keygen.service，但是若这两个 unit 没有 启动而强制启动 sshd.service 的话， 那么 sshd.service 应该还是能够 启动的！这与 Requires 的设置是有差异的喔！ |
| Before        | 与 After 的意义相反，是在什么服务启动前最好启动这个服务的意思。 不过这仅是规范服务启动的顺序，**并非强制要求**的意思。 |
| Requires      | 明确的定义此 unit 需要在哪个 daemon 启动后才能够启动！就是设置 相依服务啦！如果在此项设置的前导服务没有启动，那么此 unit 就不会被启动！ |
| Wants         | 与 Requires 刚好相反，规范的是这个 unit 之后最好还要启动什么服务 比较好的意思！不过，并没有明确的规范就是了！主要的目的是希望 创建让使用者比较好操作的环境。 因此，这个 Wants 后面接的服务如 果没有启动，其实不会影响到这个 unit 本身！ |
| Conflicts     | 代表冲突的服务！亦即这个项目后面接的服务如果有启动，那么我们 这个 unit 本身就不能启动！我们 unit 有启动，则此项目后的服务就不能启动！ 反正就是冲突性的检查啦！ |
|               |                                                              |
|               |                                                              |

**Service部分:**

| 设置参数        | 参数意义说明                                                 |
| --------------- | ------------------------------------------------------------ |
| Type            | 说明这个 daemon 启动的方式，会影响到 ExecStart 喔！一般来说， 有下面几种类型 simple：默认值，这个 daemon 主要由 ExecStart 接 的指令串来启动，启动后常驻于内存中。forking：由 ExecStart 启动 的程序通过 spawns 延伸出其他子程序来作为此 daemon 的主要服 务。原生的父程序在启动结束后就会终止运行。 传统的 unit 服务大 多属于这种项目，例如 httpd 这个 WWW 服务，当 httpd 的程序因为 运行过久因此即将终结了，则 systemd 会再重新生出另一个子程序持 续运行后， 再将父程序删除。据说这样的性能比较好！！oneshot： 与 simple 类似，不过这个程序在工作完毕后就结束了，不会常驻在 内存中。dbus：与 simple 类似，但这个 daemon 必须要在取得一个 D-Bus 的名称后，才会继续运行！因此设置这个项目时，通常也要设 置 BusName= 才行！idle：与 simple 类似，意思是，要执行这个 daemon 必须要所有的工作都顺利执行完毕后才会执行。这类的 daemon 通常是开机到最后才执行即可的服务！比较重要的项目大概是 simple, forking 与 oneshot 了！毕竟很多服务需要子程序 （forking），而有更多的动作只需要在开机的时候执行一次 （oneshot），例如文件系统的检查与挂载啊等等的。 |
| EnvironmentFile | 可以指定启动脚本的环境配置文件！例如 sshd.service 的配置文件写 入到 /etc/sysconfig/sshd 当中！你也可以使用 Environment= 后面接 多个不同的 Shell 变量来给予设置！ |
| ExecStart       | 就是**实际执行此 daemon 的指令或脚本程序**。你也可以使用 ExecStartPre （之前） 以及 ExecStartPost （之后） 两个设置项目 来在实际启动服务前，进行额外的指令行为。 但是你得要特别注意的 是，指令串仅接受“指令 参数 参数...”的格式，不能接受 <, >, >>,\|, & 等特殊字符，很多的 bash 语法也不支持喔！ 所以，要使用这些特殊 的字符时，最好直接写入到指令脚本里面去！不过，上述的语法也不 是完全不能用，亦即，若要支持比较完整的 bash 语法，那你得要使 用 Type=oneshot 才行喔！ 其他的 Type 才不能支持这些字符。 |
| ExecStop        | 与 systemctl stop 的执行有关，关闭此服务时所进行的指令。     |
| ExecReload      | 与 systemctl reload 有关的指令行为                           |
| Restart         | 当设置 Restart=1 时，则当此 daemon 服务终止后，会再次的启动此 服务。举例来说，如果你在 tty2 使用文字界面登陆，操作完毕后登 出，基本上，这个时候 tty2 就已经结束服务了。 但是你会看到屏幕 又立刻产生一个新的 tty2 的登陆画面等待你的登陆！那就是 Restart 的功能！除非使用 systemctl 强制将此服务关闭，否则这个服务会源 源不绝的一直重复产生！ |
| RemainAfterExit | 当设置为 RemainAfterExit=1 时，则当这个 daemon 所属的所有程序 都终止之后，此服务会再尝试启动。这对于 Type=oneshot 的服务很 有帮助！ |
| TimeoutSec      | 若这个服务在启动或者是关闭时，因为某些缘故导致无法顺利“正常 启动或正常结束”的情况下，则我们要等多久才进入“强制结束”的状 态！ |
| KillMode        | 可以是 process, control-group, none 的其中一种，如果是 process 则 daemon 终止时，只会终止主要的程序 （ExecStart 接的后面那串 指令），如果是 control-group 时， 则由此 daemon 所产生的其他 control-group 的程序，也都会被关闭。如果是 none 的话，则没有程 序会被关闭喔！ |
**Install部分：**

| 设置参数 |                                                              |
| -------- | ------------------------------------------------------------ |
| WantedBy | 这个设置后面接的大部分是 *.target unit ！意思是，这个 unit 本身是附挂在 哪一个 target unit 下面的！一般来说，大多的服务性质的 unit 都是附挂在 multi-user.target 下面！ |
| Also     | 当目前这个 unit 本身被 enable 时，Also 后面接的 unit 也请 enable 的意 思！也就是具有相依性的服务可以写在这里呢！ |
| Alias    | 进行一个链接的别名的意思！当 systemctl enable 相关的服务时，则此服务会进行链接文件的创建！以 multi-user.target 为例，这个家伙是用来作为 默认操作环境 default.target 的规划， 因此当你设置用成 default.target 时，这个 /etc/systemd/system/default.target 就会链接到 /usr/lib/systemd/system/multi-user.target 啰！ |



## 编写自己的服务

假设编写一个备份tmpfs的服务，脚本放在/backup下

```bash
#!/bin/bash
cp -r /dev/shm/* ~/tmpbackup/
```

接下来设计名为backup.service的启动脚本设置,路径为/etc/systemd/system/backup.service

```
[Unit]
Description=备份tmpfs文件夹

[Service]
Type=simple
ExecStart=/bin/bash /backup/backup.sh

[Install]
WantedBy=multi-user.target         
```

![image](https://user-images.githubusercontent.com/25349066/64802745-ec35e100-d5bd-11e9-91f8-497610dc4775.png)

之后便编写完成