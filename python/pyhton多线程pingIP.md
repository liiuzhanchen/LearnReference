
### 在使用ping命令ping多个网络地址时，一般是一个一个的ping，等待前一个结果出来后再ping后一个IP地址，本文使用python多线程写了一个简单的小程序，它支持同时ping多个IP地址。
需要注意的有以下几点：  
1、本代码将要处理的IP地址放入了Queue中，而Queue是线程安全的，能够确保每一次操作都是原子操作。这样就省去了自己管理多线程资源申请的问题。  
2、在pingIP地址时，使用了subprocess的call函数来调用ping命令。  
subprocess的作用是开启子进程来执行命令。  
3、ping -c 1 ip 命令的作用是只ping一次该IP地址就返回结果。  
### 流程介绍  
程序一开始开启三个子线程，每一个线程都执行函数pingme，进入一个循环等待中。  
然后主线程将所有需要的IP塞到Queue中，q.join()的调用是用来等待队列中的所有元素确保都被处理，也就是主线程等待三个子线程的while循环都退出，子线程也退出了，主线程再继续执行。
### 代码如下：

```python
from threading import Thread  
import subprocess  
from Queue import Queue  

num_threads=3  
ips=['127.0.0.1','192.168.3.119','183.232.231.173','183.232.231.174','183.232.231.175']
q=Queue()
def pingme(i,queue):
    while True:  
        ip=queue.get()  
        print 'Thread %s pinging %s' %(i,ip)  
        ret=subprocess.call('ping -c 1 %s' % ip,shell=True,stdout=open('/dev/null','w'),stderr=subprocess.STDOUT)  
        if ret==0:  
            print '%s is alive!' %ip  
        elif ret==1:  
            print '%s is down...'%ip  
        queue.task_done()  

#start num_threads threads  
for i in range(num_threads):  
    t=Thread(target=pingme,args=(i,q))  
    t.setDaemon(True)  
    t.start()  

for ip in ips:  
    q.put(ip)  
print 'main thread waiting...'  
q.join();
print 'Done'
```
