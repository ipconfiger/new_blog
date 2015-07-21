---
layout: post
title: 百撕不得骑姐系列之丢失的signal
---


过节的时候写了个小东西，主进程用multiprocess开了几个子进程来分发任务给子进程执行，完成后子进程report结果给主进程。为了在主进程死掉后把子进程也干掉，我监听了TERM的信号，然后在收到这个信号后去kill掉所有子进程。看起来ok啦，跑起来也还好，假设这个功能是放在 serv.py 里，python serv.py & 把它放到后台去，再kill 掉，一切都很ok，但是如果用supervisord把这个管理起来的话就出了问题，主进程基本上就没收到TERM的信号.

    def receive_signal(signum, stack):
	    import sys
	    for process in worker_process:
	        process.terminate()
	    logging.info('child killed')
	    sys.exit(0)
	
	signal.signal(signal.SIGTERM, receive_signal)
	
supervisord的配置

    [program:XXX]
	command=/usr/bin/python serv.py
	directory=/var/code/
	umask=022
	startsecs=0
	stopwaitsecs=0
	redirect_stderr=true
	stdout_logfile=/var/log/XXX.log
	autorestart=true
	
跑起来的时候 tail -f /var/log/XXX.log,在你 supervisorctl stop XXX 之后，是看不到child killed打印出来的。

遍阅各种资料发现supervisor是通过向子进程发TERM信号来杀进程的。但是这么貌似没有发过这个信号呢？然后我看到 stopwaitsecs=0 这个参数，当初设置成0是为了更快的重启进程。仔细想想是不是因为这个地方杀太快了还没发信号就把进程干掉了呢？立即把参数改成了  stopwaitsecs=1，然后，就解决了。

这个问题差不多花了一部电影的时间来解决，刚开始的时候真是百撕不得骑姐啊。回过头来看看，supervisor应该是用类似kill -9的方式来确保进程最终被干掉，但是如果stopwaitsecs=0的时候就干脆绕开了发TERM的信号直接就kill -9了，还是百撕不得骑姐，有机会看看supervisord的代码来看看是不是和我猜测的一样了。