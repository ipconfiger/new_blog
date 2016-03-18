--- 
layout: post
title: 用swift在开发服务端应用
---
去年Swift开源的时候就打算试试用Swift来开发服务端了, 不过事情太多, 很多轮子也不具备, 大抵上需要自己重写很多东西, 但是人民的力量是强大的, 几个月过去后, 大批的轮子已经在路上了, 由于IBM的金大腿加入, Swift进入后端开发的进度明显的加快了.IBM的云计算服务已经支持Swift作为后端服务的开发语言, 并且推出了Swift的Web框架:[Kitura ](https://github.com/IBM-Swift/Kitura) . 不过经过两天试用后发现此货编译尤其麻烦, release编译至今没有通过过一次, 且debug模式下性能问题也很突出, 遂放弃之, 后来在全球著名的同性交友网站上找到另外一个star超高的框架:[Swifton](https://github.com/necolt/Swifton)

这个框架的名字很有意思

![](http://ww1.sinaimg.cn/large/578b198bgw1f20xcpria0j20zi0l2afy.jpg)
由于路由这些都是照搬Rails, 而且又没有ORM, 脚手架命令更是空话, 文档就更是因陋就简了, 所以如果swift比较熟练的话自己RTFC是比较好的选择.
## 环境预备
Swift的服务端环境现在而今眼目下的状况是比较局促的, 如果不差钱买个垃圾桶拿去托管, 也ok, 要是自己准备Linux环境的话也就只能Ubuntu了,幸亏这个还是比较主流的,不过好些项目都在Centos上, 😂😂😂😂😂😂
本地开发环境比较好解决, 多亏了IBM的金大腿, [Kitura ](https://github.com/IBM-Swift/Kitura)  的Reponsitory里有Docker的脚本和Vagrant的vagrantfile. 由于我在本地开发用的是Vagrant, 所以只需要copy这个file到项目目录下, 然后vagrant up, 经过漫长的等待(大概20分钟), Swift的本地调试环境就搭建好了. 

## 开始快乐的coding
首先建立一个项目

    $~/project_path>swift build --init 
    Creating Package.swift
    Creating .gitignore
    Creating Sources/
    Creating Sources/main.swift
    Creating Tests/
    
然后加入Swifton的依赖, vim Package.swift, 加入依赖后就是下面这样子:

    dependencies: [
                  .Package(url: "https://github.com/necolt/Swifton.git",     versions: Version(0,0,5)..<Version(1,0,0)),
                  .Package(url: "https://github.com/necolt/Curassow.git", versions: Version(0,4,0)..<Version(1,0,0)),
             ]
         
去中Curassow是一个嵌入式的Http Server, 部署的时候用它来部署, 好处是在生产环境可以直接启动多个进程来提高性能.
之后用

    $~/project_path>swift build

就开始下载依赖开始编译了. 不过世事无常, 这里编译必定是会出错的. 因为Swift  build在Dev版加入了Test功能, 然后有个bug, 会和很多自带了Test项目的Tests目录冲突, 编译的时候报找不到Tests目录下的资源. 所以这里需要手动删除两个项目下的Tests目录

    $~/project_path>rm -rf ./Packages/PathKit-0.6.1/Tests
    $~/project_path>rm -rf ./Packages/Stencil-0.5.3/Tests

之后再运行编译就会直接通过了.
当然现在啥也干不了, 只能证明环境通顺了, 下面开始正式coding.

先创建一个Controller.

    $~/project_path>touch ./Sources/HelloController.swift

写入如下内容:

    import Swifton

    class HelloController:Controller{
        override init() {
            super.init()
            action("index") {request in
                return renderJSON(["hello": "world"])
            }
        }
    }

然后在main.swift里这么写:

    import Swifton
    import Curassow
    let router = Router()
    router.get("/", HelloController()["index"])
    serve { router.respond($0) }

然后编译

    $~/project_path>swift build
    Compiling Swift Module 'test00' (2 sources)
    Linking project_name

运行:

    $~/project_path>.build/debug/project_name
    [2016-03-18 12:34:23 +0800] [80691] [INFO] Listening at http://0.0.0.0:8000    (80691)
    [2016-03-18 12:34:23 +0800] [80692] [INFO] Booting worker process with pid: 80692

这个时候在浏览器里打开 http://127.0.0.1:8000 


打不开

对的, 不看到这里的人是会掉坑里的, 我故意的.
因为在Kitura的vagrantfile里映射的端口是8090到8090, 所以8000在外边是访问不到的, 有两个方法可以解决, 一个是修改启动的方式, 用参数 --bind 0.0.0.0:8090. 或者直接修改vagrantfile里的端口映射部分, 多加个8000到8000的映射即可
然后用浏览器打开, 就会出现久违的 

    {"hello": "world"}

##总结一下
大体上来说用Swift来开发服务端应用应该具备可以玩耍的能力了, 但是用在生产环境下现在还缺乏大量的轮子以及文档和实操经验.
通过对Swifton的代码研究发现, Swift的表达能力还是很强的, Swifton的代码相当的短小精悍. Curassow也是一个仿Gunicorn的Server, 纯Swift实现, 也非常的值得研究代码, 性能实测由于Mac本地并发数的限制没有探到头, 有空找两台Ubuntu的机器来实测一下Curassow