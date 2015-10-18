#Linux进程线程编程
##一、Linux进程
###1.main函数的入口
####crt0.o
* Linux下的实际入口
* 此文件中的内容是固定的
* 此入口调用main函数  
因此main函数返回之后还能进行一些处理工作
* cc/ld时自动链接
####main函数
	int main(int argc char *argv[])
###2.exec系列函数的使用
都用于启动一个新程序，切换到新的上下文
	#include <unistd.h>
	
	int execl(const char *path,const char *arg0,...,(char *)0);
	int execlp(const char *file,const char *arg0,...,(char *)0);
	int execle(const char *path,conse char *arg0,...,(char *)0,char *const envp[])
	int execv(const char *path,char *const argv[]);
	int execvp(const char *path,char *const argv[]);
	int execve(const char *path,char *const argv[],char *const envp[])
* 所有带l的系统调用，参数均以可变的方式提供
* 带p时给文件名即可
* 带e的系统调用还可修改当前的环境变量
* exec函数并不产生新的进程，只将当前进程的任务替换为执行新的任务  
返回后继续执行原来的任务
###3.fork的使用
	#include <sys/tyoes.h>
	#include <unistd.h>
	
	pid_t fork();
* 执行fork后有两个线程执行同一段代码
* 返回值为新PID的为原线程，继续执行  
新创建的进程获得的fork的返回值为0
####fork的使用范例
	if(fork()==0)
	{
		//子进程代码段
	}
	else
	{
		//父进程代码段
	}
将fork与exec两种系统调用结合起来，就可以创建新进程执行其它任务
###4.结束进程
####正常退出
* 从main函数返回
* 调用`exit`函数
* 调用`_exit`函数  
`exit`是C库函数，`_exit`是系统调用  
系统调用更贴近底层，可以立即结束一个进程  
	* *换而言之，`exit`还会做一些清理工作*  
	调用各种终止处理程序、清理标准I/O，最终才调用`_exit`
	* **终止处理程序**  
	可以自行编写，使用`atexit`函数

			#include <stdlib.h>
			int atexit(void (*function)(void));//向C库注册
	注册终止处理程序的本质是注册处理程序的入口地址
* 多线程程序的最后一个线程退出
####非正常结束
* 调用`abort`函数
* 被`kill`发送信号
* 最后一个线程被取消
###5.进程资源
struct task_struct（进程描述符）  
内部非常复杂，此处略过不讲
####其它资源
* 线程信息
* 文件信息
* 进程接受的信号信息
* 系统空间栈
###6.wait与waitpid
主要用于父子进程之间，等待进程结束
####原型
	#include <sys/types.h>
	#include <sys/wait.h>
	pit_t wait(int *status);			//由父进程调用，等待任意一个子进程
	pid_t waitpid(pid_t pid,int *status,int options);
* status用于收集被等待进程的返回值等信息
* options可以使waitpid非阻塞
####可能结果
* 阻塞
* 立即返回
* 出错
####实现原理
* 每个Linux下的进程在执行完毕后并不立即回收  
只有在父进程调用wait时才会将其资源释放出来
* 父进程调用`wait`时，将在系统中寻找其子进程的僵尸进程，并返回其pid
####waitpid
* 指定PID
* 非阻塞  
由options实现，置为`WNOHANG`即可，从而瞬时地看一下是否已经存在符合等待条件的僵尸进程
#####pid参数
* -1  
waitpid此时退化为wait
* >0  
等待对应PID的进程
* ==0  
回收当前进程组的子进程
* <0  
此参数为进程组组号
##二、信号
###1.信号的概念
* 本质是一个软中断
* 提供了处理异步事件的机制
* 每个信号都有以SIG开头的名字
* 信号被定义为正整数  
在<signal.h>中定义
####产生信号的方法
* 按终端键  
^C->SIGINT
* 硬件异常  
如内存错误：SIGSEGV
* kill函数
* kill命令
* 软件条件  
某种软件条件已经发生，并应将其通知有关进程->SIGALARM
####系统预定义的信号
* 见ppt
* 支持的所有信号可以在Linux中通过`man 7 signal`进行查询
###2.信号处理
* 忽略信号  
SIGKILL、SIGSTOP等严重错误不能被忽略
* 执行系统默认动作
* 捕捉信号
###3.signal函数
	#include <signal.h>
	
	typedef void (*sighandler_t)(int);
	sighandler_t signal(int signum,sighandler_t handler);
* 如果函数调用成功，返回上一个信号处理函数  
例如在执行程序某一段时希望程序不响应^C，可以在这段程序执行结束后再将其注册回去
* 如果出错，返回SIG_ERR
* handler参数
	* 用户定义的函数
	* SIG_DEF（默认信号处理）
	* SIG_IGN（忽略）
####代码范例
	static void sig_usr(int);
	
	int main()
	{
		if(signal(SIGUSR1,sig_usr)==SIG_ERR)
			err_sys("can't catch SIGUSR1");
		if(signal(SIGUSR2,sig_usr)==SIG_ERR)
			err_sys("can't catch SIGUSR2");
		
		//continue
	}
###4.信号的可靠性简述
* 信号可能在到达激活的响应函数前被丢弃
* 现代Linux将信号分为可靠信号和不可靠信号  
<SIGRTMIN（通常为32）的被认为是不可靠信号，其它的是可靠信号
* 不可靠信号可能会丢失  
如连续产生信号的情形，或多个信号同时到达
* 或是线程十分忙时，可能不会立即响应
* 信号的复位  
处理完后信号立即被重置，信号的响应函数在被调用过一次之后就改回原有的默认函数  
如果还需要继续处理，把么就要在处理函数中重新注册
###5.中断系统调用
低速系统调用在接受到信号时会被中断其等待

* 读/写某些类型的文件
* 打开某些类型设备
* pause
* wait
* ioctl
* 进程间通讯

此时返回出错值，errno==EINTR
####解决方法
	again:
		if((n==read(fd,buf,BUFFSIZE))<0)
		{
			if(errno==EINTR)
				goto again;					//just an interrupted system call
			
			//Handle other errors
		}
###6.可重入函数
即可以被中断的函数
####不可重入函数
尽量不要在信号处理函数中使用，可能有一定概率出错  
主函数无法控制信号产生的时机，因而调用这些不可重入函数的中间状态不可预料  
*从而对内核产生不可预料的影响*

* 系统资源
* 全局变量  
诸如++、--等非原子操作
* 使用静态数据结构
* 调用malloc或free
* 标准I/O函数

所有函数的清单见ppt
###7.发送信号
* 发送信号有权限要求
* root可以给所有进程发信号
* 低权限用户不能向高权限用户进程发送信号
####kill(2)
向一个指定进程发送信号

	#include <sys/types.h>
	#include <signal.h>
	
	int kill(pid_t pid,int sig);
* 可以使用`kill`发送任何信号
* pid一样可以多样取值，和waitpid相同
####raise(3)
向当前的进程（自己）发送信号

	#include <signal.h>
	
	int raise(int sig);
####alarm
过一段时间向自己发送SIGALARM信号，类似闹钟

	#include <unistd.h>
	
	unsigned int alarm(unsigned int seconds);
* 没有设定处理函数时，默认行为为退出
* 连续发送SIGALARM信号时，最新的alarm将替换掉原有的，返回出上次闹钟的剩余倒计时  
第一次调用，返回值为0
* alarm还可以用于对可能阻塞的操作做时间限制  
远离如前所述
####pause
等待一个信号

	#include <unistd.h>
	
	void pause();
####利用alarm与pause实现sleep
	void sig_alarm(int signo)
	{
		printf("alarm received.\n");
	}
	
	unsigned int sleep1(unsigned int nsecs)
	{
		if(signal(SIGALARM,sig_alarm)==SIG_ERR)
			return nsecs;
		
		alarm(nsecs);
		pause();
		
		return alarm(0);			//将闹钟清零而取消掉
	}
##四、可靠信号机制
###1.信号集操作
信号集本质就是一个集合

	#include <signal.h>
	
	int sigemptyset(sigset_t *set);
	int sigfillset(sigset_t *set);
	int sigaddset(sigset_t *set,int signum);
	int sigdelset(sigset_t *set,int signum);			//以上四个函数返回0为成功，-1为出错
	int sigismember(xonst sigset_t *set,int signum);	//返回值依照C标准
集合的返回均以参数的方式进行
###2，信号掩码
用于检测或更改（*或两者*）进程的信号掩码

	#include <signal.h>
	
	int sigprocmask(int how,const sigset_t *set,sigset_t *oldset);
oldset将返回出原有的信号集
####参数how
* SIG_BLOCK  
将信号并入掩码
* SIG_UNBLOCK  
在掩码中除去相应的信号
* SIG_SETMASK  
直接设定
####特殊情形
* 在sigprocmask调用后，任何未阻塞并且pending的信号，在函数返回前，至少有一个信号会送达进程  
所有信号集内的信号都会被屏蔽，但将会被置为pending，更多相同类型的信号将被丢弃  
被屏蔽的信号重新打开时，信号将被处理，并且一定在sigprocmask返回之前被处理
* **但SIGKILL、SIGSTOP例外**  
这两个信号无法被阻塞
####sigpending
返回当前未决的信号集

	#include <signal.h>
	
	int sigpending(sigset_t *set);		//set被用来输出信号集
####sigaction
用于检查或修改与指定信号关联的处理动作，与`signal`系统调用差不多，被用来将其替代

	#include <signal.h>
	
	int sigaction(int signum,const struct sigaction *act,struct sigaction *oldact);
* sigaction中除了函数指针以外，还会包含其它成员
	* sigaction中的屏蔽字仅在执行信号处理函数期间生效  
	当前正在响应的信号被默认直接屏蔽，不需设置
	* sa_flags可以有多种设定参数  
	见ppt
####sigsuspend
用sigmask临时替换信号掩码，在捕捉一个信号或发生终止进程的信号前，进程挂起

	#include <signal.h>
	
	int sigsuspend(const sigset *sigmask);	
* 等待信号时，信号可以是之前pending的，也可以是在等待过程中出现的
* 等待结束后，信号掩码仍然恢复到原先的设定
##五、共享内存
共享内存是内核为进程创建的特殊内存段，它可以连接到自己的地址空间，也可以连接到其它进程的地址空间

* 最快的进程间通信方式
* 不提供任何同步功能
* 线程共享内存的存储位置一定在**内核的内核态**
###1.系统调用
####分配
	#include <sys/types.h>
	#include <sys/ipc.h>
	#include <sys/shm.h>
	
	int shmget(ket_t key,int size,int flag);
* 要求系统分配一块共享内存，若成功，返回值即为内存区域的标识符  
由于通常按页分配内存，因此得到的值一般比size稍大
* key_t key是一个用户定义的标识符
* flag决定由系统找一块共享内存或是分配一块新的  
可以是NULL、`IPC_CREAT`、`IPC_PRIVATE`、`IPC_EXECL`、rwx权限  
其中定义为NULL时则为强制要求一块共享内存
####映射
进程工作在用户态，不会有权限去直接访问（尤其是写）内核的内容；向共享内存区域的读写权限被内核开放给了进程，映射就

	void *shmat(int shmid,void *addr,int flag)
* addr：可选参数，指定共享内存映射到的地址
* flag：`SHM_RND`、`SHM_RDONLY`
* 不同进程对同一块共享内存要求映射，得到的地址可能不同
####解除映射
####释放与查询
	int shmctl(int shmid,int cmd,struct shmid_ds *buf)
* 只有cmd置为`IPC_RMID`时，才进行释放操作
#####可查询的状态（不要求）
见ppt
###2.文件的内存地址映射
	void *mmap(void *addr,size_t length,int prot,int flags,int fd,off_t offset);
	int munmap(void *addr,size_t length);
* prot参数  
PROT\_READ  
PROT\_WRITE  
PROT\_EXEC  
PROT\_NONE
* flags参数  
MAP\_SHARED：文件可以被多个进程映射，但写入的内容不反映到文件上  
MAP\_ANONYMOUS：在全局空间创建一个可读写的文件  
MAP\_PRIVATE
##六、POSIX线程
###1.Linux的POSIX线程实现
* NGPT与NPTL  
两者已经合并，偏向后者
* 要对它们支持都必须对Linux内核进行更改
###2.编程预备条件
####pthread库
* /usr/lib/libpthread.so
* /usr/lib/libpthread.a
####pthread.h头文件
/usr/include/pthread.h
####编译选项
	gcc thread.c -o thread -lpthread
###3.命名习惯
所有名称以pthread_开头

* pthread\_
* pthread\_attr\_
* pthread\_mutex\_
* pthread\_mutexattr\_
* pthread\_cond\_
* pthread\_condattr\_
* pthread\_key\_
###4.基本线程函数
####创建线程
	#include <pthread.h>
	
	int pthread_create(pthread_t *thread,pthread_attr_t *attr,
		void* (*start_roiutine)(void *),void *arg);
####结束当前线程
	void pthread_exit(void *retval);
###5.Joinable与Detached线程
* 主线程默认等待所有子线程结束后才结束
* 如果要关闭这一特性，需要将子线程设定为Detached
* Linux下线程没有严格的主次之分
####相关函数
	int pthread_join(pthread_t th,void **thread_return);
	int pthread_detach(pthread_t th);
###6.线程同步
####用信号量进行同步
	#include <semaphore.h>
	
	int sem_init(sem_t *sem,int pshared,unsigned int value);
	int sem_wait(sem_t *sem);
	int sem_post(sem_t *sem);
	int sem_destroy(sem_t *sem);
	
	int sem_trywait(sem_t *sem);
	int sem_getvalue(sem_t *sem,int *sval);
* sem_trywait在信号量为0时，不会阻塞等待而立即失败返回，返回值为`EAGAIN`
####用互斥量进行同步
* 可以使用`PTHREAD_MUTEX_INITIALIZER`静态初始化
* 也可以使用相关函数动态初始化

		#include <pthread.h>
		
		int pthread_mutex_init(pthread_mutex_t *mutex,
						const pthread_mutexattr_t *mutexattr);
		int pthread_mutex_lock(pthread_mutex_t *mutex);
		int pthread_mutex_unlock(pthread_mutex_t *mutex);
		int pthread_mutex_destroy(pthread_mutex_t *mutex);
		int pthread_mutex_trylock(pthread_mutex_t *mutex);
####用条件变量进行同步
* 检查条件是否被满足  
队列的机制  
类似于Windows的事件机制
* 条件变量常与互斥量一同使用
* 条件变量提供了unlock与wait的复合原子操作  
多线程的互斥全部可以应用此模式
#####初始化条件变量
* 静态初始化

		pthread_cond_t convar=PTHREAD_COND_INITIALIZER;	//已由系统初始化好
一个宏并不只对应一个条件变量，可以被多次调用
* 动态初始化

		int pthread_cond_init(pthread_cond_t *cond,pthread_condattr_t *cond_attr);
*第二个参数在很多库里没有被实现，直接提供`NULL`即可*

静态初始化的条件变量不需要释放，但动态初始化的要销毁
#####销毁条件变量
	int pthread_cond_destroy(pthread_cond_t *cond);
#####可参与的工作方式
广播、等待、通知

	int pthread_cond_wait(pthread_cond_t *cond,pthread_mutex_t *mutex);
	int pthread_cond_timedwait(pthread_cond_t *cond,pthread_mutex_t &mutex,
		time_t *timeout);
	int pthread_cond_signal(pthread_cond_t cond);
	int pthread_cond_croadcast(pthread_cond_t cond);
* 通知只通知处于等待状态的一个条件变量  
随机的一个，具体是哪个无法被控制
* 广播将唤醒所有处于等待状态的条件变量
* wait操作在被唤醒时，互斥量会被自动锁上  
因此最终一定要手动再释放一次互斥量
#####使用模式
> ######主线程
> 负责初始化和释放
> 
> 1. 声明并初始化需要同步的全局数据和变量
> 2. 声明并初始化条件变量对象
> 3. 声明并初始化相关的互斥量
> 4. 创建工作线程
> 5. 等待并继续
> ######工作线程1
> 1. 工作，直到满足一个特定的条件
> 2. 锁上对应的互斥量并检查全局数据的值
> 3. 等待条件变量（此时互斥量自动解锁）
> 4. 手动地锁上互斥量
> 5. 继续工作
> ######工作线程2
> 1. 工作
> 2. 锁住相关的信号量
> 3. 改变全局数据的值
> 4. 检查被等待的全局数据的值，如果满足条件，唤醒一个工作线程1
> 5. 解锁信号量
> 6. 继续工作
##七、线程的更多讨论
###1.线程属性对象
pthread\_attr\_t
####初始化
	#include <pthread.h>
	
	int pthread_attr_init(pthread_attr_t *attr);
####get/set族函数
	int pthread_attr_setdetachstate(pthread_attr_t *attr,int detachstate);
	int pthread_attr_getdetachstategetdetachstate(const pthread_attr_t *attr,
		int *detachstate);
	int pthread_attr_setschedpolicy(pthread_attr_t *attr,
		int policy);
	int pthread_attr_getschedpolicy(const pthread_attr_t *attr,
		int *policy);
	int pthread_attr_setschedparam(pthread_attr_t *attr,
		int param);
	int pthread_attr_getschedparam(const pthread_attr_t *attr,
		int *param);
* detachstate属性  
`PTHREAD_CREATE_JOIN`/`PTHREAD_CREATE_DETACHED`
* schedpolicy属性  
`SCHED_OTHER`/`SCHED_RR`/`SCHED_FIFO`  
后两个都是实时调度策略
* schedparam属性  
调度参数，主要是优先级的设定
###2.终止线程
	int pthread_cancel(pthread_t thread);
	int pthread_setcancelstate(int state,int *oldstate);
	int pthread_setcanceltype(int type,int *oldtype);
* setcancelstate用于设置是否理会其它线程的cancel  
可传入`PTHREAD_CANCEL_ENABLE`或`PTHREAD_CANCEL_DISABLE`
* setcanceltype用于设置中止的时间点  
可传入`PTHREAD_CANCEL_DEFFERD`或`PTHREAD_CANCEL_ASYNCRONOUSD`  
决定直到需要任何等待时再退出或是立即退出
###3.多线程程序
####容易的错误
* 共享变量取法保护（未互斥使用）
* 常见线程时传递指针，指针指向的内容可能是共享的
####TLS
	int pthread_key_create(pthread_key_t *key,void (*destructor)(void*));
	int pthread_key_delete(pthread_key_t key);
	void* pthread_getspecific(pthread_key_t key);
	int pthread_setspecific(pthread_key_t key,const void *value);
* 与Windows不同在于，存入和取出的内容明确为void *类型
* 可以明确地指定传入数据的析构函数，用于对存入的指针指向的内容进行释放  
这里的析构函数在对应线程执行结束后自动执行  
*释放key时并不会释放key指向的内容，因此这里一定要手动进行释放*