### KCPTUN加速shadowsocks原理图

![][img11]

> **右边的8899端口是ss服务器的端口,左边的那个8899是可以与ss的端口不同的随便写都行,只是为了方便,ss的客户端不用改端口,只需要将ip设置成"127.0.0.1"就行了而已**

### ss服务端设置:

![][img12]

### kcptun服务端设置

![][img13]

### PC客户端：

#### kcptun客户端设置

![][img14]

#### ss客户端设置

![][img15]

### 安卓客户端：

> **注意点是如下图所示**

![][img16]

> **上图要从原来的ss服务端端口(8899)改成kcptun的服务端端口(29900)**

![][img17]

配置格式按下面的去设置(具体数值要对应kcptun服务端的),设置后显示成上图

>**mtu=1350;rcvwnd=2048;parityshard=30;nodelay=0;resend=2;mode=fast3;interval=20;crypt=none;autoexpire=0;acknodelay;sockbuf=4194304;nc=1;datashard=70;dscp=46;keepalive=10;sndwnd=2048**

如果是新安装的shadowsocks-android或kcptun-android,都最好要重启下手机,这样链接才会生效,实验证明速度杠杠的!


> **KCPTun官方下载地址：https://github.com/xtaci/kcptun/releases/**
>
> **Windows版KCPTun客户端GUI：**
> **https://github.com/dfdragon/kcptun_gclient/releases/**
>
> **Mac版KCPTun客户端GUI：**
> **https://github.com/dfdragon/kcptun_xclient/releases/**
>
> **Shadowsocks windows客户端: https://github.com/shadowsocks/shadowsocks-windows/releases/**
>
> **Shadowsocks 安卓客户端: https://github.com/shadowsocks/shadowsocks-android/releases/**
>
> **kcptun 安卓Shadowsocks客户端插件: https://github.com/shadowsocks/kcptun-android/releases**

[img11]:/images/11.png
[img12]:/images/12.png
[img13]:/images/13.png
[img14]:/images/14.png
[img15]:/images/15.png
[img16]:/images/16.png
[img17]:/images/17.jpg
