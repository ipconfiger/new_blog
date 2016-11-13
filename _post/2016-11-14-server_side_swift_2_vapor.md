--- 
layout: post
title: Swift in Server Side 第二弹
---
距离第一弹时间有点长了, 当时还在swift2.2的时候, 研究了半天觉得语言成熟度还不是辣么的高, 所以就没有放到在工程环境下使用的计划, 然后遇到一些事情耽误了就好几个月过去了, 现在Swift已经3.0了, 果然语法又大变了一番. 翻翻github发现又多了很多的team在ServerSide Swift的方向上发力, 其实本来是想找个ORM来研究下源码找找思路, 结果搜索到了Flunt, 然后发现是Vapor的一个组件, 然后就看到了Vapor.

看完官网发现教程完备, 看起来完成度蛮高的, 遂决定亵玩一番. 首先先检查一下本地环境, 打开终端, 输入:

    curl -sL check.vapor.sh | bash

然后如果显示

✅  Compatible

就表明你本地环境是OK的, 一般来说装了xcode8就能检测通过, 但是此处有坑. 因为不知为何, 虽然检测成功但是, 进行下一步安装的时候会报错....

所以为了以后环境切换也方便点, 还是装上swiftenv. 然后发现装了swiftenv后安装就没问题了

    git clone https://github.com/kylef/swiftenv.git ~/.swiftenv

然后将环境变量填到bash_profile

    $ echo 'export SWIFTENV_ROOT="$HOME/.swiftenv"' >> ~/.bash_profile
    $ echo 'export PATH="$SWIFTENV_ROOT/bin:$PATH"' >> ~/.bash_profile
    $ echo 'eval "$(swiftenv init -)"' >> ~/.bash_profile

新开个终端, 然后就生效了. 然后
    
    $swiftenv versions
      *system
      2.2-SNAPSHOT-2016-03-01-a
      DEVELOPMENT-SNAPSHOT-2016-03-01-a
      3.0.1
因为系统的装不上toolbox, 所以要切换到3.0.1

    $swiftenv global 3.0.1

切换后显示这样子就算ok了

    $swiftenv versions
      2.2-SNAPSHOT-2016-03-01-a
      DEVELOPMENT-SNAPSHOT-2016-03-01-a
     * 3.0.1 (set by /Users/liming/.swiftenv/version)

然后开始安装toolbox, 虽然没有toolbox也能用, 但是用了绝对要方便N多

    curl -sL toolbox.vapor.sh | bash

等待下载, 编译后就安装完毕了, 这个时候输入

    $vapor --help
    Usage: vapor <new|build|run|fetch|clean|test|xcode|version|self|heroku|docker>
    Join our Slack if you have questions, need help,
    or want to contribute: http://vapor.team

就能看到说明了, new 应该是新建一个项目的命令, 我们help一下看看:

    $vapor new --help
    Usage: vapor new <name> [--template]
    Creates a new Vapor application from a template.
        name: The application's executable name.
    template: The template repository to clone.
              Default: https://github.com/vapor/basic-template.

确认了和猜测的一样, 所以我们来新建一个项目

    vapor new Hello

然后因为要从github上下东西, 而国内的网络嘛, 所以会比较慢, 等一阵后会出现
    
    Cloning Template [Done]
    
                                                                                             **
                                                                                           **~~**
                                                                                         **~~~~~~**
                                                                                       **~~~~~~~~~~**
                                                                                     **~~~~~~~~~~~~~~**
                                                                                   **~~~~~~~~~~~~~~~~~~**
                                                                                 **~~~~~~~~~~~~~~~~~~~~~~**
                                                                                **~~~~~~~~~~~~~~~~~~~~~~~~**
                                                                               **~~~~~~~~~~~~~~~~~~~~~~~~~~**
                                                                              **~~~~~~~~~~~~~~~~~~~~~~~~~~~~**
                                                                              **~~~~~~~~~~~~~~~~~~~~~~~~~~~~**
                                                                              **~~~~~~~~~~~~~~~~~~~~~++++~~~**
                                                                               **~~~~~~~~~~~~~~~~~~~++++~~~**
                                                                                ***~~~~~~~~~~~~~~~++++~~~***
                                                                                  ****~~~~~~~~~~++++~~****
                                                                                     *****~~~~~~~~~*****
                                                                                        *************
    
                                                                               _       __    ___   ___   ___
                                                                              \ \  /  / /\  | |_) / / \ | |_)
                                                                               \_\/  /_/--\ |_|   \_\_/ |_| \
                                                                                 a web framework for Swift
    
                                                                              Project "Hello" has been created.
                                                                       Type `cd Hello` to enter the project directory.
                                                                                           Enjoy!

这样子很屌的ascii艺术, 然后项目的结构就建好了.

    $cd Hello

进入项目的目录, 然后新建xcode的工程

    $vapor xcode
    No Packages folder, fetch may take a while...
    Fetching Dependencies [Done]
    Generating Xcode Project [Done]
    Select the `App` scheme to run.
    Open Xcode project?
    y/n>y
    Opening Xcode project...

这个会等待更久的时间, 你可以趁这个机会好好看看文档. 执行完毕后会自动打开xcode, 
![](http://ww1.sinaimg.cn/large/578b198bgw1f9qytfrsu3j20w40jcmza.jpg)
将项目设置成可执行, 然后运行之

![](http://ww3.sinaimg.cn/large/578b198bgw1f9qyv6vdyij216011udju.jpg)
然后在浏览器上输入 http://127.0.0.1:8080
就能看到

![](http://ww1.sinaimg.cn/large/578b198bgw1f9qywn239wj210e0uota2.jpg)
通过xcode里的源码结构可以看到
![](http://ww2.sinaimg.cn/large/578b198bgw1f9qyyeex53j20ho14mtbr.jpg)
这货打包了一大堆的库进来, 还真是全家桶式的Framework啊.
下一步打算先用这个做个最简单的ToDO.List的程序试试手, 如果好用的话应该会在春节的时候试试写个社区什么的看看.