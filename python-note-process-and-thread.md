# python多进程-多线程-进程/线程池/同步-高并发

不管是单核CPU还是多核CPU，一旦任务数量超过核数，OS都会把每个任务轮流调度到每个核心上。OS实现多进程和多线程往往是通过时间片的形式执行的，即让每个任务(进程/线程)轮流交替执行，因为时间片切分的很小，以至于我们感觉多个任务在同时执行。

如果我们要同时执行多个任务怎么办？

有两种解决方案：

- 一种是启动多个进程，每个进程虽然只有一个线程，但多个进程可以一块执行多个任务。

- 还有一种方法是启动一个进程，在一个进程内启动多个线程，这样，多个线程也可以一块执行多个任务。

- 当然还有第三种方法，就是启动多个进程，每个进程再启动多个线程，这样同时执行的任务就更多了，当然这种模型更复杂，实际很少采用。

总结一下就是，多任务的实现有3种方式：

    多进程模式；
    多线程模式；
    多进程+多线程模式。

同时执行多个任务通常各个任务之间并不是没有关联的，而是需要相互通信和协调，有时，任务1必须暂停等待任务2完成后才能继续执行，有时，任务3和任务4又不能同时执行，所以，多进程和多线程的程序的复杂度要远远高于我们前面写的单进程单线程的程序。

线程是最小的执行单元，而进程由至少一个线程组成。
### 如何调度进程和线程，完全由操作系统决定，程序自己不能决定什么时候执行，执行多长时间。


## 多进程
Linux/Unix操作系统提供了fork()系统调用，fork执行一次会返回两次，分别在父进程和子进程内返回（先父进程再子进程），父进程返回的是子进程的ID，子进程返回0。父进程要记下所有子进程的ID，而子进程可以通过getppid()获得父进的ID。

在python中，可以通过from mutilprocessing import Process来创建进程类对象：
p = Process(target = run_proc, args = ('args1',...))
其中，run_proc是子进程函数，args是传入子进程函数的参数，用tuple格式传递
p.start()--开始执行子进程(函数)
p.join()--用来等待该进程结束，用于进程同步
举个例子：
    from multiprocessing import Process
    import os

    def run_proc(name):
        print 'Run child process %s (%s)' % (name, os.getpid())

    if __name__ == '__main__':
        print 'Parent process %s.' % os.getpid()
        p1 = Process(target=run_proc, args=('test1',))
        p2 = Process(target=run_proc, args=('test2',))
        p3 = Process(target=run_proc, args=('test3',))
        p4 = Process(target=run_proc, args=('test4',))
        print 'Process will start.'
        p1.start()
        p2.start()
        p3.start()
        p4.start()

在主进程中创建了4个子进程，但是子进程的创建顺序是由OS决定的，并不是按照程序中的顺序执行。
    #1
    Parent process 21410.
    Process will start.
    Run child process test4 (24745)
    Run child process test1 (24742)
    Run child process test2 (24743)
    Run child process test3 (24744)
    #2
    Parent process 21410.
    Process will start.
    Run child process test1 (24754)
    Run child process test4 (24759)
    Run child process test2 (24755)
    Run child process test3 (24756)

## 进程池和线程池
进程池和线程池性质相似，下面以线程池为例进行说明。

既然是“池”，我们很容易联想到“水池”，在使用水的过程中，水池起到了一个缓冲的作用，避免了频繁开关水龙头。同理，进程池也是通过事先划分一块系统资源区域，这组资源区域在服务器启动时就已经创建和初始化，用户如果想创建新的进程，可以直接取得资源，从而避免了动态分配资源(这是很耗时的)。

线程池内子进程的数目一般在3～10个之间，子线程都运行着相同的代码，并具有相同的属性，如优先级，PGID等。
当有新的任务来到时，主进程将通过某种方式选择进程池中的某一个子进程来为之服务。相比于动态创建子进程，选择一个已经存在的子进程的代价显得小得多。至于主进程选择哪个子进程来为新任务服务，则有两种方法：

- 1）主进程使用某种算法来主动选择子进程。最简单、最常用的算法是随机算法和 Round Robin （轮流算法）。

 2）主进程和所有子进程通过一个共享的工作队列来同步，子进程都睡眠在该工作队列上。当有新的任务到来时，主进程将任务添加到工作队列中。这将唤醒正在等待任务的子进程，不过只有一个子进程将获得新任务的“接管权”，它可以从工作队列中取出任务并执行之，而其他子进程将继续睡眠在工作队列上。

当选择好子进程后，主进程还需要使用某种通知机制来告诉目标子进程有新任务需要处理，并传递必要的数据。最简单的方式是，在父进程和子进程之间预先建立好一条管道，然后通过管道来实现所有的进程间通信。在父线程和子线程之间传递数据就要简单得多，因为我们可以把这些数据定义为全局，那么它们本身就是被所有线程共享的。

既然说了进程池和线程池，也不妨再了解下内存池，内存池和进程池的工作方式也很类似，都是以空间换取时间。

## 内存池

内存池是一种内存分配方式。通常我们习惯直接使用new、malloc等系统调用申请分配内存，这样做的缺点在于：由于所申请内存块的大小不定，当频繁使用时会造成大量的内存碎片并进而降低性能。

内存池则是在真正使用内存之前，先申请分配一定数量的、大小相等(一般情况下)的内存块留作备用。当有新的内存需求时，就从内存池中分出一部分内存块，若内存块不够再继续申请新的内存。这样做的一个显著优点是，使得内存分配效率得到提升。

from:http://blog.csdn.net/u011012049/article/details/48436427
--------------------------------------------------------------------------
最后用代码实现一下进程池：
    from multiprocessing import Pool
    import os, time, random

    def long_time_task(name):
        print 'Run task %s (%s)...' % (name, os.getpid())
        start = time.time()
        time.sleep(random.random()*3)
        end = time.time()
        print 'Task %s runs %0.2f seconds.' % (name, end-start)

    if __name__ == '__main__':
        print 'Parent process %s.' % os.getpid()
        p = Pool()
        for i in range(5):
            p.apply_async(long_time_task, (i,))
        print 'Waiting for all subprocesses done...'
        p.close()
        p.join()
        print 'All subprocesses done.'

    #ouput
    Parent process 21410.
    Run task 2 (25238)...
    Run task 1 (25235)...
    Run task 3 (25237)...
    Run task 0 (25233)...
    Waiting for all subprocesses done...
    Task 1 runs 0.36 seconds.
    Run task 4 (25235)...
    Task 0 runs 0.91 seconds.
    Task 3 runs 1.05 seconds.
    Task 2 runs 2.78 seconds.
    Task 4 runs 2.72 seconds.
    All subprocesses done.

python中Pool对象中默认子进程数量为4,因此任务4要等到进程池中某个子进程执行完后才能执行。当然可以自己设置Pool中子进程的数目。对Pool对象调用join()方法会等待所有子进程执行完毕，调用join()之前必须先调用close()，调用close()之后就不能继续添加新的Process了。
- apply_async(func[, args[, kwds[, callback]]]) 它是非阻塞的--异步
- apply(func[, args[, kwds]])是阻塞的-同步


## 进程间通信
有队列(Queue)和管道(Pipe)两种方式可用作进程间通信：

    #processes-communication by Queue

    from multiprocessing import Process, Queue
    import time, random

    def write(q):
        for value in ['A','B','C']:
            print 'Put %s to queue...' % value
            q.put(value)
            time.sleep(random.random())

    def read(q):
        while True:
            value = q.get(True)
            print 'Get %s from queue.' % value

    if __name__ == '__main__':
        q = Queue()
        pw = Process(target=write, args=(q,))
        pr = Process(target=read, args=(q,))

        pw.start()
        pr.start()

        pw.join()
        pr.terminate()

    #output
    Put A to queue...
    Get A from queue.
    Put B to queue...
    Get B from queue.
    Put C to queue...
    Get C from queue.


    #processes-communication by Pipes

    from multiprocessing import Process, Pipe
    import time, random

    def sendmsg(conn):
        for value in ['A', 'B', 'C']:
            print 'Send %s to Pipe' % value
            conn.send(value)
            time.sleep(random.random())

    def recvmsg(conn):
        while True:
            value = conn.recv()
            print 'Recv %s from Pipe' % value

    if __name__ == '__main__':
        conn1,conn2 = Pipe()
        ps = Process(target=sendmsg, args=(conn1,))
        pr = Process(target=recvmsg, args=(conn2,))

        ps.start()
        pr.start()

        ps.join()
        pr.terminate() 

    #output
    Send A to Pipe
    Recv A from Pipe
    Send B to Pipe
    Recv B from Pipe
    Send C to Pipe
    Recv C from Pipe

对比：
在父进程中创建两个子进程，一个往Queue中写数据，一个往Queue中读数据，队列式线程和进程安全的。

通过Pipe()生成的两个连接对象，表示管道的两端。
每个连接对象都有 send() 和 recv()方法。
要注意的是：如果两个进程(或线程)，在同一时间尝试从管道的同一端读或写时，数据会被变脏。当然，在同一时间，进程间使用不同的管道进行读写是没有任务风险的。


## 进程间同步

### Lock(锁)

    from multiprocessing import Process, Lock

    def f(l, i):
        l.acquire()
        print 'hello world', i
        l.release()

    if __name__ == '__main__':
        lock = Lock()

        for num in range(10):
            Process(target=f, args=(lock, num)).start()
        
       
        
### emaphore

### Event









## 多进程并发

## 多线程

一个进程内可以有多个线程（至少一个主线程)，多个线程共享该进程的所有变量，同时对全局变量进行访问和改写很容易出现混乱，所以需要用锁进行线程的同步控制。python由于设计时有GIL全局锁，所以多线程无法利用多核CPU,也不存在多线程并发的问题。

Python标准库中提供了_thread和threading两个模块来实现多线程，threading是_thread的高级封装，因此一般只需要import threading即可。
    
    import time, threading

    def loop():
        print('thread %s is running...' % threading.current_thread().name)
        n = 0
        while n < 5:
            n = n+1
            print('thread %s >>> %s' % (threading.current_thread().name, n))
            time.sleep(1)
        print('thread %s is end.' % threading.current_thread().name)

    print('thread %s is running...' % threading.current_thread().name)
    t = threading.Thread(target=loop, name='LoopThread')
    t.start()
    t.join()
    print('thread %s is end.' % threading.current_thread().name)

    #output
    thread MainThread is running...
    thread LoopThread is running...
    thread LoopThread >>> 1
    thread LoopThread >>> 2
    thread LoopThread >>> 3
    thread LoopThread >>> 4
    thread LoopThread >>> 5
    thread LoopThread is end.
    thread MainThread is end.

其实，多线程和多进程的实现很类似，进程启动时默认的线程为主线程，主线程之外可以开启新的子线程，current_thread()函数返回当前线程的实例。

## 多线程同步
多线程和多进程在数据共享方面有很大不同，多线程共享同一个进程的所有全局变量，多进程之间的数据是互不干扰的，因此，多线程的同步往往比进程同步要常见。和多进程同步一样，多线程之间的同步也有三种：
1. Lock
    import threading

    balance = 0
    lock = threading.Lock()

    def change_it(n):
        global balance
        balance = balance + n
        balance = balance - n

    def run_thread(n):
        for i in range(10000):
            lock.acquire()  #先获取锁
            try:        
                change_it(n)
            finally:
                lock.release()  #再释放锁

    t1 = threading.Thread(target=run_thread,args=(5,))
    t2 = threading.Thread(target=run_thread,args=(8,))
    t1.start()
    t2.start()
    t1.join()
    t2.join()

    print balance
线程锁需要先获取，执行完操作后再释放锁。




