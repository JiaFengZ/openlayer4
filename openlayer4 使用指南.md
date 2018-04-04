	openlayer 是一个开源的的js地图引擎，提供了一系列APT实现了web端的在线地图的渲染展示和交互操作。  
	这里不打算过多涉及其底层原理，本指南主要是根据个人的学习和开发经验，对其API的使用和一些常见功能实现的研究总结。

# 1 openlayer 概览

首先有几个关键的概念：`Map`、`View`、`Layers`、`Controls`、`Interaction`、`Sources`。  
`Map`（地图）是由 `layers`（图层）组成的，通过 `view`（视图）显示，允许 `interaction`（交互）对地图的内容修改，并且提供一些UI组件实现对地图的控制（controls）。`View `管理着地图的视觉参数，比如分辨率、旋转角度、定位中心、显示范围等等，openlayer提供了`ol.View`类负责相关操作。`Layers`是组成地图的关键元素，`ol.layer` 基类下扩展若干子类负责管理图层，`Map` 可以有多个 layer 叠加组成，`ol.layer `类能够实现图层添加、删除、更改层叠顺序等操作。更进一步来说，`Layers` 是地图数据源的容器，`Sources` 是 `Layers `图层的数据来源，`Sources`负责管理各种格式的数据源，比如有 `Tile source`（瓦片）、`Image source`（图片）、`Vector source`（矢量）等格式的数据源。`Map`、`Layers`、`Source`的关系可以简单看作：`Source → Layers → Map`。   

openlayer 的结构就是由上面提到的几个关键概念为骨架组建的，其主要的几个类和命名空间如下：   
	`ol` 原始基类  
	`ol.Map` 地图类  
	`ol.View` 管理视觉参数的类  
	`ol.source` 负责数据源管理的命名空间，其下扩展很多类，其中就有`ol.source.Source`  
	`ol.layer` 负责图层管理的命名空间，其下扩展有`ol.layer.Layers`  
	`ol.interaction` 管理地图交互，其下扩展有各种的交互事件类型，比如`DoubleClickZoom`(双击放大)、`DragAndDrop`(拖拽移动)、`Draw`（绘制）等等  
	`ol.control` 管理UI控件，其下扩展有 `FullScreen`(全屏)/`MousePositio`(鼠标位置)/`OverviewMap`(鹰眼)/`Rotate`(旋转)/`Zoom`(缩放)/`ScaleLine`/`ZoomSlider`(滑动缩放)/`ZoomToExtent`(缩放控制范围)等预定义好的UI控件，方便用户对地图进行操作控制。另外 `ol.control.Control`允许自定义控件。  
	`ol.Feature` 负责管理地图元素 `features`，`features`是指带有几何形状(`geometry`)和属性(`attribute`)的矢量数据，可以看作比 `sources` 更底层的组成元素。  

除了以上几个核心的类，还有几个重要的工具类:   
	`ol.style` 管理地图元素(`features`)的渲染样式    
	`ol.coordinate` 封装了坐标相关的操作，比如说坐标变换   
	`ol.events` 事件，`ol.events.Event`实现了各种事件接口，其下又有若干对应不同触发对象的事件类，比如`ol.MapEvent`就是负责`map`触发的事件，`ol.Map`中就有定义能够触发的事件类型。  

# 2 ol.Map([demo](https://jiafengz.github.io/openlayer4/demo/olMap/baseMap.html))
`map` 是 `openlayer` 的核心，`map` 是渲染视图、图层的容器，如下构造一个 `map` 地图容器：
```javasctipt
var map = ol.Map({
	view: new ol.View({
		center: [0, 0],
		zoom: 1
	}),
	layers: [{
		new ol.layer.Tile({
			source: new ol.source.OSM
		})
	}],
	target: 'map'
});
```
ol.Map 是地图 map 的构造函数，几个关键的参数值如下：   
new ol.Map(options)  
options.controls 初始添加到地图的UI控制组件(就是上文提及的全屏、鹰眼、缩放等控制按钮)，默认是 ol.control.defaults()  
options.pixelRatio 设备像素比，默认是取值 window.devicePixelRatio  
options.interactions 初始添加到地图的交互行为，默认是 ol.interation.defaults()  
options.layers 图层  
options.renderer 使用的渲染器，可选canvas、WebGl  
options.target 渲染地图的目标容器，是一个元素或者元素的id  
options.view 设置地图视图参数，使用new ol.View()构造  

上面忽略了一些其他参数，可在其API文档查看完整的参数列表。

## 2.1 map 绑定的事件
`click` 点击事件  
`dblclick` 双击事件  
`moveend` map 开始移动时触发  
`movestart` map 移动结束后触发  
`pointerdrag` 鼠标拖拽  
`pointermove` 鼠标移动  
`postrender` 地图渲染后触发  
`propertychange` 地图某个属性变化时触发  
`singleclick` 单击，click后延迟250ms触发，确保不是双击事件

## 2.2 map 对象的方法
写方法：  
* addControl(new ol.control.Control(...)) 添加控件  
* removeControl() 移除控件  
* addInteraction(new ol.interaction.Interaction(...)) 添加交互行为   
* removeInteraction() 移除交互行为  
* addLayer(layer) 添加图层  
* removeLayer() 移除图层  
* addOverlay(new ol.Overlay(...)) 添加悬浮元素  
* removeOverlay() 移除悬浮元素  
* dispatchEvent(event) 手动触发事件，将会调用所有监听了该类型事件的监听回调函数  

读方法：  
* getControls()  
* getInteractions()  
* getLayerGroup()  
* getView()  
* getLayers()  
* getOverlayById()  
* getOverlays()  
* getCoordinateFromPixel()  
* getEventCoordinate()  
* getEventPixel()  
* getFeaturesAtPixel()  
* getPixelFromCoordinate()  

* getRevision()  
* getSize()  
* getTarget()  
* getTargetElement()  
* getViewport()  


绑定事件的方法：  
* on(type, listener, opt_this) 绑定事件  
* un(type, listener, opt_this) 解绑事件

其他方法：  
* forEachFeatureAtPixel(pixel, callback, opt_options) 遍历特定像素点位置的 `features` 元素并执行回调函数  
* forEachLayerAtPixel(pixel, callback) 遍历特定像素点位置的 `layers` 图层并执行回调  
* hasFeatureAtPixel() 判断特定像素点位置是否包含 `features`  
* updateSize() 重新计算地图视口大小(viewport)  

# 3 layer([demo](https://jiafengz.github.io/openlayer4/demo/olLayer/map.html))
`ol.layer.Layer`可以创建图层，其下的子类 `ol.layer.Image` `ol.layer.Tile` `ol.layer.Vector`可创建不同格式数据源的图层。  
首先看看 `ol.layer.Layer` 的构造方法：
`new ol.layer.Layer(options)`:
options.opacity 设置图层透明度，默认是1不透明  
options.visible 设置图层的可见性  
options.extent 显示的图层的边界范围  
options.zIndex 图层的层级，有多个图层叠加时，通过zIndex管理  
options.minResolution 图层最小分辨率  
options.maxResolution 图层最大分辨率  

## 3.1 `layer` 对象具有的方法
读方法：  
* getExtent() 获取图层边界范围
* getMaxResolution() 
* getMinResolution()
* getOpacity() 获取图层透明度
* getSource() 获取图层 source 数据源
* getZIndex() 获取图层层级

写方法：  
* setExtent() 设置图层边界显示范围
* setMap() 设置图层附属的map对象
* setMaxResolution()
* setMinResolution()
* setSource() 设置图层的数据源source
* setOpacity() 设置透明度
* setVisible() 设置可见性
* setZIndex() 设置层级
## 3.2 事件
* once(type, listener, opt_this) 绑定一次性事件
* on(type, listener, opt_this) 绑定事件
* un(type, listener, opt_this) 解绑事件

## 3.3 一个创建矢量数据图层(Vector Layer)的例子
```javascript
//创建一个Vector Layer,使用 ol.source.Vector类型的数据源构造图层
var vectorLayer = new ol.layer.Vector({
    source: new ol.source.Vector({
      url: 'https://openlayers.org/en/v4.6.5/examples/data/geojson/countries.geojson',
      format: new ol.format.GeoJSON()
    }),
    style: function(feature) {  //使用回调函数构造返回style
      var style = new ol.style.Style({
        fill: new ol.style.Fill({
          color: 'rgba(255, 255, 255, 0.6)'
        }),
        stroke: new ol.style.Stroke({
          color: '#319FD3',
          width: 1
        }),
        text: new ol.style.Text({
          font: '12px Calibri,sans-serif',
          fill: new ol.style.Fill({
            color: '#000'
          }),
          stroke: new ol.style.Stroke({
            color: '#fff',
            width: 3
          })
        })
      });
      style.getText().setText(feature.get('name')); //从source数据源中的feature(json数据)中读取name属性
      return style;
    }
  });
```
可以看到这个 vector 图层的数据源是通过 `https://openlayers.org/en/v4.6.5/examples/data/geojson/countries.geojson`获取的，实际上请求得到的是json数据，在控制台抓包其响应数据如图：  
![json数据](/images/vector-json.png)

## 3.4 管理图层的属性
  openlayer 提供了API获取和设置图层的显示范围、可见性、透明度、叠加层级等属性。
  ```javascript
  //创建图层
  function createLayer(color, coordinate) {
	//创建一个 feature 地图点元素
	var feature = new ol.Feature(new ol.geom.Point(coordinate));
	//设置feature的样式，绘制成一个矩形
	feature.setStyle(new ol.style.Style({
        image: new ol.style.RegularShape({
            fill: new ol.style.Fill({color: color}),
            stroke: new ol.style.Stroke({color: 'black', width: 1}),
            points: 3,
            radius: 80,
            rotation: Math.PI / 4,
            angle: 0
        })
    }));
    //使用feature构造地图数据源
  	var source = new ol.source.Vector({
  		features: [feature]
  	});
	//利用source构造layer
  	var vectorLayer = new ol.layer.Vector({
      source: source
    });
    return vectorLayer;
  }

	var layers = [];
	layers.push(createLayer('red', [40, 40])); //添加两个图层
	layers.push(createLayer('blue', [0, 40]));
  	var map = new ol.Map({
	  	layers: layers,
	    target: 'map',
	    view: new ol.View({
	      center: [0, 0],
	      zoom: 18
	    })
  })
	
  function getProperties(layer) {
	console.log(layer.getExtent());
	console.log(layer.getVisible());
	console.log(layer.getZIndex());
	console.log(layer.getOpacity());
  }

  

  //使用构造的vectoreLayer图层创建一个地图
  var map = new ol.Map({
    layers: [vectorLayer],
    target: 'map',
    view: new ol.View({
      center: [0, 0],
      zoom: 1
    })
  });

  layers[0].setVisible(false);
  layers[1].setOpacity(0.5);
  layers[1].setZIndex(2);
  layers[1].setExtent([10, 10]);
  getProperties(layers[0]);
  getProperties(layers[1]);

```

# 4 source([demo](https://jiafengz.github.io/openlayer4/demo/olSource/map.html))
## 4.1 `source`的几种数据格式
在深入 `source`的API使用前，有必要先分析一下不同的数据源格式，简单来说可以分为 `Image` `Vector` `Tile`三大类型的数据源，`Image`和`Tile`本质上都是图片或者图片集，`Vector`则是矢量数据源，其分别对应 GIS 的两种数据存储模型：矢量数据模型和栅格数据模型，矢量数据模型以离散的点坐标表示地理元素，通过点、线、面几何元素表达地图的空间元素，数据存储以对象作为单位，一般行政边界、街道会采用矢量数据存储模型。而栅格数据模型则是以一系列基于网格的不同颜色和灰度的像素元来表示空间元素，适用于连续空间变化的空间数据，比如遥感地图。对应于web端的二维地图就是矢量地图和瓦片地图两种形式，矢量地图就是采用矢量数据模型存储的矢量数据组织的地图，服务端返回的是点、线、面等离散的数据对象，瓦片地图则是根据缩放层级和显示范围计算获取相应图片，在web端拼接成一张完整的地图。回到`openlayer`的`Image` `Tile` `Vector` 三大类型数据源，`Image`和`Tile`返回的就是图片集，`Vector`返回的是离散的数据对象。  
在 `openlayer` 中这三者有可向下扩展细分若干子类：  
```
ol.source.Source ──├── ol.source.Image ──────├── ol.source.ImageArcGISRest
		   ├  			     |── ol.source.ImageCanvas
		   ├── ol.source.VectorTile  ├── ol.source.ImageStatic
		   ├── ol.source.TileJSON    ├── ol.source.ImageWMS
		   ├── ol.source.TileWMS     ├── ol.source.Raster
		   ├── ol.source.TileImage
		   ├── ol.source.Tile ───────├── ol.source.TileDebug
		   ├ 			     ├── ol.source.TileUTFGrid
		   └── ol.source.Vector`     ├── ol.source.UrlTile

											
```
特别值得注意的是 `ol.source.Vector` 格式的数据源所包含组织的features数据需要通过解析器进行数据解析，openlayer通过`ol.format.Feature`的子类来解析各种格式的矢量数据，比如`EsriJSON` `GeoJSON` `TopoJSON` `MVT` `Text` `XML`等格式。  
上面只是一个粗略的介绍，更过细节可以参考 openlayer的手册，要了解更多个关于地图数据模型的内容可参考专业的WebGIS书籍。

## 4.2 `ol.source.Source`基类上几个方法
* getProjection() 获取投影坐标系
* getState() 获取source加载状态:`undefined`, `loading`, `ready` ,`error`
* get(key) 获取属性值
* refresh() 刷新图层source，会触发 `change` 事件

## 4.3 `ol.source.Vector` 类型的数据加载
`new ol.source.Vector(options)`  
options.attributes ol.Attribute类型的数据，可以是string/string数组  
options.features 组成source的数据对象  
options.format 当feature通过url加载获取时，需设置format对解析数据  
options.loader 加载feature的加载器，如果没有设置loader但设置了url，则使用 XHR feature 加载器    
options.url 使用 XHR feature loader(`ol.featureloader.xhr`)加载features
上面仅仅列举了关键的几个option，完整的配置请看官网API。 
对于format加载器，不同格式的数据需要相应的解析器，例如对于`GeoJSON`类型的数据加载如下：
```javascript
var vectorSource = new ol.source.Vector({
  format: new ol.format.GeoJSON(),
  loader: function(extent, resolution, projection) {
     var proj = projection.getCode();
     var url = 'https://ahocevar.com/geoserver/wfs?service=WFS&' +
         'version=1.1.0&request=GetFeature&typename=osm:water_areas&' +
         'outputFormat=application/json&srsname=' + proj + '&' +
         'bbox=' + extent.join(',') + ',' + proj;
     var xhr = new XMLHttpRequest();
     xhr.open('GET', url);
     var onError = function() {
       vectorSource.removeLoadedExtent(extent);
     }
     xhr.onerror = onError;
     xhr.onload = function() {
       if (xhr.status == 200) {
         vectorSource.addFeatures(
             vectorSource.getFormat().readFeatures(xhr.responseText));
       } else {
         onError();
       }
     }
     xhr.send();
   },
   strategy: ol.loadingstrategy.bbox
 });
```

## 4.4 专属`ol.source.Vector`的方法和事件
source上的属性发生变化触发的事件：
* addFeature() 当source添加feature时触发
* changeFeature() 当source中的feature更新时触发
* clear() 在source上调用clear方法时触发
* removeFeature() 从source中移除feature时触发

其他扩展的专属方法：
* getExtent() 获取显示的边界范围
* getFeatureById() 通过id获取feature
* getFeatures() 获取features
* getFeatureAtCoordinate() 获取那些几何元素(geometry)包含特性坐标点的所有features
* getFeaturesInExtent() 获取一定范围的features
* getClosestFeatureToCoordinate() 获取距离坐标最近的feature
* getLoader()
* setLoader()
* removeFeature()
* addFeature()
* addFeatures()

# 5 feature
# 6 geometrory 的相关操作
# 7 ol.Style 类设置元素样式
# 8 视图 view 的相关操作
# 9 control 控件
# 10 事件
# 11 使用 ol.interation.draw 进行绘制
# 12 overlay 的使用
# 13 一些应用的例子
