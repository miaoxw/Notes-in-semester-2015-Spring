#05 Activity and UI
##一、Activities
###1.创建Activities
* 创建Activity的子类
* 实现各类回调方法  
在状态转换时调用的方法
####onCreate()
* 在创建活动时被调用
* 做一些初始化的工作
####onPause()
* 在应用失去焦点时调用
* 调用onPause方法并不代表活动不可见
* 通常保存当前状态以便后续恢复
###2.实现UI
####View与ViewGroup
* View->Widgets
* ViewGroup->Layout
####startActivity()
	Intent intent=new Intent(this,SignInSctivity.class);
	startActivity(intent);

* 也可以在启动其它应用中的activity
* 可以在创建的activity中要求返回结果
	* 通过回调方法onActivityResult()通知运行的结果
###3.Activity的生命周期
* 由不同的回调函数进行管理
* 与构想的手机使用场景密切相关
####嵌套循环
详情见ppt

* 前景生命周期  
onResume()/onPause()
* 可见生命周期  
加上onRestart()/onStart/onStop()
* 完全生命周期  
再加上onCreate()/onDestroy()
####Activity的状态保存
* onSaveInstanceState()回调方法
###4.Bundle
* 可以进行数据的储存
* 以**键值对**的方式进行数据的保存
###5.重叠的Activity
##二、UI
###1.使用View
* 设置属性
* 设置焦点
* 配置监听器
###2.布局
####ID
* +  
意味着一个新的ID
* @  
意味着字符串的开始
####Layout
* LinearLayout
	* 在屏幕上按顺序流式排布
	* Gravity
* RelativeLayout  
以其它View的相对位置决定新组件的位置
####Adapter
* ArrayAdapter
* SimpleCursorAdapter
#####nofityDataSetChanged()
当数据发生更改后，调用此函数跟着数据更改View
###3.事件响应
####方法1
生成一个匿名类
####方法2
继承Listener显示定义
####方法3
将对事件的响应写在XML文件中
###4.Loader
异步读取数据的Activity
###5.GridView的示例