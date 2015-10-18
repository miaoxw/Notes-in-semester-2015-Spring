#Hello Tizen
##一、开发过程
总体与Android类似
##二、环境配置——Tizen SDK
* https://developer.tizen.org/zh-hans/...
* 安装可配置  
SDK的下载很慢，可以考虑使用光盘源安装
##三、工程创建
###1.本地应用
* inc目录  
保存各种头文件
* src目录
####设置项目属性
通过xml文件
####基于EFL的GUI
* 本地的图形库  
继承自X-Window
####构建与运行
* 本地应用打包成tpk文件
###2.Web应用
实际上，是将Web应用打包成.wgt文件，再安装到手机上使用
