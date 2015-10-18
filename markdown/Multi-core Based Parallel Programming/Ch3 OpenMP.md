#OpenMP*2.5*
有底层API后，就已经可以实现并行编程；然而，很多时候串行算法已经成型，如果继续使用原有的底层API，还将面临转换和调试的问题。OpenMP正是为了解决这样的问题。
##一、OpenMP的介绍
###1.概览
* 提供线程级别的并行模型
* 基于共享内存的模型
* 本身只是提供一种规范  
具体的实现由各个系统和编译器负责实现
###2.本质
* 一套多线程的API
* 面向程序员的高层接口
* 提供一系列的编译和预处理的指导语句
* 主要提供Fortran、C、C++的多线程支持
* 以SMP的物理结构完成多线程的实现
###3.实现层次
* 编译时的指导语句
* 库函数的支持
* 环境变量的支持
* OpenMP的标准可以实现在任何编译器上  
*不同的编译器支持程度不同*
###4.历史
（略）
###5.OpenMP的目标
####标准化
* 在不同的语言和架构上都可以以相同的方式编写多核程序
####简洁有效
* 编译器的指导语句尽可能地少
####易用性
* 允许程序逐步并行化
* 使对串行程序的修改尽可能地少
####可移植性
* 多种语言
* 不同平台
###6.OpenMP编程模型
####共享内存、基于线程的并行模型
####显式并行
####Fork-Join模型
* 程序启动后是单线程
* 达到需要并行的部分（**并行区**）时，产生多个线程同时运行
* 所有线程同时执行完后互相等待，一起结束
####基于编译器指导语句
####支持嵌套并行
####动态线程的创建与销毁  
线程的数量可以由OpenMP自适应
####I/O
* OpenMP并没有指定I/O的接口，仍然按原有的方式进行读写
* 因此并行区中的读写会面临冲突的问题，需要程序员自己解决
####内存模型
###7.OpenMP的层次
* SMP的硬件结构
* 系统的线程支持与OpenMP的运行时库
* 编译器指导语句、库函数和环境变量
* 应用程序和最终用户
###8.示例代码
	#include <omp.h>
	
	void main()
	{
		#pragma omp parallel			//编译指导语句，将大括号括起的范围内做成一个并行区
		{
			int ID=omp_get_thread_num();
			printf("hello(%d)",ID);
			printf("world(%d)\n",ID);
		}
	}

编译时，需要增加参数`-fopenmp`（gcc）、`-mp`（pgi）、`/Qopenmp`（Intel）、`/openmp`（Visual Studio，或直接在项目属性中添加OpenMP支持）
####更一般的形式
	#include <omp.h>
	
	int main()
	{
		int v1,v2,v3;
		//Serial code
		#pragma omp parallel private(v1,v2) shared(v3)
		{
			//
			//Join
		}
		//Back to serial code
	}
* 大括号必须紧跟编译指导语句书写
* 语法格式是固定的
##二、创建线程
###1.Fork-Join结构
* 主线程按那些创建一组线程执行并行任务
* 并行区完全可以嵌套
	* 并行区中，主线程担任一个线程的工作
	* 子并行区中，仍有相应概念上的主线程
###2.指定线程的个数
虽然线程个数可以由OpenMP自动指定，但是也可以手动设置

	omp_set_num_threads(4);

这使得此函数之后的每个并行区都是4个线程同时运行

也可以使用指导语句，这样只对一个并行区生效

	#pragma omp parallel num_threads(4)
##三、同步方式
###1.临界区
多线程同时只能由一个进入临界区执行

	float res;
	
	#pragma omp parallel
	{
		float B;
		int i,id,nthrds;
		id=omp_get_thread_num();		//当前线程的ID
		nthrds=omp_get_num_threads();	//当前的线程个数
		for(i=id,i<niters;i+=thrds)		//巧妙的for循环，尽可能将循环任务平均地分配到各线程中去
		{
			B=big_job(i);
			#pragma omp critical
				consume(B,res);
		}
	}
###2.原子操作
原子操作不会被多线程打断  
*然而原子操作和临界区的功能是一样的，因为有复合语句的存在，原子操作的功能实际上还要弱一些*  
原子操作中不能使用复合语句，也不能进行函数调用

	#pragma omp parallel
	{
		double tmp,B;
		B=DOIT();
		tmp=big_ugly(B);
		#pragma omp atomic
			X+=tmp;
	}
* 提供原子操作的意义在于**效率**  
使用原子操作的效率，比使用临界区要高很多，因为可以调用一些系统底层的特殊功能来实现原子操作
###3.路障同步
###4.同步次序
###5.flush
###6.锁
##四、并行循环
###1.SPMD与worksharing
* 工作共享创建了一个Single Program Multiple Data的程序结构
* 使得多个线程以看起来一样的代码完成不同的工作
###2.分配循环用的worksharing
	#pragma omp for
		for(i=0;i<N;i++)
		{
			something();
		}
* i将自动地成为每个线程的私有变量
* 默认得到{0,1,2,3},{4,5,6,7},...这样的循环划分方法  
可以调整，但无法任意划分
###3.worksharing的结构特点
* worksharing结构**不会**创建线程  
仅仅对执行做分配
* worksharing结构在入口没有路障同步，但出口处有  
而且都是隐式的
###4.worksharing结构的限制
* 必须放在并行区内
* 待分配的任务无法执行一部分，要么整个分配，要么不分配
* 分配时有固定的次序，不支持自定义的次序  
也不会随机分配
###5.worksharing结构的类型
* section可以进行手动分配
* single可以分配给单个线程
###6.parallel与worksharing的组合
	double res[MAX];
	int i;
	#pragma omp parallel for
		for(i=0;i<MAX;i++)
			res[i]=huge();

###7.规约
* OpenMP提供的特殊、常见数据类型的支持
####编译指导语句的基本格式
	#pragma omp directive-name [clause,...] newline
####规约指导语句
	reduction(op:list)
####归约操作的操作符和初始值
* 由OpenMP规定
* 无法自行定义
##五、同步
###1.Barrier
	#pragma omp barrier				//手动的路障同步
	#pragma omp for nowait			//指明取消末尾的隐式路障同步

* 直到所有线程执行到此位置才继续执行
* 离开临界区时有隐式的路障同步
###2.Master结构
* 标记一个代码块只被一个线程执行
* 其它线程简单跳过
* 默认没有路障同步，需要显式指定
###3.Single结构
* 此结构中的内容只有一个线程执行
* 可能由任何一个线程执行，未必是master线程
* 出口处有隐式的路障同步
###4.ordered
* 只加在for循环后
* 表明for循环存在次序依赖  
标记出的语句将按照for循环的**串行迭代序**被执行
* 对性能将产生很大的影响
###5.锁
####简单锁
可以认为是简单的布尔变量  
omp\_*\_lock

* init
* set
* unset
* test
* destroy
####嵌套锁
与简单锁不同，可以被同一个进程反复地加锁，解锁时也要进行相应数量的解锁  
omp\_*\_nest\_lock

* init
* set
* unset
* test
* destroy
####简单锁的例子
	#include <omp.h>
	omp_lock_t lock;
	omp_init_lock(&lck);
	
	#pragma omp parallel private(tmp,id)
	{
		id=omp_get_thread_num();
		tmp=do_lots_of_work(id);
		omp_set_lock(&lock);
		omp_unset_lock(&lock);
	}
	omp_destroy_lock(&lock);
##六、OpenMP的库函数
###1.修改、设置线程数量
* `omp_set_num_threads(int)`
* `omp_get_num_threads()`  
获取此韩式调用时的线程数量
* `omp_get_thread_num()`  
获取当前线程的线程号
* `omp_get_max_threads()`  
获取**下一个开辟的并行区**每个线程要开启的线程数
###2.是否在并行区域内
* `omp_in_parallel()`
###3.是否允许系统动态调整线程数量
* `omp_set_dynamic(int)`
* `omp_get_dynamic()`
###4.系统处理器数量
* `omp_num_procs()`
###5.环境变量
环境变量的优先级比库函数要低一些

* OMP\_NUM\_THREADS
* OMP\_SCHEDULE  
设置for循环是横切或竖切
##七、数据环境
###1.默认存储属性
* 共享内存的编程模型
* 全局变量在线程间共享
* 静态变量是共享的
* 堆内存是共享的  
动态分配的内存
####默认情况下的私有变量
* 并行区内定义的变量
###2.private子句
* 为变量创建每个线程一份的副本
* 未经初始化的变量，在OpenMP中的初始值未被定义  
主流平台上，private变量的修改对外围没有改变
* 外部变量作为私有变量，对定义为私有变量的变量的修改，修改谁并没有明确的定义  
*实际平台上的主流编译器都修改全局变量*
###3.firstprivate与lastprivate子句
* 和private子句几乎相同
* firstprivate  
私有变量的初值定义为全局变量原先的值
* lastprivate  
出并行区时，全局变量的值将被改变  
通常执行的最后一条更新的值反映到全局变量中
###4.default子句
	default(PRIVATE|SHARED|NONE)
* `default(SHARED)`是默认存在的，因此不需写出来  
`#pragma omp task`除外
* 在C中，`default(PRIVATE)`不被支持
* `default(NONE)`将不为变量设定默认值  
此时必须为每个变量**显式**指定属性  
*良好的自虐的编程实践~*  
通常只在需要编译器提醒哪个变量没有指定属性时才使用
###5.threadprivate子句
	int counter=0;
	#pragma omp threadprivate(counter)

* 定义为`threadprivate`的变量是可以穿越多个并行区的  
变量的值以线程号一一对应
####copyin子句
	int a=100;
	#pragma omp threadprivate copyin(a)

* 可以将全局变量的值拷贝进对应的私有变量
####copyprivate子句
* 只能在single中使用
* 在路障同步点处由执行single的线程拷贝到所有其它线程
####指针的传递
* 在线程之间，指针不要随便乱传

		#pragma omp parallel private(x) shared(p0,p1)
		x=...;
		p0=&x;
	在另一个线程中使用p0指针会造成不可预料的后果
##八、Schedule子句
###1.section子句
	#pragma omp parallel
	{
		#pragma omp sections
		{
			#pragma omp section
			calculation1();
			#pragma omp section
			calculation2();
			#pragma omp section
			calculation3();
		}
	}
* 这些任务由系统自由分配给不同线程运行
* 任务数与线程数相等时，分配显然
* 任务数多于线程数时  
先用任务把线程占满，哪个线程执行完在分配剩下的任务
* 任务数少于线程数时  
其它线程等待
###2.schedule子句
	schedule(mode[,chunk])

*实际上大多数编译器除了static，另外三种都没实现*
####静态调度
* 所有分配方式在编码时写死
* 默认的分配方式
* chunk默认为最大值（迭代数/线程数）  
chunk是循环任务分块的大小  
如果需要循环纵切，**chunk设置为1**即可
* 静态调度的分配方式是非常明确的，第一个chunk给线程0，以此类推
####动态调度
* 每个chunk可以动态分配给某个线程了
####guided调度
* chunk定义的是块的最小值
* 实际上可以更大
####runtime调度
* 全部参数交由编译器决定
##九、内存模型
###1.弱一致性
* 在代码中，读写顺序在不改变语义的情况下是可以改变的 
* 以S表示**数据**同步操作  
OpenMP中保证，S->W、S->R、R->S、W->S、S->S  
在OpenMP中就是flush操作
###2.flush
	a=...;
	<other computaion>
	#pragma omp flush(a)
* 变量值在内存中的改变最早发生在写操作，最晚在数据同步操作时进行
####隐式数据同步
其它所有同步都会自动带上数据同步
##十、OpenMP 3.0与任务
###1.任务
* 其它结构的工作量都是静态的，但task的任务是可以动态分的
###2.例子
	for(int i=0;i<N;i+=a[i])
		task(a[i]);

* 此循环不能使用`#pragma omp for`
* 想要并行就必须使用task
###3.task的结构
	#pragma omp task [clause[[,],clause]...]
* 子句可以加入`if`、`untitled`与所有数据环境
####并行的链表举例
	#pragma omp parallel
	{
		#pragma omp single private(p)	//由一个线程进行预处理，其它线程什么都不做
		{
			p=listhead;
			while(p)
			{
				#pragma omp task
				process(p);				//将链表内多个结点的处理并行进行，
										//占用并行区内原本闲置的线程
				p=next(p);
			}
		}
	}
###4.untied子句
* 创建的任务，默认将会与某个线程绑定，只能由某个线程来完成
* `untied`可以用来解除这样的绑定
####举例
	#pragma omp single
	{
		#pragma omp task untied
		for(i=0;i<ONEZILLION;i++)
			#pragma omp task
			process(item[i]);
	}
* 如果不作为united的任务，源源不断的新任务将撑爆内存
* untied允许任务的创建在其它线程间迁移
###5.if子句
* 如果表达式为`false`，整个编译指导语句无效
* 默认为`if(true)`
