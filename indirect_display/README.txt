
这是支持 WIN10 1607 之后的 Indirect Display 虚拟显示器驱动。

【1】安装办法：
打开 设备管理器，操作-> 添加过时硬件，一路“下一步”，直到选择从磁盘安装，
进入对应的驱动目录下，选择 indirect_device.inf，
会出现 “Fanxiushu Indirect Display Bus System Device”，安装。
安装之后，会在设备管理器出现一个带感叹号的“Fanxiushu-Indirect-Device1”，
右击，“更新驱动程序”，选择 同样目录下的indirect_display.inf  安装。
安装成功后会在 显示适配器出现 ”Fanxiushu Indirect Display Driver “

如果之前安装了此驱动，更新办法：
进入设备管理器，在显示适配器中，右击“Fanxiushu Indirect Display Driver”，选择卸载驱动，
弹出对话框中，选中“删除此设备的驱动程序软件”，然后卸载。
卸载之后，“操作”菜单栏选择“扫描检测硬解改动”；
或者找到“系统设备”里边的“Fanxiushu Indirect Display Bus System Device”，先禁用然后再启用；
之后会出现带感叹号的“Fanxiushu-Indirect-Device1”
然后按照上面方式安装更新版本的驱动。

【2】驱动签名情况：
驱动没做任何签名，如果想要安装成功，需要你自己签名，
或者 按住 Shift + 重启， 进入”配置启动“，选择禁用驱动强制签名。

【3】indirect_user目录：
这是演示如何截取本驱动内部生成的图像数据，然后显示到窗口的代码工程。
要求驱动开启 capture_image 功能，具体说明请查看配置文件中的说明:

【4】使用办法：
本次更新中，直接去除了使用应用层程序添加和删除虚拟显示器，
驱动直接读取配置文件的方式来动态添加和删除虚拟显示器。
这是本驱动采用内核总线驱动而不是应用层使用SwCreateDevice函数创建总线带来的好处：
完全不使用应用层程序操作IDD驱动，全部可通过简单的配置文件控制虚拟显示器。
并且支持在配置文件中添加多种分辨率和刷新率。
需要把 indirect_display.ini配置文件放置到Windows目录中，比如 C:\Windows ，
如果想放到另一个目录而不是windows目录，请查看配置文件中的说明。

【5】用途之一：
插入显示器之后，会在系统中出现一个新的虚拟显示器，可以设置成扩展桌面，设置分辨率等。
这时可以使用xdisp_virt或其他软件，这类支持多显示器功能的远程控制程序来查看操控虚拟显示器桌面。
一定程度上可以代替duetdisplay，DisplayLink，iDIsplay等这类专门实现显示器扩展的软件。

【6】其他可能的用途：
当你的电脑的显卡不连接显示器
（一般都不会有此嗜好，除非是特例，比如作为云虚拟机，或者作为后台服务器，通过远程桌面类软件来显示此类桌面）
不接显示器的显卡基本都处于节能状态，所以这个时候分辨率可能只支持极低的800X600或1027*768等，
即便有些显卡可能支持分辨率多些，但是可能截屏性能会比较差，
比如显卡处于接显示器的工作状态的时候，使用dxgi截屏可以达到很高的截屏效率，
而未接显示器的节能状态，DXGI截屏效率可能会非常低。（当然可以选择去外接一个显卡欺骗器。）
可以使用本驱动模拟出虚拟显示器，然后在系统的显示设置中，选择本虚拟显示器为显示输出。
这个时候，显卡也会处于正常的工作状态。

【7】本驱动同时自定义了刷新率，可以突破真实显示器无法达到的刷新率。
比如外接了一个只有60 FPS的刷新率的显示器，
当用远控软件的时候，即便显卡本身支持144FPS以上的刷新率，使用DXGI采集都无法突破 60 FPS，
当使用本驱动创建一个虚拟显示器，且设置 144FPS 的刷新率，
再使用远控软件采集这个虚拟显示器的时候，DXGI采集就能达到 144FPS 的采集率。
（也许别的电脑会有不同，但在我电脑中就是这种情况:
笔记本显示器只支持60FPS，DXGI采集无法超过60FPS，当使用Idd虚拟显示器的时候,能达到更高刷新率）


【8】indirect_display.ini配置文件内容如下：
--------------------------------------------------------------------------------------------------
;;;By fanxiushu 2022-05-17, 改进版本的 IDD驱动

;;;从 monitor1到monitor15，一共可生成15个虚拟显示器，
;;;最后一个 monitor_primary 是第16个显示器，用于特殊用途：
;;;;    当电脑中没有插入显示器时候，monitor_primary自动生成，当电脑中有其他显示器时候，monitor_primary自动删除。
;;;;    用于解决电脑没插入显示器时候，自动模拟出新显示器用于云桌面或远程桌面. 

;;;;每个 monitorXXX 里边的enable字段如果是1 表示允许此虚拟显示器，为0或者不存在，则表示禁止。
;;;
;;;; capture_image如果是1表示在驱动内部会生成截屏数据到共享内存，
;;;; 一般情况下不需要设置此字段,因为大部分情况下都可以通过DXGI等截屏方式获取虚拟显示器图像
;;
;;;;  res0 到 res49, 一共 50项，表示可以罗列出多个显示器分辨率，格式是;
;;;;      显示器宽带 X（或*）显示器高度,  显示器刷新率（比如60，144等）
;;
;;;   hide_cursor 字段表示是否让 IDD画鼠标，设置为1或者此字段不存在，都表示不画鼠标; 设置0 表示让IDD画鼠标
;;;   hide_cursor=1，会让DXGI截图，或者从本驱动直接获取的图像中没有鼠标，
;;;   hide_cursor=0,  会让DXGI截图，或者从本驱动直接获取的图像中显示鼠标图形。
;;;   从 hide_cursor=1 转到0，需要重启电脑才能生效（或者不必这么麻烦，直接设备管理器中先禁用“Fanxiushu Indirect Display Driver”，然后再启用）
;;;   调用IDD的函数 IddCxMonitorSetupHardwareCursor，并且设置 IDDCX_XOR_CURSOR_SUPPORT_NONE参数，即可以实现隐藏鼠标功能。

;;;;此配置文件默认放到windows目录下，比如 C:\Windows\indirect_display.ini, 固定indirect_display.ini文件名。
;;;;如果需要另外替换到别的位置，请打开注册表，找到如下位置：
;;;;HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\WUDF\Services\indirect_display\Parameters
;;;;里边添加 ConfigPath 的字符串值，然后填写完整的 配置文件路径.
;;;;驱动会每隔一秒定时检测配置文件的变化，自动根据配置的变化动态生成或删除虚拟显示器。

;;;以下是配置实例，可根据自己的需要，自行修改。

[monitor1]
enable= 1
capture_image=1
hide_cursor=1

res0=1920*1080, 144
res1=2560*1600,80

[monitor2]
enable=0
capture_image=0 
hide_cursor=0

res0=1024X768,60
res1=1600X900,80

[monitor_primary]
enable=1
capture_image=0 
hide_cursor=1

res0=1920*1080, 144
res1=2560*1600,100
-------------------------------------------------------------------------------------------------

【9】系统支持情况：
目前测试的系统包括：WIN10系统的 1703，1709, 1803, 1809, 1903 。测试的是64位系统，都没有问题。
其中只有 1903 在真实电脑上测试， 其他版本均是在vmware环境中测试。
至于 1607 版本的WIN10，在vmware环境中，测试都是失败的，真机没测试过。
失败现象：调用 IddCxAdapterInitAsync 函数会出现 
Invalid parameter passed to C runtime function 错误，
并且 IddCxAdapterInitAsync 返回 0xC000009A （STATUS_INSUFFICIENT_RESOURCES）错误。
原因暂时不明。

【10】WIN10 1607之前系统如果想要模拟虚拟显示器：
打开控制面板->显示->屏幕分辨率，“更改显示器外观”，选择“检测”，就会出现“未检测到其他显示器”，选中之后，
下面“多显示器”一栏，会出现“依然尝试在以下对象上进行连接：VGA”，选择它，就这样就建立了一个新的虚拟显示器，
当然这个新的显示器是不支持3D加速的。

详情：
https://blog.csdn.net/fanxiushu/article/details/93524220

fanxiushu 2022-05-17


