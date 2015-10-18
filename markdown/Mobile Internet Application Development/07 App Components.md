#07 App Components
##一、Intent
* 四个核心构件中有三个依靠intent激活
* 组件间延迟的运行时绑定
###1.发送Intent
###2.内容
* Component name
* Action
	* CATEGORY_CALL
	* EDIT
	* MAIN
	* SYNC
	* VATTERY_LOW
	* HEADSET_PLUG
	* SCREEN_ON
	* TIMEZONE_CHANGED
	* ……
* Data
* Category
* Extras  
存放一些键值对
* Flag
###3.Intent的解析
####Intent Filters
* 通过filter解析哪些intent与当前应用有关
* 应用中进行Intent matching
###4.管理Activity的生命周期
##二、设计有效的导航
* 从需求分析出发
* 多尺寸布局
* 集合相关与
* 基于按钮的导航
* 基于集合的导航
* 基于Tab的导航
	* Tabs
	* Tickmarks
	* Labels
* 要支持返回的临时导航
##三、实现有效的导航
##四、魅族的Flyme悬浮球
* 系统级别、应用间的导航