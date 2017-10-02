# shadowsocksr-linux-client-CLI
在Linux上面使用shadowsocksr python客户端

## 安装步骤

* [在linux环境安装shadowsocksR客户端](https://www.djangoz.com/2017/08/16/linux_setup_ssr/)

    wget http://www.djangoz.com/linux_setup_ssr/ssr #ssr脚本已经被下载到git文件夹中
    sudo mv ssr /usr/local/bin
    sudo chmod 766 /usr/local/bin/ssr
    ssr install
    ssr config #目前只能配置单服务器版本，多服务器版本以后再添加

* 配置全局代理
	
> shadowsocks以及shadowsocksr服务端本身只能提供socks5代理，但是多数应用需要的传输协议是http和https的，所以需要一个代理软件把socks5的流量转换成http/https的流量，下面是一种开启全局代理的方式。

    #为了让整个系统都走sahdowsocks通道，需要配置全局代理，这里我们使用polipo软件。
    #安装polipo
    sudo apt-get install polipo
    #修改polipo的配置文件`/etc/polipo/config`:
    `logSyslog = true
    logFile = /var/log/polipo/polipo.log

    proxyAddress = "0.0.0.0"

    socksParentProxy = "127.0.0.1:1080"
    socksProxyType = socks5

    chunkHighMark = 50331648
    objectHighMark = 16384

    serverMaxSlots = 64
    serverSlots = 16
    serverSlots1 = 32`

    #重启polipo服务
    sudo /etc/init.d/polipo restart
    #为终端配置http/https代理
    export http_proxy="http://127.0.0.1:8123/"
    export https_proxy="https://127.0.0.1:8123/"
    #接着测试能不能翻墙,如果返回网页源码，则表示配置成功
    curl www.google.com
