# 多线程
多任务可以由多进程完成，也可以由一个进程内的多线程完成。执行运算时，很多时候以效率来说，多进程>单进程单线程>多线程，使用多线程甚至会拖后腿。那为何要使用多线程呢？多线程，形象来说就等于有十几个搬家工人，每个人都负责从房子里搬出东西，运输后放到新房子里去，但钥匙只有一把，意味着同时只能有一个工人能打开房间。但这把钥匙可以传送，即一个工人在路上运输的时候，他可以将钥匙传送给下一个工人，这样就提升了效率。这种类似搬家的操作便是IO操作，在IO密集型的操作中，使用多线程是最好的选择。例如在爬虫中，请求一个页面会有等待时间，这个时间内可以执行另一个线程的任务，这样速度就大大提升了。

##python中的多线程
在python中，由于GIL锁的存在，python在多线程里面其实是快速切换,以下为代码示例：
```
import time
import threading
 
def f0():
    pass
 
def f1(a1,a2):
    time.sleep(5)
    f0()
 
'''下面代码是直接运行下去的，不会等待函数里面设定的sleep'''
t= threading.Thread(target=f1,args=(111,112))#创建线程
t.setDaemon(True)#设置为后台线程，这里默认是False，设置为True之后则主线程不用等待子线程
t.start()#开启线程
 
t = threading.Thread(target=f1, args=(111, 112))
t.start()
 
t = threading.Thread(target=f1, args=(111, 112))
t.start()
#默认情况下程序会等线程全部执行完毕才停止的，不过可以设置更改为后台线程，使主线程不等待子线程，主线程结束则全部结束import time
import threading
 
def f0():
    pass
 
def f1(a1,a2):
    time.sleep(5)
    f0()
 
'''下面代码是直接运行下去的，不会等待函数里面设定的sleep'''
t= threading.Thread(target=f1,args=(111,112))#创建线程
t.setDaemon(True)#设置为后台线程，这里默认是False，设置为True之后则主线程不用等待子线程
t.start()#开启线程
 
t = threading.Thread(target=f1, args=(111, 112))
t.start()
 
t = threading.Thread(target=f1, args=(111, 112))
t.start()
#默认情况下程序会等线程全部执行完毕才停止的，不过可以设置更改为后台线程，使主线程不等待子线程，主线程结束则全部结束
```
在线程里面setDaemon（）和join（）方法都是常用的，他们的区别如下

（1）join ()方法：主线程A中，创建了子线程B，并且在主线程A中调用了B.join()，那么，主线程A会在调用的地方等待，直到子线程B完成操作后，

     才可以接着往下执行，那么在调用这个线程时可以使用被调用线程的join方法。join([timeout]) 里面的参数时可选的，代表线程运行的最大时

     间，即如果超过这个时间，不管这个此线程有没有执行完毕都会被回收，然后主线程或函数都会接着执行的，如果线程执行时间小于参数表示的

     时间，则接着执行，不用一定要等待到参数表示的时间。

 （2）setDaemon()方法。主线程A中，创建了子线程B，并且在主线程A中调用了B.setDaemon(),这个的意思是，把主线程A设置为守护线程，这

        时候，要是主线程A执行结束了，就不管子线程B是否完成,一并和主线程A退出.这就是setDaemon方法的含义，这基本和join是相反的。此外，还有

       个要特别注意的：必须在start() 方法调用之前设置，如果不设置为守护线程，程序会被无限挂起，只有等待了所有线程结束它才结束。
## 多线程里的锁
使用多线程时，为了防止各线程间数据进行交互出错，需要使用锁来进行保护，操作完了再释放。锁的使用比较复杂，下面介绍一种更快捷的使用方法。
## 使用队列
python中由queue库使用队列（2版本首字母大写），先将任务全部放入队列中，然后多线程的每一个线程轮次从队列中取任务，方便快捷，省去了使用锁的步骤，因为python的queue设计的是线程安全的，所以可以放心使用。下面的代码块是自己写的一个用于queue-threading配合的模版，这样写可以省去很多顾虑，如锁，父子进程这些，（同时也可以不用去学锁这些复杂的东西）大家可以参考下。
```
def get_proque():
    proque = Queue.Queue()
    for url_inf in url_infs:
        proque.put(url_inf)
    return proque

def get_source(proque):
    while not proque.empty():
        url_inf = proque.get_nowait()
        get_kahui(url_inf)

def run(nums):
    threads = []
    for i in range(nums):
        t = threading.Thread(target=get_source, args=(proque,))
        threads.append(t)
    for t in threads:
        t.start()
    for t in threads:
        t.join()
    print('over')
```
最后在main中调用：
```
proque = get_proque()
    run(15)
```
使用多线程最重要的是确定队列中放入什么，比如爬取淘宝，可以先爬取每一页 ，将商品的url放入队列，再利用多线程分别去爬取每一个商品的信息。另外，爬取url时也可以使用多线程，将page放入队列，实现每一页的多线程爬取。队列中数据的设置是多线程中最重要的一部分，好的设定可以大大提高效率，差的设定后最后可能和单线程的速度差不了多少。因为每个线程从队列中取任务，所以当队列中没有任务时，线程就会一个一个的减少，最后只剩下一个线程。所以在设定队列时，千万要注意，一个任务有100个网页，另一个任务只有2个网页这种情况。





