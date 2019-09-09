# RPI-ZeroW-Buster-Note
2019 Buster：记录自已这一个月来折腾树莓派的辛酸经历

树莓派不断更新，网上很多教程试来试去发现配置已变了，浪费了不少时间，所以记录一下这个把多月折腾树莓派的辛酸经历，给和我一样的初学者共勉


【0】工具准备：

        * 树莓派+电源
        * USB转串口 
        * SD卡（最好4G class10以上）SD卡速度是硬伤

        * PanasonicSDFormatter
        * Win32DiskImager
        * 串口终端（我使用的是MACOS自带终端）,或(WIN系统)SCCOM+PUTTY


【1】安装系统

    下载镜像：
            https://www.raspberrypi.org/downloads/raspbian/
            网页下面的3个版本都试过，zero最好还是跑lite版本，其它版本也行，配置成命令行界面启动就好了，图形界面太卡了。
    
    *格式化SD卡：
            使用PanasonicSDFormatter,  特别是SD卡重新刷系统时这一步必须
    
    *烧写镜像
            使用* Win32DiskImage, 烧写镜像到格式化好的SD卡，也可以使用命令行，如果不熟悉还是罢了，万一写错盘可是大事。
            [我在使用电脑自带的SD读卡器烧写的镜像无法启动，换了一个读卡器才正常]
    
    *启用串口
            将SD卡的boot盘目录下的config.txt加上一行 : uart_enable=1 （启用串口）
            如果有接HDMI显示器那么同时将#hdmi_force_hotplug=1的#号去掉
    
    *安全弹出SD卡，SD卡放入到树莓派并接入电源，可以看到绿灯不停闪烁表示启动中，电流才80ma。
            
    
    *串口连接树莓派
            串口连接树莓派（图中GPIO14,GPIO15 ）, 并启动终端（例如我使用的是MACOS）：
             ls /dev/cu* 查看电脑串口， 比如我的使有的CP2102查看到串口设备是/dev/cu.SLAB_USBtoUART
            终端输入 screen /dev/cu.SLAB_USBtoUART 115200 启动一个串口终端，回车后会提示输入用户名和密码登录树莓派
            用户名：pi, 
            密码：raspberry
    PS: WIN系统使用类似的串口工具（比如SSCOM），在设备管理器查看对应的COM口后，使用串口工具连接


![Alt text](/doc/raspi_pins.jpg)

【2】系统配置

        *系统配置： 使用命令 sudo raspi-config 启动配置工具

                -》2 Network Options -》N2 Wi-Fi -》CN China (选择国家), 然后跟据提示输入WIFI的SSID和密码
                -》 4 Localisation Options -》 I1 Change Locale -》按空格选中en_US和zh_CN后按回车确认
                -》5 Interfacing Options  -》P2 SSH（激活SSH）P4 SPI（激活SPI）
                -》7 Advanced Options -》A1 Expand Filesystem （扩展更的SD卡空间，因为在镜像写入时，分区大小被限制了）

        *校验一下结果：
                ifconfig  如果WIFI正常，这里的wlan0可以看到连接状态和IP, 复制这个IP用于SSH连接
                sudo nano /etc/wpa_supplicant/wpa_supplicant.conf  将会打开配置文件，可以看到刚才输入的WIFI信息 (CTRL+X退出)

                解决locale问题 : locale: Cannot set LC_CTYPE to default locale: No such file or directory
                sudo nano /etc/default/locale //增加 LC_CTYPE=xx 比如LC_CTYPE=en_US.UTF-8
                locale -a         //查看当前系统系统支持的字符编码方式
                locale                //查看locale

        *SSH，
                MACOS：在终端使用命令：ssh pi@x.x.x.x   （ssh 树莓派用户名@树莓派IP）
                                        第一次连接会有一个提示，输入yes就可以了，然后输入密码就可以登陆了。
                WIN：则可以使用putty，填入用户名和IP连接就可以了

        *设置静态IP
                如果每次都要查看IP才到通过SSH连接，那真是麻烦死了，这时可以设置静态IP
                sudo nano /etc/dhcpcd.conf 将会打开配置文件，在最后面加入（注意大小写）：

        interface wlan0 
        
        ssid OOOOO
        inform 192.168.0.9/24
        static ip_address=192.168.0.9/24
        static routers=192.168.0.1
        static domain_name_servers=192.168.0.1


                这里的ssid, ip, router, domain都要换成你指定的，否则无法上网和连接了
                关于IP的后缀，如果掩码是255.255.255.0那么就填/24, 如果掩码是255.255.255.240那么就填/28 

                这个时候可以复位一下树莓派了 sudo reboot， 可以留意串口的启动信息。

                重启动，使用新的自定义IP连接SSH
                        ssh pi@x.x.x.x


        * 这个时候如果出现一个警告 @    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @，可以使用命令清除掉缓存后再次连接SSH。
                ssh-keygen -R x.x.x.x  (记住这里的xxxx是当前要连接的IP)

        哇哈哈哈，您可以扔掉USB转串口工具了。

【3】更换软件源并更新系统

        编辑源文件 `sudo nano /etc/apt/sources.list` 文件，删除原文件所有内容，用以下内容取代：
                deb http://mirrors.tuna.tsinghua.edu.cn/raspbian/raspbian/ buster main non-free contrib
                deb-src http://mirrors.tuna.tsinghua.edu.cn/raspbian/raspbian/ buster main non-free contrib

        编辑源文件 `sudo nano /etc/apt/sources.list.d/raspi.list` 文件，删除原文件所有内容，用以下内容取代：
                deb http://mirrors.tuna.tsinghua.edu.cn/raspberrypi/ buster main ui

        编辑镜像站后，请使用sudo apt-get update命令，更新软件源列表，同时检查您的编辑是否正确, 显示信息:
                Get:1 http://mirrors.tuna.tsinghua.edu.cn/raspbian/raspbian buster InRelease [15.0 kB]
                ......
                49% [5 Sources 10.4 MB/11.3 MB 92%] 

        然后使用sudo apt-get dist-upgrade命令，更新所有包。
                dist-upgrade如果这个包没有发布更新，就不管它；如果发布了更新，就把包下载到电脑上，并安装。
                由于包与包之间存在各种依赖关系。upgrade只是简单的更新包，不管这些依赖，它不和添加包，或是删除包。
                而dist-upgrade可以根据依赖关系的变化，添加包，删除包


【4】 驱动SPI
![Alt text](/doc/st7735s_pins.jpg)

        脚位图  

        驱动库   https://github.com/notro/fbtft/issues
        我使用的屏是SPI TFT 这里是资料   ST7735S_V1.1_20111121.pdf (2.49 MB, 下载次数: 1) 
        最新的树莓派已经内置了SPI TFT的库fbtft，我们只需要加载就可以了，直接sudo modprobe fbtft_device name=sainsmart18 

        对于我使用的SST7735S还需要调整一下参数才能正确显示。需要修改一下配置参数

        【A.】 创建flexfb 驱动， 新建一个配置文件 ‘sudo nano /etc/modprobe.d/fbtft.conf’， 然后加入下面内容，CTRL+S保存
        
            options fbtft_device name=flexfb gpios=reset:25,dc:24,led:18 speed=12000000 bgr=0 fps=100 custom=1 height=128 width=192
            options flexfb setaddrwin=0 width=192 height=128 init=-1,0x01,-2,150,-1,0x11,-2,500,-1,0xB1,0x01,0x2C,0x2D,-1,0xB2,0x01,0x2C,0x2D,-1,0xB3,0x01,0x2C,0x2D,0x01,0x2C,0x2D,-1,0xB4,0x07,-1,0xC0,0xA2,0x02,0x84,-1,0xC1,0xC5,-1,0xC2,0x0A,0x00,-1,0xC3,0x8A,0x2A,-1,0xC4,0x8A,0xEE,-1,0xC5,0x0E,-1,0x20,-1,0x36,0xC8,-1,0x3A,0x05,-1,0xE0,0x0f,0x1a,0x0f,0x18,0x2f,0x28,0x20,0x22,0x1f,0x1b,0x23,0x37,0x00,0x07,0x02,0x10,-1,0xE1,0x0f,0x1b,0x0f,0x17,0x33,0x2c,0x29,0x2e,0x30,0x30,0x39,0x3f,0x00,0x07,0x03,0x10,-1,0x29,-2,100,-1,0x13,-2,10,-3


        重点有三个地方：
        speed,  控制spi输出的时钟速度，要适配你的屏，比如我接SPI TFT时设的是48M，接LED屏时只能设12M
        fps,    刷新率，实际测试并不完全准确，实际上刷新率会受屏的分辩率及speed的影响。如果你关心FPS，最好实测一下
        height width  屏幕大小, 比如我的SPI屏7735设成128 128就会有黑边，只能设成132 128，而LED屏我需要设成192X128来匹配
         
        
        【B.】 新建驱动加载文件 ： ‘sudo nano /etc/modules-load.d/fbtft.conf’ ，并加入下面内容 让系统启动时加载7735S的驱动
                spi-bcm2835
                flexfb
                fbtft_device
        
        【C.】安装fbcp驱动 重启一下树莓派，用心观察会发现，先是雪花屏，然后变成全黑，说明驱动起作用了。那么我们可以把SPI设为主屏，或者将主屏镜像到SPI，我选择的是后者。

                sudo apt-get install cmake git
                cd ~
                git clone https://github.com/tasanakorn/rpi-fbcp 
                cd rpi-fbcp/
                mkdir build
                cd build/
                cmake ..
                make
                sudo install fbcp /usr/local/bin/fbcp

        安装完成后执行‘fbcp &’（注意&号一定要加，这样程序会在后台运行），屏幕上就可以看到树莓派字符界面了

        如果你想设置开机启动fbcp，可以使用命令行打开配置文件：
                sudo nano /etc/rc.local
        然后在 exit 0 前面添加 fbcp &。注意一定要添加"&" 。

        【D.】 修改主屏分辨率，是不是发现字符都被挤压了一样，修改一下主屏分辨率就可以了，使用命令行打开配置文件：’sudo nano /boot/config.txt‘
                修改这二个参数：
                framebuffer_width=128
                framebuffer_height=128

                或者我显示LED屏时
                framebuffer_width=192
                framebuffer_height=128

                然后重启树莓派：

 

        LED屏和SPI屏同用一个驱动，只是稍改一下上面所描述的参数，关于LED屏，可以看这里了解更多https://www.amobbs.com/thread-5716651-1-1.html


   ![Alt text](/doc/login.jpeg)


【5】安装FileZilla电脑与树莓派交换文件， ，配置参数：

        主机：填树莓派IP
        用户名：树莓派用户名
        密码：树莓派密码
        端口：22

        点击连接，就可以增删修改树莓派上文件了。

【6】安装网络共享文件 Samba 服务， 可以实现系统之间的文件与打印机的共享

        * 安装samba： 
                apt-get install samba

        * 创建专门的用户和目录
                sudo useradd homeuser
                sudo passwd homeuser
                sudo mkdir /var/share
                sudo chown -R homeuser /var/share

        * 配置共享        
                sudo nano /etc/samba/smb.conf

                在结尾加入
                [share]
                   comment = Share    
                   path = /var/share          # 共享文件的路径
                   read only = no
                   browseable = no        # 可被其他人看到资源名称（非内容）
                   create mask = 0777         # 新建文件的权限
                   directory mask = 0777        # 新建目录的权限

        * 设置samba远程访问密码
                sudo smbpasswd -a homeuser                //远程访问密码

        * 重启树莓派并测试samba。
                WIN 在运行中输入IP \\xx.xx.xx.xx
                MACOS 访达-》前往-》连接服务器-》填写 smb://xx.xx.xx.xx/share

                输入用户名和密码（如上面则是homeuser, 远程访问密码), 这样就可以方问Samba服务器中的文件了。

【7】搭建DLNA流媒体服务器

        *安装
                sudo apt-get install minidlna

        *可以查看minidlna服务器状态是否正常运行
                sudo systemctl status minidlna
        
        *大多时候我们会修改成自定义的目录，比如我是这样修改的
                ：直接在smb目录下创建DLNA目录，并增加homeuser用户
                        cd /var/share
                        sudo mkdir DLNA
                        cd DLNA
                        sudo mkdir Music Video Picture
                        cd /var
                        sudo chown -R homeuser ./share  //homeuser是之前创建的samba用户

                ：打开配置文件自定义目录
                        sudo nano /etc/minidlna.conf

                ：=A表示这个目录是存放音乐的，当minidlna读到配置文件时，它会自动加载这个目录下的音乐文件
                        media_dir=A,/var/share/DLNA/Music
                        media_dir=P,/var/share/DLNA/Picture
                        media_dir=V,/var/share/DLNA/Video

        *重新启动minidlna服务， 让配置生效，并查看运行状态
                sudo systemctl restart minidlna
                sudo systemctl status minidlna

        *在电脑上查看流媒体资源个数
                http://树莓派的IP地址:8200/

        打开电视和电脑或手机，可以愉快地娱乐一下了
        
        
【9】安装播放器

    [*] 安装
            sudo apt install mplayer
            sudo apt install omxplayer
            sudo apt install fbi
    
    [*] mplayer播放音视频
            sudo mplayer -nolirc -vo fbdev2:/dev/fb0 v.mp4
            sudo mplayer -fs -vf scale=192:128 -nolirc -vo fbdev2:/dev/fb0 '001 冰淇淋_高清.mp4'
            ****参数fb0是输出到主屏，fb1是车出到LCD，如果启用了fbcp，那么应该输出到fb0
            ****如果文件名有空格，要用''号括起来
    
    [*] omxplayer播放音视频 [硬件h264解码]
            sudo omxplayer --vol -2000 -o alsa '001 冰淇淋_高清.mp4'
    
    [*] fbi 显示图片
            sudo fbi -d /dev/fb0 -T 1 -noverbose -a test.jpg
    
    
    使用FileZilla或Samba上传些 '善良的' 文件，让我娱乐一下再继续.......

【10】输出PWM音频

    sudo nano /boot/config.txt，找到dtparam=audio=on，增加下面配置信息
    
            dtparam=audio=on
            dtoverlay=pwm-2chan,pin=18,func=2,pin2=13,func2=4        #13,18输出双声道
            disable_audio_dither=1                #据说可以消除一点杂音，在ZERO上没鬼用
            audio_pwm_mode=2                #据说声音会好一点 
    
    sudo reboot 重启树莓派生效

    【*】音量调节工具alsamixer
    【*】又或者可以安装蓝牙耳机，PWM音频太差了
            //依耐: bluez pulseaudio
            sudo apt-get install pulseaudio-module-bluetooth
            然后使用bluetoothctl打开蓝牙控制界面配置蓝牙

【11】安装AirPlay服务: shairport 

        //安装依赖工具包
        sudo apt install libssl-dev libavahi-client-dev libasound2-dev libao-dev libpulse-dev
        //安装shairport
        sudo apt install shairport-sync

        习惯性sudo reboot 重启系统让服务生效。

        重启后在IPHONE声音控制界面会出现AirPlay按钮，或MAC声音控制图标那里会出现一个小音箱图标Raspberrypi， 说明AirPlay已开始正常工作....
        AirPlay默认音乐比较小，留意使用alsamixer调大音量。

【12】搭建个人网站

        这个比较简单：
                sudo apt install apache2 
                apt install php

        服务状态查询，或控制
                 service apache2 status[]

        查看临听的端口
                netstat -ln|grep 80

        使用电脑浏览器输入: 树莓派IP (比如我的 162.20.10.9), 就会打开一个Apache2 Debian Default Page
        这个文件在树莓派的地址：/var/www/html/index.html

【13】安装花生层内网穿透服务

        安装：
                sudo dpkg -i phddns_rapi_3.0.1.armhf.deb
        
        如果想删除它：
                sudo dpkg -r phddns
        
        完成安装后会出现设备ID
        
        +--------------------------------------------------+
        |             Oray PeanutHull Linux 3.0            |
        +--------------------------------------------------+
        |  SN: RAPI000000000000   Default password: admin  |
        +--------------------------------------------------+
        |    Remote Management Address http://b.oray.com   |
        +--------------------------------------------------+
        Processing triggers for systemd (241-5+rpi1) ...
        复制代码


        在浏览器上打开http://b.oray.com，输出SN和PASSWORD，会出现绑定二维号
        手机下载一个花生壳并注册一个免费帐号，然后扫描这个二维码绑定设备，花生壳会自动生成一个域名
        然后在内网穿透配置页面输入你的树莓派IP和其它信号完成穿透绑定即可


        最后手机断开WIFI，使用4G上网，输入域名测试是否成功穿透。这时如果正常会打开树莓派的 /var/www/html/index.html页面

        然后，，JS+PHP+PYTHON+树莓派，可以做很多很多事情，，，，，，，


