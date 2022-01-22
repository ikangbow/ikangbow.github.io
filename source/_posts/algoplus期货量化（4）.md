---
title: algoplus期货量化（4）
date: 2022-01-19 20:50:49
tags: algoplus
category: algoplus
---

## Python的multiprocessing,Queue，Process

在多线程multiprocessing模块中，有两个类，Queue（队列）和Process（进程）

队列Queue：

Queue是python中的标准库，可以直接import引用在队列中;

Queue.Queue(maxsize)创建队列对象，如果不提供maxsize,则队列数无限制。

    # _*_ encoding:utf-8 _*_
	import Queue
	
	q = Queue.Queue(10)
	q.put('LOVE')
	q.put('You')
	print (q.get())
	print (q.get())


当一个队列为空的时候,用get取回堵塞,所以一般取队列的时候会用,get_nowait()方法,这个方法在向一个空队列取值的时候会抛一个Empty异常,所以一般会先判断队列是否为空,如果不为空则取值;

不阻塞的方式取队列

![](1147589.png)

判断队列是否为空，为空返回True，不为空返回False


![](11475891.png)

返回队列的长度

![](11475893.png)

Queue.get([block[, timeout]]) 获取队列，timeout等待时间  
Queue.get_nowait() 相当Queue.get(False) 
非阻塞 Queue.put(item) 写入队列，timeout等待时间  
Queue.put_nowait(item) 相当Queue.put(item, False)

Multiprocessing中使用子进程的概念Process：

from multiprocessing import Process

可以通过Process来构造一个子进程

p=Process(target=fun,args=(args))

再通过p.start()来启动子进程

再通过p.join()方法来使得子进程运行结束后再执行父进程

在multiprocessing中使用pool：

如果需要多个子进程时可以考虑使用进程池（pool）来管理

Pool创建子进程的方法与Process不同,是通过p.apply_async(func,args=(args))实现,一个池子里能同时运行的任务是取决你电脑CPU的数量,如果是4个CPU,那么会有task0,task1,task2,task3同时启动,task4需要在某个进程结束后才开始。

 

多个子进程间的通信:

多个子进程间的通信就要采用第一步中的队列Queue,比如,有以下需求,一个子进程向队列中写数据,另一个进程从队列中取数据;

    # _*_ encoding:utf-8 _*_

	from multiprocessing import Process,Queue,Pool,Pipe
	import os,time,random
	
	#写数据进程执行的代码：
	def write(p):
	    for value in ['A','B','C']:
	        print ('Write---Before Put value---Put %s to queue...' % value)
	        p.put(value)
	        print ('Write---After Put value')
	        time.sleep(random.random())
	        print ('Write---After sleep')
	
	#读数据进程执行的代码：
	def read(p):
	    while True:
	        print ('Read---Before get value')
	        value = p.get(True)
	        print ('Read---After get value---Get %s from queue.' % value)
	
	if __name__ == '__main__':
	    #父进程创建Queue，并传给各个子进程：
	    p = Queue()
	    pw = Process(target=write,args=(p,))
	    pr = Process(target=read,args=(p,))
	    #启动子进程pw，写入：
	    pw.start()
	    #启动子进程pr，读取:
	    pr.start()
	    #等待pw结束：
	    pw.join()
	    #pr进程里是死循环，无法等待其结束，只能强行终止：
	    pr.terminate()