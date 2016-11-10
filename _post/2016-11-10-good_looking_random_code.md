--- 
layout: post
title: 好认好记的验证码
---
其实验证码实现起来多简单的, Python不算上import random的话也就一行就搞定. 一般来说是这样子的:


    from random import sample
    "".join(sample('0123456789', 6))  
      

但是这样子生成的验证码确并不一定好用, 比如:


    from random import sample
    for i in range(10):
        print "".join(sample('0123456789', 6))
    

循环10次得到结果如下:

572041
278306
570843
301895
157230
978012
036485
872901
042571
802743

这些验证码的问题在于, 每位的数字都不相同, 所以在念出来的时候你需要记住6个数字的组合, 所以比较费脑, 对数字不敏感的同学尤其觉得困难.

如果将验证码中出现的数字降低到3个, 那么记忆起来的难度就会降低很多了.

所以我们先选3个数出来, 然后再组合成6位随机数


    from random import sample
    "".join(sample(sample('0123456789', 3), 6))
    

但是这样子写是没法运行的


    In [7]: "".join(sample(sample('0123456789', 3), 6))
    ---------------------------------------------------------------------------
    ValueError                                Traceback (most recent call last)
    <ipython-input-7-6f9b3c0172dc> in <module>()
    ----> 1 "".join(sample(sample('0123456789', 3), 6))
    
    /System/Library/Frameworks/Python.framework/Versions/2.7/lib/python2.7/random.pyc in sample(self, population, k)
        321         n = len(population)
        322         if not 0 <= k <= n:
    --> 323             raise ValueError("sample larger than population")
        324         random = self.random
        325         _int = int
    
    ValueError: sample larger than population
    

因为sample的大小要大于生成数列的大小, 所以我们修改一下, 将生成的sample乘以3就行了.


    from random import sample
    "".join(sample(sample('0123456789', 3)*3, 6))
    

这样子我们再生成10组随机数出来:

699955
004504
311553
449111
665655
639396
711222
949409
001210
151445

再看看是不是容易记多了? 起码能一次记住不用老看手机了.


祝玩得开心
