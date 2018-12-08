# VULTR云服务器搭建SS

这篇文章源于自己近来使用vultr折腾ShadowSocks而做的记录
**科学上网，请合法参考、使用本文**

- [本文地址-GitHub](https://github.com/wangjian92/ShadowSocks)
- [本文地址-简书](https://www.jianshu.com/p/216885c33ed8)

## PREPARE
大体流程是需要在vultr官网上使用邮箱注册登录，购买并设置服务器，在服务器上安装并搭建shadowsocks，本地安装shadowsocks客户端，按照服务器上设置的端口密码与服务器建立连接，完成。

### 实现原理
>下面摘自wikipedia
Shadowsocks的运行原理与其他代理工具基本相同，使用特定的中转服务器完成数据传输。
在服务器端部署完成后，用户需要按照指定的密码、加密方式和端口，使用客户端软件与其连接。在成功连接到服务器后，客户端会在本机上构建一个本地Socks5代理（或VPN、透明代理）。浏览网络时，网络流量会被分到本地Socks5代理，客户端将其加密之后发送到服务器，服务器以同样的加密方式将流量回传给客户端，以此实现代理上网。

### 提前准备好这些
[ShadowSocks](https://github.com/shadowsocks/shadowsocks-windows/releases)——我选用的是比较老的2.5.8版本
[xshell](https://xshell.en.softonic.com/)——用于操作远程服务
还有购买vultr服务需要准备至少$10...土豪请忽略这行

##  BEGIN
### 一、充值VULTR服务
VULTR官方地址：https://www.vultr.com/

![VULTR首页](https://upload-images.jianshu.io/upload_images/6177162-6176ab5c45ecfe40.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

需要输入邮箱及密码进行注册，登录之后进入下图，点击左侧 **Billing** 后，选择支付方式，编者使用的是Alipay支付宝，选好后会显示出二维码，用支付宝扫描后扣款，支付宝会计算费率。这里最少需要购买$10，扣款规则是会根据之后选择的服务器类型进行扣款，10刀如果选便宜的类型的话够用2~3个月。

![付款页面](https://upload-images.jianshu.io/upload_images/6177162-2b634a90b57a21c0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

顺利完成上面的步骤就充值成功了，再次回到 **Billing** 页面会在右上角看到余额。

### 二、配置VULTR服务
完成步骤一后，就可以在 **Servers** 中实例服务了，进入**Servers**，点击右上角的加号新建，进入下图页面：

![Servers实例服务页面](https://upload-images.jianshu.io/upload_images/6177162-7067d3545d2d67e3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在这里选择自己想要的配置，编者这里选择的地址是洛杉矶节点，类型是CentOS 7 x64，因为我只搭建ShadowSocks所以服务大小是20GB SSD，$3.50/mo，其他选项选填，点击**Deploy Now**，就完成一个配置了，可在Server中查看配置的信息。

![服务列表](https://upload-images.jianshu.io/upload_images/6177162-d837b631258f48ff.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![服务详情](https://upload-images.jianshu.io/upload_images/6177162-becde0684e99aabe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

至这里便完成了服务的购买充值并配置的准备工作。

### 三、连接配置好的远程服务器（windows环境）
使用XShell连接配置好的远程服务器，使用的参数有步骤二详情中的IP Address，Username，Password，都可以在页面上复制

![Xshell连接配置](https://upload-images.jianshu.io/upload_images/6177162-7126630e94f9f7a5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

之后输入用户名和密码进行连接，连接成功后会进入shell界面，之后就可以在其中操作，进行下一步：搭建ShadowSocks服务。

### 四、搭建ShadowSocks服务

在服务器上安装shadowsocks，顺序执行下面操作
```
yum install m2crypto python-setuptools
easy_install pip
pip install shadowsocks
```

配置shadowsocks
```
vi  /etc/shadowsocks.json
```

写入如下配置
```
{
    "server":"0.0.0.0", //这里写入IP Address
    "server_port":443, //端口
    "local_address": "127.0.0.1", //这里写入IP Address
    "local_port":8080, // 
    "password":"123456", //客户端shadowsocks连接服务器的密码
    "timeout":300,
    "method":"aes-256-cfb",
    "fast_open": false
}
```

配置防火墙
```
//安装防火墙
yum install firewalld
//启动防火墙
systemctl start firewalld
```

开启防火墙相应的端口
```
// 端口号是你自己设置的端口
firewall-cmd --permanent --zone=public --add-port=443/tcp
firewall-cmd --reload
```

启动 Shadowsocks 服务
```
// 后台运行    
ssserver -c /etc/shadowsocks.json -d start

// 调试时使用下面命令，实时查看日志
ssserver -c /etc/shadowsocks.json
```

### 五、本地ShadowSocks连接服务器
找到准备工作中下载的ShadowSocks安装包，解压双击exe文件，结合步骤四中的配置信息，进行配置
![shadowsocks配置](https://upload-images.jianshu.io/upload_images/6177162-3eabb0713f057fdc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

桌面托盘中右键Shadowsocks**启用系统代理**

这个时候再试试访问外网，just enjoy it！
