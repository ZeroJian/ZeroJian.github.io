###Vps Vultr 搭建安装 Shadowsocks 和 FinalSpeed


####Vultr 注册

<http://www.vultr.com/?ref=6910451>

这里是我的推荐链接,如果想不使用推荐链接,去掉后缀即可

####部署 VPS

![](http://7xo0hj.com1.z0.glb.clouddn.com/VPS%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-06-30%20%E4%B8%8B%E5%8D%8811.37.39.png)

服务器选择,推荐使用日本服务器

![](http://7xo0hj.com1.z0.glb.clouddn.com/VPS%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-06-30%20%E4%B8%8B%E5%8D%8811.38.05.png)

硬件配置,按需选择即可,普通使用 5 刀每月够用了

![](http://7xo0hj.com1.z0.glb.clouddn.com/VPS%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-06-30%20%E4%B8%8B%E5%8D%8811.37.55.png)

系统推荐 CentOS

![](http://7xo0hj.com1.z0.glb.clouddn.com/VPS%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-06-30%20%E4%B8%8B%E5%8D%8811.38.15.png) 

![](http://7xo0hj.com1.z0.glb.clouddn.com/VPS%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-06-30%20%E4%B8%8B%E5%8D%8811.38.43.png) 

按需配置,如果添加了 `SSH Keys` 登陆服务器不用每次输入密码,没有也不影响

`Server Label` 随便填一个名字即可

确定后等待安装好即可,如果状态显示 `Running` 点击 `Manage`

![](http://7xo0hj.com1.z0.glb.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-06-30%20%E4%B8%8B%E5%8D%8811.44.00.png)

记下服务器 ip 地址和密码 

打开终端或者 putty 或者其他工具 ssh 服务器

因为我的是 mac 电脑,终端直接可以 ssh

终端输入 

	ssh root@服务器地址

没有使用 SSH keys 需要输入密码,输入服务器上给的密码即可,连接后效果:

![](http://7xo0hj.com1.z0.glb.clouddn.com/VPS%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-06-30%20%E4%B8%8B%E5%8D%8811.48.01.png)

####安装 Shadowsocks

终端依次执行:

	wget –no-check-certificate https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks.sh

	chmod +x shadowsocks.sh

	./shadowsocks.sh 2>&1 | tee shadowsocks.log



根据提示输入 shadowsocks 密码和 端口号 (都可自定义)

![](http://7xo0hj.com1.z0.glb.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-06-30%20%E4%B8%8B%E5%8D%8810.27.52.png)

出现这一步表示安装成功

下载 Shadowsocks 电脑客户端配置服务器

![](http://7xo0hj.com1.z0.glb.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-06-30%20%E4%B8%8B%E5%8D%8811.53.02.png)

地址为 vps 地址

端口和密码为终端安装 Shadowsocks 的端口和密码

现在你可以正常浏览 google 了

####安装 FinalSpeed

FinalSpeed 可以加速网络连接,如果有经常观看 youtubu 的需求,推荐安装,1080p 无压力

如果需要加速效果,在 vps 端和电脑端都需要配置安装

#####VPS端安装

一键安装:

	rm -f install_fs.sh
	wget http://finalspeed.org/fs/install_fs.sh
	chmod +x install_fs.sh
	./install_fs.sh 2>&1 | tee install.log

debian,ubuntu下如果执行脚本出错,请切换到dash

切换方法: `sudo dpkg-reconfigure dash` 选no

安装完后查看日志

	tail -f /fs/server.log

设置服务端口:

默认udp 150 和 tcp 150 ,修改端口后服务端会自动修改防火墙.

linux版:

	 mkdir -p /fs/cnf/ ; 

echo 端口号

	 > /fs/cnf/listen_port ; sh /fs/restart.sh

windows 版: 在cnf目录下新建文件 listen_port,文件内容为端口号.

`一般情况下不需要更改服务器端口,默认即可`

设置开机启动:

	chmod +x /etc/rc.local 

	vi /etc/rc.local

打开 vi 后,默认为命令模式（command mode）,按下键盘 `i `进入插入模式（Insert mode)

新起一行加入 

	sh /fs/start.sh

按下键盘 `esc` 退回到命令模式,按下键盘 `:`输入 `wq` 保存退出

设置为每天晚上2点自动重启,清除缓存:

	crontab -e

这里同样会变成 vi 命令模式,按着同样方法加入: 

	0 2 * * * sh /fs/restart.sh

服务器端配置完成

#####电脑端安装

FinalSpeed客户端Windows版

<http://finalspeed.org/fs/finalspeed_install1.2.exe>

FinalSpeed客户端Java版,支持OS X,Linux

<http://finalspeed.org/fs/finalspeed_client1.2.zip>

下载后解压文件,在终端跳转到文件夹所在路径,假设 finalspeed_client.jar 所在路径为 /fsclient ,先切换到该路径cd /fsclient

执行 

	sudo java -jar finalspeed_client.jar

需要 root 权限运行,提示输入 mac 密码即可



系统需安装 java 运行环境,如果终端提示没有 -jar 命令,那么需要安装 jdk 工具,网上搜索下安装上电脑再回来继续


成功后要求输入上下带宽,按自行带宽填写即可,我填的 50 和 10

![](http://7xo0hj.com1.z0.glb.clouddn.com/%E4%BF%A1%E6%81%AF%E5%9B%BE%E5%83%8F%281057203959%29.png)

地址为 vps 地址 

点击添加,名称自定义,加速端口填写 ss 服务端口, 本地端口自定义,我填的 9090

打开 Shadowsocks 客户端,添加 `127.0.0.1` 端口填入刚才自定义的本地端口号,密码为 ss 密码

FinalSpeed 官方文档:

<http://finalspeed.org>

打开 youtubu 选择 1080p 看看效果吧!




