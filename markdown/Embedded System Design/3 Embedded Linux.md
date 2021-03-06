#3 Embedded Linux
##二、嵌入式Linux开发流程
* 建立开发环境
* 配置开发主机
* 建立bootloader
* 下载Linux操作系统
* 建立根文件系统
* 建立应用程序的Flash磁盘分区
* 开发应用程序
* 烧写内核、根文件系统、应用程序
* 发布产品
###1.嵌入式Linux开发模式
####建立交叉开发环境
* 有开发和商业两种类型
* 典型的是GNU工具链
* 在不同阶段可以使用的目标机与宿主机的连接方式不同
####交叉编译和链接
####交叉调试
* 硬件调试  
使CPU在内部直接实现调试功能
* 软件调试
* 应用软件的调试  
本地调试/远程调试

通常逻辑功能先在宿主机上调试完成，再在主机上进行实际的功能调试
####系统测试
* 内存分析
* 性能分析
* 覆盖分析
* 缺陷跟踪
###2.主机环境
####开发主机的能力
* 交叉工具链和程序库
	* gcc compiler
	* binutil
	* C Library
	* C Header
* 目标系统软件包  
包括程序、工具和程序库
* 主机工具
	* vim
	* make
* 为目标板提供服务的服务器
	* DHCP
	* TFTP
	* NFS
####Linux下的编辑器vi
*在vi下达到生存的水平即可，不用特别关注*
####嵌入式开发工具
* 编译开发工具：arm-linux-gcc
* 调试工具：gdb
* 软件工程工具：make、cvs
###3.嵌入式Linux系统
####引导加载程序
* 两种加载模式
* 烧写  
可以使用超级终端的传送->发送文件  
Linux下有minicom  
**SecureCRT**
####Linux内核
* 内核和文件系统下载  
	* USB方式（使用DNW）
	* 网络方式（主机建立TFTP服务器，宿主机作为服务器下载相应内核）  
	需要使用BOOTP或DHCP协议  
	要了解无盘工作站的相关工作原理
* 应用程序和文件传输
	* 通常选择ftp进行传输  
	在上位机装ftp服务器软件
	* NFS服务
		* 跟文件系统的大小不会受限于板卡有限的资源
		* 开发过程中，对应用程序文件所做的改动立刻体现在目标系统中
		* 可以在开发调试跟文件系统之前调试和引导内核
* 使用虚拟机开发环境
	* 注意检查串口的相关配置
	* 网络模式的选用  
	桥接模式和NAT
####文件系统
####用户应用