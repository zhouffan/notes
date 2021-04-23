

### **1、常用示例**

- dumpsys window | grep Focus

- traceroute 110.167.132.1（路由）

- cat /sys/class/video/axis ：视频播放窗口 （0 1 1918 1077）

- adb shell screenrecord --size 1920x480 /storage/sdcard0/demorecord.mp4   屏幕录制  （Ctrl + C停止）

  adb shell screenrecord --time-limit 100 /storage/sdcard0/demorecord.mp4   设置时长

- input keyevent 82   （input text 192.168.2.183:8081）

- adb remount   （echo 1 >/sys/class/remount/need_remount;mount -o remount /system）   \# remount '/system'分区 as read-write

  mount -o remount /system

- **cat /proc/meminfo** （查看设备内存）

- du -k  或者 -m   （文件占用空间的大小）

- adb logcat -c && adb logcat -v time >D:\iii.log （用命令抓log）

- dumpsys meminfo com.android.smart.terminal.ctsh.launcher  （查看内存）

 

### **2、linux常用命令**

```java
cd    ——>改变当前目录 
pwd   ——>查看当前所在目录完整路径
ls    ——>查看目录或者文件的属*，列举出任一目录下面的文件 
mkdir ——>建立目录 
cp    ——>拷贝文件 
rm    ——>删除文件和目录 
mv    ——>移走目录或者改文件名
chmod/chown ——>权限修改
clear ——>清屏 
mount ——>加载一个硬件设备 
su    ——>在不退出登陆的情况下，切换到另外一个人的身份
grep  ——>文本内容搜索 
find  ——>文件或者目录名以及权限属主等匹配搜索 
kill  ——>可以杀死某个正在进行或者已经是dest状态的进程 
df    ——>命令用来检查文件系统的磁盘空间占用情况
```



### 3、命令查询表




| 指令                                                         | 作用                                                  | 备注                                                         |
| ------------------------------------------------------------ | ----------------------------------------------------- | :----------------------------------------------------------- |
| adb devices                                                  | 查看已连接的设备列表                                  |                                                              |
| adb start-server                                             | 开启ADB服务                                           |                                                              |
| adb kill-server                                              | 关闭ADB服务                                           |                                                              |
| adb connect [IP]                                             | 连接设备                                              | [IP]为连接设备的ip地址。如果无法连接，尝试先执行adb tcpip 5555。 |
| adb disconnect [IP]                                          | 断开设备                                              | [IP]为断开设备的ip地址。                                     |
| adb install -r [apk的路径]                                   | 安装apk                                               | -r 代表如果apk已安装，重新安装apk并保留数据和缓存文件。apk路径则可以直接将apk文件拖进cmd窗口，记得加空格。 |
| adb uninstall [apk包名]                                      | 卸载apk                                               |                                                              |
| adb uninstall -k [apk包名]                                   | 卸载 app 但保留数据和缓存文件                         |                                                              |
| adb remount                                                  | 获取文件的读写权限                                    | 有些设备并不能直接adb remount，必须要先以root身份进入，先执行adb root，在执行adb remount |
| adb push [文件路径] [想要push的路径]                         | 将PC上的文件推到设备指定路径下。【PC --> 移动设备】   | [文件路径] 为PC上文件的路径，[push的路径]为设备上的路径，常用的地址：/sdcard/、/sdcard/Download【需要获取文件的读写权限】 |
| adb pull [文件路径] [想要pull的路径]                         | 将设备中指定路径的文件拉取到PC上。【移动设备 --> PC】 | [文件路径] 为设备上文件的路径，[push的路径]为保存到PC上的文件路径。【需要获取文件的读写权限】 |
| adb logcat                                                   | 查看logcat日志                                        |                                                              |
| adb shell dumpsys window displays                            | 查看屏幕详细信息                                      |                                                              |
| adb shell wm size                                            | 查看屏幕的分辨率                                      |                                                              |
| adb shell wm density                                         | 查看屏幕密度                                          |                                                              |
| adb shell screencap -p [截图文件保存的路径]                  | 截屏                                                  | [截图文件保存的路径]：/sdcard/Pictures/Screenshots/1.png     |
| adb shell screenrecord [视频文件保存的路径]                  | 录屏                                                  | 需要停止时按 Ctrl-C，默认录制时间和最长录制时间都是 180 秒。 |
| adb shell am force-stop [apk包名]                            | 强制关闭程序                                          | [apk包名]：应用程序的包名，输入命令后就能直接杀死程序。      |
| adb shell killall [包名]                                     | 杀死包下的所有进程                                    | 需要su权限                                                   |
| adb shell input keyevent [事件key]                           | 模拟按键输入                                          | 4:返回键， 82:菜单, 187:最近运行列表                         |
| adb shell am start -n [包名]/[包名].[activity类名]           | 启动activity                                          |                                                              |
| adb shell am startservice -n [包名]/[包名].[service类名]     | 启动service                                           |                                                              |
| adb shell am broadcast -a [广播动作]                         | 发送广播                                              |                                                              |
| adb shell settings get secure android_id                     | 获取Android ID                                        |                                                              |
| adb shell su ifconfig eth0 192.169.5.20 netmask 255.255.255.0 up route add default gw 192.168.5.1 | 设置以太网的静态IP地址                                | 设置静态IP地址为192.168.5.20，子网掩码为255.255.255.0， 默认网关是192.168.5.1【前提是需要root权限】 |
| adb shell ifconfig                                           | 查看设备的ip地址信息                                  |                                                              |
| adb shell netcfg                                             | 查看设备的网络端口                                    |                                                              |
| iptables -t nat -A PREROUTING -p tcp --dport 21 -j REDIRECT --to-port 2121 | 进行端口映射                                          | 将21端口映射到2121端口上                                     |

