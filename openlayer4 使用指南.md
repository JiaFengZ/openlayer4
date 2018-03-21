openlayer 是一个开源的的js地图引擎，提供了一系列APT实现了web端的在线地图的渲染展示和交互操作。这里不打算过多涉及其底层原理，本指南主要是根据个人的学习和开发经验，对其API的使用和一些常见功能实现的研究总结。

# 1 openlayer 的基本概念和几个主要的类

首先有几个关键的概念：Map、View、Layers、Controls、Interaction、Sources。Map（地图）是由 layers（图层）组成的，通过 view（视图）显示，允许 interaction（交互）对地图的内容修改，并且提供一些UI组件实现对地图的控制（controls）。View 管理着地图的视觉参数，比如分辨率、旋转角度、定位中心、显示范围等等，openlayer提供了ol.View类负责相关操作。Layers是组成地图的关键元素，ol.layer 基类下扩展若干子类负责管理图层，Map 可以有多个 layer 叠加组成，ol.layer 类能够实现图层添加、删除、更改层叠顺序等操作。更进一步来说，Layers 是地图数据源的容器，Sources 是 Layers 图层的数据来源，Sources负责管理各种格式的数据源，比如有 Tile source（瓦片）、Image source（图片）、Vector source（矢量）等格式的数据源。Map、Layers、Source的关系可以简单看作：Source → Layers → Map。   

openlayer 的结构就是由上面提到的几个关键概念为骨架组建的，其主要的几个类和命名空间如下：
ol 原始基类  
ol.Map 地图类  
ol.View 管理视觉参数的类  
ol.source 负责数据源管理的命名空间，其下扩展很多类，其中就有ol.source.Source  
ol.layer 负责图层管理的命名空间，其下扩展有ol.layer.Layers  
ol.interaction 管理地图交互，其下扩展有各种的交互事件类型，比如DoubleClickZoom(双击放大)、DragAndDrop(拖拽移动)、Draw（绘制）等等  
ol.control 管理UI控件，其下扩展有 FullScreen(全屏)/MousePositio(鼠标位置)/OverviewMap(鹰眼)/Rotate(旋转)/Zoom(缩放)/ScaleLine/ZoomSlider(滑动缩放)/ZoomToExtent(缩放控制范围)等预定义好的UI控件，方便用户对地图进行操作控制。另外 ol.control.Control允许自定义控件。  
ol.Feature 负责管理地图元素 features，features是指带有几何形状(geometry)和属性(attribute)的矢量数据，可以看作比 sources 更底层的组成元素。  

除了以上几个核心的类，还有几个重要的工具类：  
ol.style 管理地图元素(features)的渲染样式  
ol.coordinate 封装了坐标相关的操作，比如说坐标变换  
ol.events 事件，ol.events.Event实现了各种事件接口，其下又有若干对应不同触发对象的事件类，比如ol.MapEvent就是负责map触发的事件，ol.Map中就有定义能够触发的事件类型。  

## 1.1 map
## 1.2 layer
## 1.3 source
## 1.4 feature
# 2 layer 图层的相关操作
## 2.1 layer 类的属性和方法
## 2.2 图层层级的管理
## 2.3 设置图层的透明度
## 2.4 设置图层的可见性
## 2.5 设置图层的显示范围
## 2.6 监听图层属性变化事件
# 3 source 的相关操作
## 3.1 source 的属性和方法
# 4 feature 的相关操作
# 5 geometrory 的相关操作
# 6 ol.Style 类设置元素样式
# 7 视图 view 的相关操作
# 8 control 控件
# 9 事件
# 10 使用 ol.interation.draw 进行绘制
# 11 overlay 的使用
# 12 一些应用的例子