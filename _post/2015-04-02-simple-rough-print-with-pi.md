---
layout: post
title: 不要驱动，简单粗暴的用树莓派驱动USB打印机
---

网上很多文章都是再说如何用树莓派来做一个通用打印服务器，但是在很多应用场景下，配置CUPS什么的真的是自己zuo自己die的好途径，各类linux下的驱动配置起来令人吐血。而驱动各种热敏票据打印机，比如打胶带啊，二维码贴纸啊，小票之类的打印机因为根本找不到linux的驱动，要搞起来更是Mission Imposiable。所以本文的目的就是为了不用驱动直接用USB接口的各类热敏打印机。因为没有驱动，所以我们只能用简单粗暴的方式通过USB直接操作打印机了。下面来看看怎么搞：

首先，你得有一台打印机，淘宝有卖的，几十元到一两百，可以打热敏胶带，所以做个打印服务器标签的东西也不错的，其他用途可以自行开发。

先把打印机用usb线接到树莓派上，然后在树莓派执行 lsusb 命令，这个时候会列表连接上的所有usb设备，如下:

    
    Bus 005 Device 001: ID 0000:0000 
    Bus 001 Device 001: ID 0000:0000 
    Bus 004 Device 001: ID 0000:0000 
    Bus 003 Device 001: ID 0000:0000 
    Bus 002 Device 006: ID 15d9:0a37 
    Bus 002 Device 001: ID 0000:0000

这个时候不知道谁是打印机呢！不过不要紧，你拔掉打印机的usb线后再执行一次，看缺谁，谁就是打印机了。

ID后冒号隔开的两个数字就是usb设备的 vendor ID和product Id了，记下来先，一会儿连接的时候有大用。

为了连接打印机，你需要安装python-usb这个库，用于直接通过usb接口来操作usb设备。本文的第一个坑就出在这里，因为pip库里的版本有一个bug的方式在后面的库会用到，所以必须用从github里最新的去除了bug的代码里安装才不会出问题。所以只能

    git clone https://github.com/walac/pyusb.git
    cd pyusb
    python setup.py install

用这样子的方式来安装才行。

安装好后我们就可以通过usb接口来操作打印机了，由于大多数打印机都支持EPSON的打印协议（很古老的协议了，所以到处都支持），所以我们可以安装一个叫python-escpos 的库来通过python-usb来用EPSON的协议操作打印机。

    sudo pip install python-escpos
    
但是此处还是有坑，因为这货的文档基本上和实际情况就是牛头不对马嘴。所以就别管这货的文档了。

    from escpos import *
    pt = printer.Usb(0x0fe6, 0x811e, 0, out_ep=0x03)

此处要注意  out_ep 不能用默认值，默认的铁定打不了，但是这里的封装又有问题不能去自动获取，所以下面给一段自动获取 out_ep 的代码

    import usb.core
    import usb.util
    import sys
 
    dev =  usb.core.find(idVendor= 0x5345, idProduct= 0x1234)
 
    cfg = dev.get_active_configuration()
    intf = cfg[(0,0)]
    ep = usb.util.find_descriptor(
        intf,
        # match the first OUT endpoint
        custom_match = \
        lambda e: \
        usb.util.endpoint_direction(e.bEndpointAddress) == \
        usb.util.ENDPOINT_OUT
    )
    dev.reset()
    
我手头的打印机获取到的out_ep是0x03,所以我就写的这个值。
之后呢就可以愉快的打印了：

    from escpos import *
    usb = printer.Usb(0x0fe6, 0x811e, 0, out_ep=0x03)

    usb.text(u"终于可以愉快的打印啦\n\n\n\n\n\n\n\n".encode('gbk'))

    usb.image(‘image path’)  #打印图片（黑白2值）

    usb.qr(‘值’)   #打印二维码

    usb.set(codepage=None, align=‘center’)  #设置页面居中
    
    usb.cut()  #切纸
    
    usb.close()  #关闭连接
    
    
祝玩得愉快。