# pthread

（待完善）

POSIX线程（POSIX threads）

Pthreads API中大致共有100个函数调用，全都以"pthread_"开头，并可以分为四类：

- 线程管理，例如创建线程，等待(join)线程，查询线程状态等。
- [互斥锁](https://baike.baidu.com/item/互斥锁)（Mutex）：创建、摧毁、锁定、解锁、设置属性等操作
- [条件变量](https://baike.baidu.com/item/条件变量)（Condition Variable）：创建、摧毁、等待、通知、设置与查询属性等操作
- 使用了互斥锁的线程间的[同步](https://baike.baidu.com/item/同步)管理

### 数据类型：

pthread_t：线程句柄

- 使用函数pthread_equal()对两个线程ID进行比较
- 获取自身所在线程id使用函数pthread_self()。

pthread_attr_t：线程属性。主要包括scope属性、detach属性、堆栈地址、堆栈大小、优先级。

- __detachstate：新线程是否与进程中其他线程脱离同步。
  - PTHREAD_CREATE_JOINABLE（缺省）
  - PTHREAD_CREATE_DETACHED，新线程不能用pthread_join()来同步，且在退出时自行释放所占用的资源。一旦设置为PTHREAD_CREATE_DETACHED状态，不论是创建时设置还是运行时设置，则不能再恢复到PTHREAD_CREATE_JOINABLE状态。
  - 可以在线程创建并运行以后用pthread_detach()来设置。
- __schedpolicy：新线程的调度策略
  - SCHED_OTHER（正常、非实时）（缺省）
  - SCHED_RR（实时、轮转法）
  - SCHED_FIFO（实时、先入先出）
  - 缺省为SCHED_OTHER，后两种调度策略仅对超级用户有效。运行时可以用过pthread_setschedparam()来改变。
- __schedparam，一个struct sched_param结构，目前仅有一个sched_priority整型变量表示线程的运行优先级。
  - 这个参数仅当调度策略为实时（即SCHED_RR或SCHED_FIFO）时才有效，并可以在运行时通过pthread_setschedparam()函数来改变，缺省为0。系统支持的最大和最小的优先级值可以用函数sched_get_priority_max和sched_get_priority_min得到。
- __inheritsched：
  - PTHREAD_EXPLICIT_SCHED（缺省）：新线程使用显式指定调度策略和调度参数（即attr中的值）
  - PTHREAD_INHERIT_SCHED：新线程继承调用者线程的值。
- __scope，表示线程间竞争CPU的范围，说线程优先级的有效范围。
  - PTHREAD_SCOPE_SYSTEM：与系统中所有线程一起竞争CPU时间
  - PTHREAD_SCOPE_PROCESS：仅与同进程中的线程竞争CPU。
  - 目前LinuxThreads仅实现了PTHREAD_SCOPE_SYSTEM一值。

pthread_barrier_t：同步屏障数据类型

pthread_mutex_t：mutex数据类型

pthread_cond_t：条件变量数据类型

### 函数

#### 线程操纵函数（简介起见，省略参数）:

pthread_create()：创建一个线程

pthread_exit()：终止当前线程

pthread_cancel()：请求中断另外一个线程的运行。被请求中断的线程会继续运行，直至到达某个取消点(Cancellation Point)。取消点是线程检查是否被取消并按照请求进行动作的一个位置。POSIX 的取消类型（Cancellation Type）有两种，一种是延迟取消(PTHREAD_CANCEL_DEFERRED)，这是系统默认的取消类型，即在线程到达取消点之前，不会出现真正的取消；另外一种是异步取消(PHREAD_CANCEL_ASYNCHRONOUS)，使用异步取消时，线程可以在任意时间取消。系统调用的取消点实际上是函数中取消类型被修改为异步取消至修改回延迟取消的时间段。几乎可以使线程挂起的库函数都会响应CANCEL信号，终止线程，包括sleep、delay等延时函数。
pthread_join()：阻塞当前的线程，直到另外一个线程运行结束
pthread_kill()：向指定ID的线程发送一个信号，如果线程不处理该信号，则按照信号默认的行为作用于整个进程。信号值0为保留信号，作用是根据函数的返回值判断线程是不是还活着。
pthread_cleanup_push()：线程可以安排异常退出时需要调用的函数，这样的函数称为线程清理程序，线程可以建立多个清理程序。线程清理程序的入口地址使用栈保存，实行先进后处理原则。由pthread_cancel或pthread_exit引起的线程结束，会次序执行由pthread_cleanup_push压入的函数。线程函数执行return语句返回不会引起线程清理程序被执行。
pthread_cleanup_pop()：以非0参数调用时，引起当前被弹出的线程清理程序执行。
pthread_setcancelstate()：允许或禁止取消另外一个线程的运行。
pthread_setcanceltype()：设置线程的取消类型为延迟取消或异步取消。

#### 线程属性函数：

**pthread_attr_init()**：初始化线程属性变量。

- 运行后，pthread_attr_t结构所包含的内容是操作系统支持的线程的所有属性的默认值。

pthread_attr_setdetachstate()：设置线程属性变量的detachstate属性（决定线程在终止时是否可以被joinable）

pthread_attr_getdetachstate()：获取脱离状态的属性
pthread_attr_setscope()：设置线程属性变量的__scope属性
pthread_attr_setschedparam()：设置线程属性变量的schedparam属性，即调用的优先级。
pthread_attr_getschedparam()：获取线程属性变量的schedparam属性，即调用的优先级。
pthread_attr_destroy()：删除线程的属性，用无效值覆盖

#### mutex函数：

pthread_mutex_init()：初始化互斥锁

pthread_mutex_destroy()：删除互斥锁

pthread_mutex_lock()：占有互斥锁（阻塞操作）

pthread_mutex_trylock()：试图占有互斥锁（不阻塞操作）。即，当互斥锁空闲时，将占有该锁；否则，立即返回。

pthread_mutex_unlock(): 释放互斥锁

pthread_mutexattr_(): 互斥锁属性相关的函数

#### 条件变量函数：

pthread_cond_init()：初始化条件变量

pthread_cond_destroy()：销毁条件变量

**pthread_cond_signal():** **发送一个信号给正在当前条件变量的线程队列中处于阻塞等待状态的线程，使其脱离阻塞状态，唤醒后继续执行**。如果没有线程处在阻塞等待状态，pthread_cond_signal也会成功返回。一般只给一个阻塞状态的线程发信号。假如有多个线程正在阻塞等待当前条件变量，则根据各等待线程优先级的高低确定哪个线程接收到信号开始继续执行。如果各线程优先级相同，则根据等待时间的长短来确定哪个线程获得信号。但pthread_cond_signal在多处理器上可能同时唤醒多个线程，当只能让一个被唤醒的线程处理某个任务时，其它被唤醒的线程就需要继续wait。POSIX规范要求pthread_cond_signal至少唤醒一个pthread_cond_wait上的线程，有些实现为了简便，在单处理器上也会唤醒多个线程。所以最好对pthread_cond_wait()使用while循环对条件变量是否满足做条件判断。
**pthread_cond_wait(pthread_cond_t, pthread_mutex_t):** 等待条件变量的特殊条件发生；pthread_cond_wait() 必须与一个pthread_mutex配套使用。该函数调用实际上依次做了3件事：**对当前pthread_mutex解锁、把当前线程挂起到当前条件变量的线程队列、被其它线程的信号唤醒后对当前pthread_mutex申请加锁。**如果线程收到一个信号被唤醒，将被配套的互斥锁重新锁住，pthread_cond_wait() 函数将不返回直到线程获得配套的互斥锁。需要注意的是，一个条件变量不应该与多个互斥锁配套使用。

pthread_cond_broadcast(): 某些应用，如线程池，pthread_cond_broadcast唤醒全部线程，但我们通常只需要一部分线程去做执行任务，所以其它的线程需要继续wait.

pthread_condattr_(): 条件变量属性相关的函数
线程私有存储（Thread-local storage）:
pthread_key_create(): 分配用于标识进程中线程特定数据的pthread_key_t类型的键
pthread_key_delete(): 销毁现有线程特定数据键
pthread_setspecific(): 为指定线程的特定数据键设置绑定的值
pthread_getspecific(): 获取调用线程的键绑定值，并将该绑定存储在 value 指向的位置中
同步屏障函数
pthread_barrier_init(): 同步屏障初始化
pthread_barrier_wait():
pthread_barrier_destory():

　　其它多线程同步函数：
pthread_rwlock_*(): 读写锁

　　工具函数：
pthread_equal(): 对两个线程的线程标识号进行比较
pthread_detach(): 分离线程
pthread_self(): 查询线程自身线程标识号
pthread_once()： 某些需要仅执行一次的函数。其中第一个参数为pthread_once_t类型，是内部实现的互斥锁，保证在程序全局仅执行一次。