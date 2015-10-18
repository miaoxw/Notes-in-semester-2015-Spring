#04 Application Fundamentals
##一、Android架构
* 蓝色——Java程序
* 绿色——C/C++库
* 黄色——虚拟机
* 红色——内核
###1.Linux内核
* 虽然建立在Linux上，但Android本身不是Linux
* 没有原生的界面系统
* 没有glibc的支持
* 提供操作系统的必要、基本服务  
进程、内存、电源管理、网络、驱动模型、安全……
####使用Linux内核的原因
* Linux本身已被证明适用于各种平台和应用
* Linux可提供硬件抽象层，使上层屏蔽硬件改变后的细节
* 大多数设备都容易找到Linux驱动，不用重复编写
* 一整套成熟的开发步骤可以复用
####Android的特殊设备
* Ashmem  
共享内存的分配
* Binder  
本身是RPC的机制  
主设备号为10
* Framebuffer
* PM Solution  
电源管理对手机非常重要  
通过电源管理策略，以及**wake lock**机制
###2.硬件抽象层
* 用户层的C/C++库
* 定义整个Android的接口
* 通过抽象，将软硬件接口分离
* 由于Linux内核遵循GPL协议，完全公开的代码可能产生一些代价  
对厂商提供一定的保护
###3.原生C库
####Bionic libc
* 为嵌入式设备做了定制
* 使用BSD协议
* 代码体积更小，做了些优化
####功能库
#####Webkit
* 浏览器的核心
* 开源，且质量尚可
#####Media Framework
#####SQLite
#####Surface Flinger
* 将各应用的层进行合并
* 各个surface可2D可3D
##二、应用
###1.编译与解释
###2.广义解析器
* 词法分析
* 生成基于栈的字节码  
JVM使用，操作与硬件无关，完全是逻辑层面的东西
###3.dex的生成
* Android使用DVM
* JDK编译出.class文件
* Android提供的DX工具转换为.dex文件
####Dalvik字节码
* 基于寄存器的架构
* 编译的指令数目减少，执行效率要高一些
###4.Android的进程描述
* 每个应用认为是一个不同的用户
* 每个应用运行在单独的虚拟机上
* 系统不再需要此进程或内存不够时，将此进程杀死
####Zygote
* 由系统启动时的init进程产生
	* 完成DVM初始化
	* 完成库的加载
	* 预置类库的加载和初始化操作
* 运行新Android应用时，通过`fork`来共享内存
* 负责后续框架层的其它进程的创建和启动工作
* Zygote首先创建SystemServer进程  
负责启动系统关键服务，各种*ManagerService
* 需要启动Android应用时，ActivityManagerService通过socket通信通知Zygote`fork`出新的进程
####最小权限原则
* Android系统默认使用
* 默认情况下什么权限都没有
* 需要权限，需要向系统申请
###5.应用组件
####Activity
* UI组件
* 通常对应一屏的UI  
从而可以方便地管理应用
####Service
不需要界面运行的任务

* 做一些耗时的运算或操作  
避免在主线程上进行耗时的操作导致UI无法响应
* 其它组件可以使其开始运行或与其绑定
####Broadcast Receiver
负责响应通知和状态的改变

* 处理系统的广播
####Content Provider
用于应用（进程间）的数据共享
####Activating Methods
* startActivity()/startActivityForResult()  
需要返回结果时使用后者，此时附带一个Intent
* startService()/bindService()
* sendCroadcast()/sendOrderedCroadcast()/sendStickyBroadcast()
* query()
####manifest
* 包含对应用的所有配置
* 所有组件都要在manifest中声明  
`<activyty>`/`<service>`/`<receiver>`/`<provider>`
* Intent Filter  
帮助系统定位需要的活动
####Activity与Task
* 为所有activity设定了栈结构进行存储
* 不同的task使用不同的栈存储