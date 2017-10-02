# shadowsocksr-linux-client-CLI
在Linux上面使用shadowsocksr python客户端

## 安装步骤

* Linux安装shadowsocksR Python客户端

> [参考链接](https://www.djangoz.com/2017/08/16/linux_setup_ssr/)

    wget http://www.djangoz.com/linux_setup_ssr/ssr #ssr脚本已经被下载到git文件夹中
    sudo mv ssr /usr/local/bin
    sudo chmod 766 /usr/local/bin/ssr
    ssr install
    ssr config #目前只能配置单服务器版本，多服务器版本以后再添加

* 配置全局代理（可选）
	
> shadowsocks以及shadowsocksr服务端本身只能提供socks5代理，但是多数应用需要的传输协议是http和https的，所以需要一个代理软件把socks5的流量转换成http/https的流量，下面是一种开启全局代理的方式。

    #为了让整个系统都走sahdowsocks通道，需要配置全局代理，这里我们使用polipo软件。
    #安装polipo
    sudo apt-get install polipo
    #修改polipo的配置文件`/etc/polipo/config`:
    logSyslog = true
    logFile = /var/log/polipo/polipo.log

    proxyAddress = "0.0.0.0"

    socksParentProxy = "127.0.0.1:1080"
    socksProxyType = socks5

    chunkHighMark = 50331648
    objectHighMark = 16384

    serverMaxSlots = 64
    serverSlots = 16
    serverSlots1 = 32

    #重启polipo服务
    sudo /etc/init.d/polipo restart
    #为终端配置http/https代理
    export http_proxy="http://127.0.0.1:8123/"
    export https_proxy="https://127.0.0.1:8123/"
    #接着测试能不能翻墙,如果返回网页源码，则表示配置成功
    curl www.google.com

    #注意事项:
    #服务重启之后ssr start 和export http_proxy="http://127.0.0.1:8123/" && export https_proxy="https://127.0.0.1:8123/"要同时执行。

* 单独为Chrome浏览器配置代理软件SwitchyOmega

> [SwitchyOmega官网](https://www.switchyomega.com)

> 在上面的链接里下载适合自己浏览器版本的`SwitchyOmega`安装文件，打开Chrome浏览器地址栏输入`chrome://extensions`,打开文件管理器，将刚刚下载的`*.crx`文件用鼠标托入浏览器中`chrome://extensions`页面下开始安装。本文件附带一份SwitchyOmega.crx，如果不能安装则自己下载。

> 进入[SwitchyOmega设置教程](https://www.switchyomega.com/settings.html)，按照教程上面的设置插件，不过SSR服务开启才能访问Google。之后Chrome就可以正常科学上网了。

* 设置SSR服务开机自动启动

> 由于部分常用Linux发行版本都将初始化系统从`init.d`改成了`systemd`，而且题主本人的ubuntu也从16.04开始放弃了`init.d`改为使用`systemd`，所以今天使用systemd配置SSR服务开机自动启动。

    #创建服务控制文件
    vim /lib/systemd/system/ssr.service
    
    #服务控制文件内容：
    [Unit]
    #描述服务
    Description=shadowsocksR CLI client
    #用于指定服务启动的前置条件 
    After=network.target
    #帮助文件的地址如http://baidu.com/ ，可缺省
    Documentation=https://github.com/mengmengmengqiang/shadowsocksr-linux-client-CLI
    Wants=network.target

    [Service]
    Type=forking
    PIDFile=/var/run/shadowsocks.pid
    #服务启动命令，此项必填
    ExecStart=/usr/bin/python /usr/local/share/shadowsocksr/shadowsocks/local.py --pid-file /var/run/shadowsocks.pid -d start -c /usr/local/share/shadowsocksr/config.json
    #服务终止命令，可缺省
    ExecStop=/usr/bin/python /usr/local/share/shadowsocksr/shadowsocks/local.py --pid-file /var/run/shadowsocks.pid -d stop -c /usr/local/share/shadowsocksr/config.json
    
    #用来定义如何启动，以及是否开机启动
    [Install]
    #当服务开机启动后，会放入什么文件夹，影响启动顺序
    WantedBy=multi-user.target

> 之后可使用如下命令来使用ssr服务

    #修改ssr.service服务控制文件之后刷新守护进程
    sudo systemctl daemon-reload
    #设置ssr开机启动
    sudo systemctl enable ssr.service
    #取消ssr开机启动，并取消服务
    sudo systemctl disable ssr.service
    #开启ssr服务
    sudo systemctl start ssr.service
    #关闭ssr服务
    sudo systemctl stop ssr.service
    #查看ssr服务状态
    sudo systemctl status ssr.service
