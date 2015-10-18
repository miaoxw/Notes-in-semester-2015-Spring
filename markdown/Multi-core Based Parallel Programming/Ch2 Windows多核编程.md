#Windows多核编程
##一、进程的概念
###1.1978年全国操作系统学术会议
> 进程是一个可以并发执行的具有独立功能的程序关于某个数据集合的一次执行过程，也是操作系统进行资源分配和保护的基本单位。

* 并发性
* 数据隔离
###2.SUSv3
* 有一个或多个线程
* 共享同一个地址空间
* 这些线程要求的系统资源
###3.进程的组成部分
####内核对象
存放进程的管理信息
####独立的地址空间
对同一个进程而言，其地址空间内包含了所有代码段和数据段

进程的开销相对比较大
###4.Windwos对多线程的支持
单CPU机器：不同线程的轮转调度
###5.线程
####线程的组成部分
#####内核对象
存放线程的管理信息
#####线程栈
* 不需要独立的地址空间
* 线程栈保存函数的调用历史、参数信息和局部变量
* 每个线程都要维护一个线程栈
###6.进程的特点
####独立性
* 进程是系统中资源分配和保护的基本单位，也是系统调度的独立单位
* 凡是未建立进程的程序，都不能作为独立单位参与运行
* 通常每个进程都可以以各自独立的速度在CPU上推进  
又称为**异步性**  

*各进程难免会出错，此时希望进程出错不会导致整个系统的瘫痪，因此才将各进程设计为独立的*
##二、线程API的实现层次
* 操作系统层（如WIN32 API）
* 库或运行时环境（MFC、.Net框架）
* 专门的多线程库（pthread）
###1.C/C++ Runtime Library
* libc.lib  
单线程默认系统库
* libcd.lib  
libc.lib的debug版本
* libcmt.lib  
多线程默认系统库
* libcmtd.lib  
libcmt.lib的debug版本
* msvcrt.lib  
加载动态链接库，同时支持单线程与多线程
* msvcrtd.lib  
msvcrt.lib的debug版本
####多线程应用程序的编译、链接
* 早期VS版本中，通过选择运行时库配置编译、链接选项
* 新VS中，选择方式类似，在项目属性中选择  
代码生成->运行时库
####链接不同库的区别
* 单线程与多线程库的区别显然
* 动态库在运行时加载，没有链接到目标代码
	* 生成一个dll时，对应生成一个lib，里面定义了各个函数的接口
	* lib中没有函数的实际执行内容，仅用于链接，在运行时寻找适当的文件载入
	* lib也可以不提供，但此时源代码中需要额外写一些代码
###2.Win/MFC线程API
* 创建线程
* 管理线程
* 线程同步
	* 事件、其它同步对象、互锁函数（原子操作）
	* 消息队列
* 其它
	* 线程池
	* 线程优先级
	* 处理器亲和  
	进程的处理器亲和可以直接在`taskmgr`中做，而线程的不行
####创建线程
* 最普通、最简单的  
不可能绕过这对API创建线程，但这对API实际用得较少

		HANDLE CreateThread(LP_SECURITY_ATTRIBUTES lpThreadAttributes,
				//用于确定子线程的权限，确定一些线程的安全值，没有特殊需要时，提供NULL即可
			DWORD dwStackSize,			//线程栈的大小，可以置0，此时意义为提供默认大小
			LPTHREAD_START_ROUTINE lpStartAddress,		//线程主函数的函数指针
			LPVOID lpParameter,			//传递给线程主函数的参数内容
			DWORD dwCreationFlags,		//标志位，定义线程立即执行(0)或创建后先挂起(CREATE_SUSPENDED)
			LPDWORD lpThreadId);		//输出参数，将新线程的ID填充到此位置
		
		VOID ExitThread(DWORD dwExitCode);
		
		DWORD WINAPI ThreadFunc(LPVOID data)
		{
			//Do the work of the thread
			return 0;					//退出时会自动地隐式调用ExitThread(0)
		}
	* 句柄理解为指向一个复杂结构的指针，本身占用的内存不大
	* 返回值是当前线程对象的指针
	* 所有LP*类型都是对应类型的指针
	* DWORD表示双字长整数
* 广泛使用的

		unsigned long _beginthreadex(void *security,
			unsigned stack_size,
			unsigned (*start_address)(void *),
			void *arglist,unsigned initflag,
			unsigned *theradID);
		void _endthreadex(unsigned retval);
	* 与上面一对相比，多做了些处理工作
	* 参数类型与功能和上面的没有任何区别
	* API被进一步封装，参数用起来更舒服
	* 安全性比直接调用API更好  
	上下文增加了对多个线程同时调用库函数时内存泄漏和访问冲突的检查和预防
	* 存在于C/C++运行时库的多线程版本中
* MFC

		CWinThread AfxBeginThread(AFX_THREADPROC pfnThreadProc,
			LPVOID pParam,
			int nPriority=THREAD_PRIORITY_NORMAL,
			...
		);
		AfxEndThread();
####获取进程自身句柄
	HANDLE GetCurrentProcess();
	HANDLE GetCurrentThread();
	
	DWORD GetCurrentProcessId();
	DWORD GetCurrentThreadId();
####管理线程
	DWORD SuspendThread(HANDLE hThread);
	DWORD ResumeTHread(HANDLE hThread);
	BOOL TerminateThread(HANDLE hThread,DWORD dwExitCode);
* 手动中止线程需要当心内存泄漏
* 随意挂起、中止线程需要当心死锁的问题  
挂起、中止线程时不会自动释放资源
####同步
#####同步方式的分类
* 用户态机制
* 内核态机制

功能复杂时，使用内核态机制进行同步，只需要简单同步时，用户态机制可以得到更快的速度
#####用户态机制
######互锁函数
* 以原子操作的方式修改一个值
* 相比其它同步（互斥）方式，速度极快
* 实现时，互锁函数对总线发出硬件信号，防止另一个CPU访问同一内存地址

* 对单个变量的原子操作

		InterlockedIncrement(PLONG);			//使一个LONG变量加1
		InterlockedDecrement(PLONG);			//使一个LONG变量减1
		InterlockedExchangeAdd(PLONG,LONG);		//将参数2加到参数1指向的变量中
		InterlockedExchange(PLONG,PLONG);		//将参数2的值赋给参数1指向的值，返回原始值
		InterlockedExchangePointer(PVOID*,PVOID);//功能同上
		InterlockedCompareExchange(PLONG,LONG,LONG);
					//将参数1指向的值与参数3比较，相同则把参数2的值赋给参数1指向的值，不同则不变
		InterlockedCompareExchangePointer(PVOID*,PVOID,PVOID);//功能同上
	* 几乎全部使用指针类型进行操作
* 对单链表的原子操作

		InitializeSListHead();
		InterlockedPushEntrySList();
		InterlockedPopEntrySList();
		InterlockedFlushSList();
		QueryDepthSList();
######自旋锁（循环锁）
* 非阻塞锁  
不断地尝试来获得该锁
* 应用与锁**持有的时间（受保护资源使用时间）较短**的情形
* 此锁占用CPU很大  
单核计算机中要尽量避免使用
* 可以用InterlockedExchange来实现循环锁

		while(InterlockedExchange(&g_fResourceInUse,TRUE))
			sleep(0);
######临界区对象
* 保证临界区内所有被访问的资源不被其它线程访问，直到当前线程执行完临界区代码

		void InitializeCriticalSection(LPCRITICAL_SECTION);
		void EnterCriticalSection(LPCRITICAL_SECTION);
		void LeaveCriticalSection(LPCRITICAL_SECTION);
		void DeleteCriticalSection(LPCRITICAL_SECTION);
* 例子

		CRITICAL_SECTION csMyCriticalSection;
		
		void CriticalSectionExample()
		{
			InitializeCriticalSection(&csMyCticicalSection);
			__try
			{
				EnterCriticalSection(&csMyCriticalSection);
				//Do something
			}
			__finally
			{
				LeaveCriticalSection(&csMyCriticalSection);
			}
		}
	* try-catch机制用于防止代码的异常导致其它线程继续被阻塞
#####内核态机制
######可用于同步的内核对象
* Processes, Threads, Jobs, Files
* Events
* Semaphore, Mutexes
* File change notifications
* Waitable timers
* Console input

每个内核对象，要么位于触发状态，要么位于未触发状态
######等待函数
线程可以等待内核对象的触发

	DWORD WaitForSingleObject(HANDLE hObject,DWORD dwMilliseconds)  
* 返回值有WAIT\_OBJECT\_0, WAIT\_TIMEOUT, WAIT\_FAILED三种
* 出错指内核对象在等待过程中被释放等情形

---

	DWORD WaitForMultipleObjects(DWORD dwCount,
		CONST HANDLE *phObjects,BOOL fWaitAll,DWORD dwMilliseconds)  

* fWaitAll用于指定这些内核对象的等待关系是且还是或
* 返回值在WAIT\_OBJECT\_0,...,WAIT\_OBJECT\_0+dwCount-1（依赖于第一个触发的对象）以及两种错误状态中选择

---

* ***等待副作用***
	* 在等到内核对象的过程中，等待函数会将已经触发的内核对象重置为未触发状态
	* 可能会造成死锁
######事件对象
* 是Win32提供的最灵活的线程间同步方式
* 事件的状态  
激发与未激发
* 事件分类
	* 手动设置  
	只能用程序来手动设置，在需要该事件或者事件发生时，采用SetEvent及ResetEvent来进行设置
	* 自动恢复  
	一旦事件发生并被处理后，自动恢复到没有事件状态，不需要再次设置
* 事件API
	* CreateEvent()
	
			HANDLE CreateEvent(
				PSECURITY_ATTRIBUTES psa;
				BOOL fManualReset,
				BOOL fInitialState,
				PCTSTR pszName);
	* SetEvent()  
	将事件设置为激发状态

			BOOL SetEvent(HANDLE hEvent);
	* ResetEvent()  
	将事件重置为未激发状态

			BOOL ResetEvent(HANDLE hEvent);
* ***补充：事件在进程中的共享***
######信号量对象
* 创建信号量

		HANDLE CreateSemaphore(
			LPSECURITY_ATTRIBUTES lpSemaphoreAttributes,
			LONG lInitialCount,
			LONG lMaximumCount,
			LPCTSTR lpName);
* 测试信号量  
之前介绍的WaitForSingleObject()
* 释放信号量

		BOOL ReleaseSemaphore(
			HANDLE hSemaphore,
			LONG lReleaseCount,
			LPLONG lpPreviousCount);
######互斥量对象
* 创建互斥量

		HANDLE CreateMutex(
			LPSECURITY_ATTRIBUTES lpMutexAttributes,
			BOOL bInitialOwner,
			LPCTSTR lpName);
* 测试互斥量  
之前介绍的WaitForSingleObject()
* 释放互斥量

		BOOL ReleaseMutex(HANDLE hMutex);
####线程池
一些应用需要动态创建线程

* 如Web服务器  
不能使用单线程处理用户响应
* 创建/销毁线程的开销很大  
用户量很大时，开销大到不能接受
* 因此预先将这批线程创建好
* 由操作系统提供支持
#####线程池API
	BOOL QueueUserWorkItem(
		LPTHREAD_START_ROUTINE Function,	//所有线程的入口函数
		PVOID Context,						//线程主函数的参数
		ULONG Flags);						//控制符，默认置0

* 控制符可以用来释放池中的线程或创建新线程
* 第一次调用此API时，Windows创建一个线程池，其中一个线程执行Function函数  
此线程返回后，将**返回线程池**，等待新任务
####线程优先级
#####设置优先级
线程优先级=进程优先级+线程相对优先级

	Bool SetThreadPriority(HANDLE hThread,int nPriority)
* 返回值为是否设置成功
#####处理器亲和
* 当线程被调度执行时，操作系统确定线程运行在哪个处理器上
* 亲和  
线程对其所运行的处理器的有限选择
	* 如I/O密集型线程与计算密集型线程的搭配
	* 指定处理器亲和的效果必须谨慎
######API
	DWORD_PTR SetThreadAffinityMask(
		HANDLE threadHandle,
		DWORD_PTR threadAffinityMask)
* 线程的AffinityMask必须是所属进程AffinityMask的子集
* DWORD_PTR是一个01串序列，直接写十六进制常数即可  
每个bit表示一个核是否使用，希望谁使用，就置相应位为1
* 设置多个核时，由系统在这些核中进行调度和切换
###3.其它API
####GetSystemInfo()
	void GetSystemInfo(SYSTEM_INFO &systemInfo);
* 可得到系统OEM ID、核的数量、处理器类型、当前进程掩码、分页大小等等信息
##三、线程局部存储（TLS）
###1.TLS介绍
* 方便的存储线程局部数据的系统
* 可使用TLS将数据与特定的线程相关联
* 本质上仍然是全局变量
###2.Windows的TLS实现
* 定义了一个数组
* 需要使用时向其申请一个项
* 早期时，项的数量为64
* 一旦TLS被启用，此变量将使得每个线程都拥有一份
* 每个线程有其私有空间，其中的数组与公有空间的数组长度对应
* 公有空间中将相应元素置为FREE或INUSE  
INUSE表示私有空间中的数组上有值  
FREE表示私有空间上的位置没有被使用  
默认情况下，所有私有空间都没有被使用
###3.涉及的API
####TlsAlloc()
	DWORD WINAPI TlsAlloc();		//分配TLS数组中的一个空闲索引
* 每个进程都维护一个长度为`TLS_MINUMUM_AVAILABLE`的位数组
* 此位数组用于记录哪个下标被使用
* 实现时直接遍历数组寻找一个空闲项  
如果找不到一个为FREE的成员，返回`TLS_OUT_OF_INDEXES`,表示失败
####TlsSetValue()与TlsGetValue()
	BOOL WINAPI TlsSetValue{
		__in DWORD dwTlsIndex,
		__in_opt LPVOID lpTlsValue);
	
	LPVOID WINAPI TlsGetValue{__in DWORD dwTlsIndex);
* 只能改变线程自身的数组，不能设置其它线程的值
* 存储的值一般为指针
####TlsFree()
	BOOL WINAPI TlsFree(__in DWORD dwTlsIndex);	//将对应索引的TLS释放
* 直接简单地将全局的位数组置为FREE
###4.使用例子
见ppt