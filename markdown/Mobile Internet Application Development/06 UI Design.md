#06 UI Design
##一、概览
Creative Vision
###1.Enchant Me
* 在一些细节上产生细微的变化效果
* 给予实际的操纵感
* 提供设置的空间
* 使用户认为系统了解用户的使用习惯
###2.Simplify My Life
* 提示语尽量简洁明了
* 多用图片
* 给用户预先做一些决定  
但要提供最后的选择权
* 一些元素在想要的时候才显示出来
* 令用户轻松地跳转与导航
* 用户的设置信息做同步以免丢失
* 外观与表现都要统一风格
* 除非紧急，不要打断用户操作
###3.Make Me Amazing
* 给用户无处不在的小惊喜
* 出问题之后不要责备用户  
除了错误以外，更要提供解决方案
* 显现出一个任务的多个步骤
* 提供快捷键功能
* 最重要的东西永远放在最快的地方
###4.UI Architecture
####主屏、应用列表、最近应用
####System Bars
####通知
####通常的App UI
* Action Bar
* View Control
* Navigation Drawer
* Content Aera
####应用结构
* Top level views
* Category views
* Detail/edit views
####手势
* 点击
* 长按
* 拖动
* 双击
* 放大/缩小
####Back与Up button
* Back反映时序意义
* Up反映逻辑意义
##二、Building Blocks
###1.Tabs
* 可固定的
* 可以是栈式的
###2.Lists
* 要为每个Section做一个划分
* Grid list
###3.Scrolling
###4.Spinner
* 作为保险色，黑、白、灰的使用会比较稳妥
* 主题也定义了Spinner在不同状态下的显示形式
###5.Button
* 仅图标
* 仅文字
* 图标带文字
* 有无立体感
###6.无框按钮
* 一个图标没有任何边框，与图标就没有区别
* 因而“无框按钮”并不是完全没有框
* 按钮与其它元素之间会有些比较小的分隔
###7.Text fields
###8.Text selection
###9.Seek bar与Slider
常用于对数值进行连续的改变
###10.进度条
###11.Activity indicater
* 转圈时不要再显示文字
###12.可定制indicator
###13.开关
###14.对话框
* 标题
* 内容区域
* 选择操作
###15.警告
###16.Pop-up
###17.Toast
###18.Picker
* 可以嵌入对话框中使用
##三、风格
###1.设备与展示
###2.主题
* 使用近似色的设计思想
* 要有触摸反馈
###3.度量与边界
每行内容占用48dp

* 图标32*32
* 上下留出8dp空白
* 文字占中间16dp
* 内容与屏幕左右边界空出16dp
###4.字体
* 字体粗细  
庄重肃穆的场合使用粗字体
* 字体等宽与否
* 字体的上下
* 字体的常规大小
####使用自定义字体
* 放进fonts目录
* 通过代码设定即可
###5.颜色
Google提供的一系列配色方案
###6.图标
####启动器图标
* 48*48dp
* 需要有512*512px大小
* 图标统一将边缘抠掉
* 带一定的3D感觉
####Action Bar
* 颜色
* 大小
* 图标风格
####小图标
* 颜色
* 大小
* 风格
####通知栏图标
* 颜色
* 大小
* 风格
###7.写作风格
* 文字要短
* 友好提示
* 最关键的内容放在最前面
* 只描述必要内容  
需要突出数字时，使用阿拉伯数字而不是文字形式
##四、模式
###1.Action Bar
* 一定有应用图标
* 箭头可有可无
* 带有View Control
	* 固定式
	* 滑动式
* 带一些操作按钮
* 最后是其它菜单项（Drawer）
* 根据屏幕方向做适配
* 可以考虑将其分裂为上下两个
* 上下文的Action Bar
* Action button遵循**FIT原则**
	* Frequent
	* Important
	* Typical

导航和动作一定要区分开来
###2.Navigation Drawer
* 抽屉以导航来完成目标  
通常在深层次目录使用导航  
实现低层次目录的**交叉**导航，有效展现应用的逻辑结构
* 取消Drawer的四种方式
* 抽屉在标题前有三条横线做提示  
必要时在初次进入应用时给出帮助
* 上下文Action Bar在抽屉打开时要隐藏
* 给用户一些拉动的反馈
* 通过抽屉进行导航时，如果进行了返回操作，应该**回退到层次结构上层或主屏**
* 抽屉也有其固定的设计风格
###3.横屏/竖屏的切换
* 内容的拉伸与压缩
* Stack模式  
改变栈的生长方向
* 内容的扩展与分解
* 显示/隐藏
###4.Swipe View
* 在细节View上的滑动  
在前后的细节中切换
* 在Tab上的滑动
###5.Full Screen
####回到原有状况的方式
* 点击屏幕
* 边缘拉动
###6.确认与通知
###7.通知栏
* 通知栏的基本元素
* 可以直接展开
* 可以直接放置Action
* 同一应用不宜同时显示太多通知  
需要进行梳理，同时可以结合展开的功能
#####显示通知的时机
* 事件是时间敏感的
* 活动可能牵涉到另外一个人
####不要显示通知的时机
通知不能打断用户当前的事务
###8.Widget
* 类型
	* 信息展现
	* 集合展现
	* 控制
	* 混合类型
* 只支持触摸和滑动手势
* 要能够改变Widget的大小
###9.设置
###10.帮助文字
* 表达简洁
* 使用户知道怎么办  
*而不是技术细节*  
提供的措施用户要能办到
* 对一些简单操作应该默认用户已知
###11.向后兼容
###12.可访问性
###13.Pure Android
* 不要模仿其它平台的UI风格

Pure Android是否好