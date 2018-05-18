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
options.interactions 初始添加到地图的交互行为，默认是 ol.interaction.defaults()  
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
* getControls() 获取UI控件
* getInteractions() 获取交互行为
* getLayerGroup() 获取图层集合
* getView() 获取视图
* getLayers() 或如图层
* getOverlayById() 通过id获取overlay浮层元素
* getOverlays() 获取所有overlay浮层元素
* getCoordinateFromPixel() 根据像素坐标获取在地图上的地理坐标
* getEventCoordinate() 获取事件的地理坐标
* getEventPixel() 获取事件的像素坐标
* getFeaturesAtPixel() 获取某像素坐标点的features
* getPixelFromCoordinate() 根据地理坐标获取像素坐标
* getRevision() 获取map对象的版本号，每次map对象属性的改变都会使版本号(version number)递增
* getSize() 获取地图大小
* getTargetElement() 获取地图容器元素
* getViewport() 获取视口


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
上面只是一个粗略的介绍，更多细节可以参考 openlayer的手册，要了解更多个关于地图数据模型的内容可参考专业的WebGIS书籍。

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
* getFeatureAtCoordinate() 获取那些具有包含特性坐标点的几何元素(geometry)的features
* getFeaturesInExtent() 获取一定范围的features
* getClosestFeatureToCoordinate() 获取距离坐标最近的feature
* getLoader()
* setLoader()
* removeFeature()
* addFeature()
* addFeatures()

# 5 feature
openlayer中的`feature`可以通过`ol.Feature`类构造，`feature`用于描述地图中几何元素，是包含了要渲染的几何对象以及其他属性的json对象。  
可以通过`setStyle`方法设置其渲染样式。
`new ol.Feature(options)`:  
* geometry：几何对象，`ol.gemo`下的各个子类可构造各种类型的几何对象(点、线、面)，如`ol.geom.Polygon`,`ol.geom.Point`,`ol.geom.Circle`...
* name: 名字
一般而言`feature`会关联使用`geometry`属性定义的几何对象进行渲染显示，但是允许定义其他的几何对象，通过`setGeometryName(name)`方法改变关联渲染的几何对象。例如：
```javascript
var feature = new ol.Feature({
  geometry: new ol.geom.Polygon(polyCoords), //默认渲染这个几何对象
  labelPoint: new ol.geom.Point(labelCoords), //定义另外一个几何对象，可调用feature.setGeometryName(labelPoint)更改渲染对象
  name: 'My Polygon'
});

feature.setGeometryName('labelPoint'); //setGeometryName(labelPoint)更改渲染对象
```
## 5.1 `feature`对象的属性和方法
* getGeometry() 获取关联渲染的几何对象
* getGeometryName() 获取渲染的几何对象的属性名字
* getId() 获取feature id，通过setId设置id
* getKeys() 获取属性名字集合
* getStyle() 获取渲染样式style
* setGeometry() 设置几何对象
* setGeometryName() 通过名字更改关联渲染的几何对象
* setId() 设置id
* setStyle() 设置渲染样式style

## 5.2 `feature` `source` `layer` `map`之间的关系小结
到这里，我们已经按照由上层到底层的顺序分析了 `map` → `layer` → `source` → `feature`(vector)，`map`有一个或者多个`layer`组成，而`source`则是构造`layer`的数据源，对于`vector`类型的`source`我们显然有更多的控制权，在页面中通过js可以对矢量图层进行一定程度上的编辑，`vector`类型的`source`是通过`feature`集合构造组成的，对矢量图层的操作归根到底是对`feature`的操作，`feature`控制着要渲染显示的点、线、面以及渲染的样式，比如说填充、描边的颜色、线型、线宽甚至文字。下一节将会介绍与`feature`关系紧密的`geometry`几何对象。

# 6 `geometry` 几何对象[demo](https://jiafengz.github.io/openlayer4/demo/olGeom/map.html)
上一节简单介绍了`feature`，`feature`是一个表层的概念，其关键还是`geometry`几何对象，不同的`geometry`决定了`feature`的不同表现形式。  
`ol.geom.GeometryType`定义了`geometry`的类型：
* `Circle` 圆
* `Point` 点
* `LineString` 线段
* `LinearRing` 圆环
* `Polygon` 多边形
* `MultiPoint` 多个点
* `MultiLineString`多线段 
* `MultiPolygon` 多个多边形
* `GeometryCollection`几何对象集合

## 6.1 `ol.geom`的类结构以及基本方法
```
ol.gemo.Geometry ─├── ol.geom.Geometry─├── ol.geom.GeometryCollection
                  ├                    ├── ol.geom.SimpleGeometry─├── ol.geom.Circle
                  ├                                               ├── ol.geom.LinearRing
                  ├                                               ├── ol.geom.LineString
                  ├                                               ├── ol.geom.MultiLineString
                  ├                                               ├── ol.geom.MultiPoint
                  ├                                               ├── ol.geom.MultiPolygon
                  ├                                               ├── ol.geom.Point
                  ├                                               ├── ol.geom.Polygon
                  ├
                  ├
                  └── ol.source.GeometryCollection
```
结构非常清晰，`ol.geom.Geometry`是基类，几何对象上拥有的一部分方法是继承自`ol.geom.Geometry`类：
* getClosestPoint()
* getExtent() 获取边界范围
* intersectsCoordinate(coordinate) 判断差传入的坐标点是否被包含
* rotate(angle, anchor) 旋转几何对象
* scale(sx, sy, opt_anchor) 伸缩几何对象
* transform(source, destination) 变换坐标系
* simplify(tolerance) 得到一个简化的几何对象

## 6.2 `ol.geom.SimpleGeometry`简单几何对象及其操作
我们已经知道`openlayer`提供了`Circle` `Point` `Polygon`等几种简单的几何对象。下面的例子展示几何对象的操作：
```javascript
//features of circle

function genreateFeatures() {
  var circle = new ol.geom.Circle({
    center: [0, 0],
    radius: 50
  })
  var point = new ol.geom.Point({
    coordinates: [20, 20]
  })
  var coordinates = [[0, 0], [10, 10], [20, 20], [0, 0]];
  polygon.setCoordinates([coordinates]);
  var features = [];
  features.push(new ol.Feature({
    geometry: circle,
    name: 'circle'
  }))
  features.push(new ol.Feature({
    geometry: point,
    name: 'point'
  }))
  features.push(new ol.Feature({
    geometry: polygon,
    name: 'polygon'
  }))
  features[0].setStyle(new ol.style.Style({
    stroke: new ol.style.Stroke({
      color: '#ffcc33',
      width: 2
    })
  }))
  features[1].setStyle(new ol.style.Style({
    stroke: new ol.style.Stroke({
      color: 'blue',
      width: 2
    })
  }))
  features[2].setStyle(new ol.style.Style({
    stroke: new ol.style.Stroke({
      color: 'yellow',
      width: 2
    })
  }))
  return features;
}
var source = new ol.source.Vector({
  features: (genreateFeatures)()
});
var vectorLayer = new ol.layer.Vector({
  source: source
});

 var map = new ol.Map({
    layers: [vectorLayer],
    target: 'map',
    view: new ol.View({
      center: [0, 0],
      zoom: 18
    })
  });
```

## 6.3 直接在画布上绘制`geometry`
```javascript
var canvas = document.getElementById('canvas');
var vectorContext = ol.render.toContext(canvas.getContext('2d'), {size: [100, 100]});

var fill = new ol.style.Fill({color: 'blue'});
var stroke = new ol.style.Stroke({color: 'black'});
var style = new ol.style.Style({
  fill: fill,
  stroke: stroke,
  image: new ol.style.Circle({
    radius: 10,
    fill: fill,
    stroke: stroke
  })
});
vectorContext.setStyle(style);

vectorContext.drawGeometry(new ol.geom.LineString([[10, 10], [90, 90]]));
vectorContext.drawGeometry(new ol.geom.Polygon([[[2, 2], [98, 2], [2, 98], [2, 2]]]));
vectorContext.drawGeometry(new ol.geom.Point([88, 88]));
```

# 7 ol.style 类设置元素样式 
`ol.style` 用于设置 `features` 的渲染样式，如果没有显式设置`features`的样式，则会使用`layer`上设置的样式。  
## 7.1 `ol.style`的结构
`ol.style`的类结构如下：  
```javascript
ol.style -> ol.style.Image  -> ol.style.RegularShape -> ol.style.Circle  
                            -> ol.style.Icon  
         -> ol.style.Fill  
         -> ol.style.Stroke  
         -> ol.style.Text  
```

## 7.2 使用例子
```javascript
var fill = new ol.style.Fill({
  color: 'blue'
})
var stroke = new ol.style.Stroke({
  width: 2,
  color: 'red'
})
var style = new ol.style.Style({
  image: new ol.style.Circle({
      fill: fill,
      stroke: stroke
    })
})
var vectorLayer = ol.Layer.Vector({
  source: new ol.source.Vector({
    url: 'https://openlayers.org/en/v4.6.5/examples/data/geojson/countries.geojson',
    format: new ol.format.GeoJSON()
  }),
  style: style
})

var map = new ol.Map({
    layers: [vectorLayer],
    target: 'map',
    view: new ol.View({
      center: [0, 0],
      zoom: 18
    })
})
```

## 7.3 聚合(features)样式设置
```javascript
 var clusterSource = new ol.source.Cluster({
    distance: 40,
    source: new ol.source.Vector({
        features: (function() {  //随机生成10000个feature元素点，添加到聚合source中，多个小的feature会就近聚合成一个大的feature
            var features = new Array(10000);
            var e = 4500000;
            for (var i = 0; i < 10000; ++i) {
              var coordinates = [2 * e * Math.random() - e, 2 * e * Math.random() - e];
              features[i] = new ol.Feature(new ol.geom.Point(coordinates));
            }
            return features;
        })()
    });
  });

var styleCache = {};
var clusters = new ol.layer.Vector({
  source: clusterSource,
  style: function(feature) {  //每个feature是由多个feature聚合组成
    var size = feature.get('features').length;
    var style = styleCache[size];
    if (!style) {
      style = new ol.style.Style({
        image: new ol.style.Circle({
          radius: 10, //聚合半径10px
          stroke: new ol.style.Stroke({ //描边颜色
            color: '#fff'
          }),
          fill: new ol.style.Fill({ //填充背景色
            color: '#3399CC'
          })
        }),
        text: new ol.style.Text({  //聚合点数量
          text: size.toString(),
          fill: new ol.style.Fill({
            color: '#fff'
          })
        })
      });
      styleCache[size] = style; //缓存复用style对象，减少对象内存空间耗用
    }
    return style;
  }
});

var map = new ol.Map({
  layers: [clusters],
  target: 'map',
  view: new ol.View({
    center: [0, 0],
    zoom: 2
  })
});
```
# 8 使用 ol.interaction 进行交互
## 8.1 地图默认具有的交互行为
`ol.interaction`是 openlayer 提供给用户和地图交互的操作类，使得地图可以识别响应用户特定动作，例如鼠标操作双击/单击/拖拽，键盘按键操作等。当使用 `ol.Map`创建 map 的实例时，地图会默认添加 `ol.interaction.defaults`包含的操作类，使地图具有默认的交互行为：
* ``ol.interaction.DragRotate`` 同时按下 shift 和 alt键时，按下鼠标或者拖拽可以旋转地图
* ``ol.interaction.DoubleClickZoom`` 双击放大地图 
* ``ol.interaction.DragPan``  拖拽移动地图
* ``ol.interaction.PinchRotate`` 触控 双指扭转旋转地图
* ``ol.interaction.PinchZoom`` 触控 双指收拢缩放地图
* ``ol.interaction.KeyboardPan`` 键盘控制 导航键移动地图
* ``ol.interaction.KeyboardZoom`` 键盘控制 + - 键缩放地图
* ``ol.interaction.MouseWheelZoom`` 鼠标滚轮控制 缩放地图
* ``ol.interaction.DragZoom`` 拖拽放大地图 需同时按下 shift 键

以上是地图的默认行为，可以使用`ol.interaction.defaults`配置开启关闭特定的交互行为：
```javascript
var map = new ol.Map({
        layers: layers,
        target: 'map',
        view: new ol.View({
          projection: 'EPSG:4326',
          center: [0, 0],
          zoom: 2
        }),
        interaction: ol.interaction.defaults({
          altShiftDragRotate: true,  //是否开启alit shift 键拖拽旋转
          constrainResolution: false, //当缩放动作结束后是否自动缩放到整数层级
          doubleClickZoom: true, //是否开启双击放大
          keyboard: true, //是否开启键盘控制
          mouseWheelZoom: true, //是否开启鼠标滚轮放大
          shiftDragZoom: true, //是否开启shift键拖拽放大
          dragPan: true, //是否开启拖拽移动
          pinchRotate: true, 是否开启触控旋转
          pinchZoom: true, //是否开启触控放大
          zoomDelta: 1, //放大层级增量
          zoomDuration: 100 //缩放动画过程持续时间
        })
      });
```

## 8.2 几个其他特殊的交互行为
* `ol.interaction.DragAndDrop`  通过拖放给source添加vector数据，就是说可以在外部拖拽矢量数据添加到地图中动态显示
```javascript
var interaction = new ol.interaction.DragAndDrop({
  formatConstructors: function(new ol.format.Feature) {}, //格式构造函数，处理添加输入的features
  source: ..., //feature要添加到的source容器，当feature添加到source容器时，会移除source中原有的feature，如果只要添加feature，则不要使用该选项，在addFeatures事件中进行处理
  projection: ...,
  target: ... //拖放的目标区域，默认是视口 viewport 元素
})
```

* `ol.interaction.Select`   
首先翻译一段来自官网API文档的原话来说明`ol.interaction.Select`的作用：这是为选择vector features（矢量元素）而设置的交互行为，通常而言被选择的feature具有不同的渲染样式style，因此这种选择元素的交互操作可以用来高亮特定的feature，也可以选择元素提供给其他的操作进行处理，例如修改编辑被选择的元素。有三种方式筛选要选择的元素：使用浏览器事件的 `condition`选项/ toggle add remove multi 选项/使用`filter`选项过滤函数。下面的例子（改造自[Box Selection](http://openlayers.org/en/latest/examples/box-selection.html)，我修改后的例子[olDraw](https://jiafengz.github.io/openlayer4/demo/olInteraction/olDraw.html)）就展示使用`ol.interaction.Select`结合`ol.interaction.DragBox`选择地图元素的功能：
```javascript
      var map = new ol.Map({
        layers: [
          new ol.layer.Vector({
            source: new ol.source.Vector({
              url: 'https://openlayers.org/en/v4.6.5/examples/data/geojson/countries.geojson',
              format: new ol.format.GeoJSON()
            })
          })
        ],
        target: 'map',
        view: new ol.View({
          center: [0, 0],
          zoom: 2
        })
      });

      var select = new ol.interaction.Select(); //创建一个选择交互行为
      map.addInteraction(select); //该地图添加选择交互
      var selectedFeatures = select.getFeatures(); 
      var dragBox = new ol.interaction.DragBox(); //创建一个绘制矩形盒子的交互行为
      map.addInteraction(dragBox); //添加到地图上

      var vectorSources = [];
      map.getLayers().forEach(function(layer) {
          if (layer instanceof(ol.layer.Vector)) vectorSources.push(layer.getSource());
      })
      var vectorSource = vectorSources[0];
      dragBox.on('boxend', function() {
        var extent = dragBox.getGeometry().getExtent();
        vectorSource.forEachFeatureIntersectingExtent(extent, function(feature) { //遍历矩形盒子范围内的feature
          selectedFeatures.push(feature);  //添加到select的features集合中
        });
      });

      dragBox.on('boxstart', function() {
        selectedFeatures.clear();
      });

      selectedFeatures.on(['add', 'remove'], function() {
        var names = selectedFeatures.getArray().map(function(feature) {
          return feature.get('name');
        });
        if (names.length > 0) {
          console.log(names.join(', '));
        } else {
          console.log('No countries selected');
        }
      });
```

* `ol.interaction.Draw`   
这个是openlayer提供的一个很有用很强大的交互功能，允许用户实时在地图上进行绘制 feature geometry 元素几何图形。  
`new ol.interaction.Draw(options)`  
  `features`  绘制的目标元素集合，绘制的feature会添加到该目标元素集合
  `source`  绘制的目标数据容器，绘制的feature会添加到该数据源容器中
  `type`  绘制图形的类型，可选 Point/LineString/Polygon/MultiPoint/MultiLineString/MultiPolygon/Circle
  `finishCondition`  一个回调函数根据传入`ol.MapBrowserEvent`浏览器事件返回true或者false决定绘制动作是否结束
  `geometryName` 设置绘制的feature组成的几何图形的名字   
  `geometryFunction` 当绘制的坐标点更新时调用的方法，可以通过该方法自定义绘制的图形

```javascript
var source = new ol.source.Vector();  //创建一个source容器
var vector = new ol.layer.Vector({  //使用该source构造一个矢量图层，并且设置渲染样式
  source: source,
  style: new ol.style.Style({
    fill: new ol.style.Fill({
      color: 'rgba(255, 255, 255, 0.2)'
    }),
    stroke: new ol.style.Stroke({
      color: '#ffcc33',
      width: 2
    }),
    image: new ol.style.Circle({
      radius: 7,
      fill: new ol.style.Fill({
        color: '#ffcc33'
      })
    })
  })
});

var map = new ol.Map({  //使用上面的矢量图层创建一个地图实例
  layers: [vector],
  target: 'map',
  view: new ol.View({
    center: [0, 0],
    zoom:4
  })
})

var draw = new ol.interaction.Draw({
  source: source，  //绘制的图形会添加到上面构造的的source中，最终渲染显示在地图map上
  type: 'Circle',  //绘制一个圆形
})

map.addInteraction(draw); //地图添加绘制圆形的交互操作
```
有一些很有用的方法，说明如下：
* `ol.interaction.Draw.createBox()` 返回一个绘制盒子形状的函数 ，利用该方法与``type:Circle``可以改变绘制的默认图形：
```javascript
var draw = new ol.interaction.Draw({
  source: source,
  type: 'Circle',
  geometryFunction: ol.interaction.Draw.createBox() //type 选项设置绘制的图形是圆形，但是geometryFunction函数覆盖了type的作用，绘制出来的将会是矩形盒子
})
```
* `ol.interaction.Draw.createRegularPolygon()` 返回一个绘制正多边形的函数 ，利用该方法与``type:Circle``可以改变绘制的默认图形：
```javascript
var draw = new ol.interaction.Draw({
  source: source,
  type: 'Circle',
  geometryFunction: ol.interaction.Draw.createRegularPolygon({
    sides: 5, //五边形
    angle: 0 //第一个顶点的角度，默认是0
  }) //type 选项设置绘制的图形是圆形，但是geometryFunction函数覆盖了type的作用，绘制出来的将会是正五边形
})
```
* `ol.interaction.Draw.handleEvent` 绘制完成触发的事件

# 9 overlay 的使用
## 9.1 `overlay`的属性及方法
`ol.Overlay`顾名思义，就是地图上的浮层元素，不同于 map 地图上通过canvas绘图环境绘制的地图元素，`overlay`是完全我们自定义的普通html元素，有更多的控制自由度。因此通过 `overlay` 可以根据地理坐标创建自定义标注或者浮层/弹窗，移动地图`overlay`元素也会随着地图移动。  
``new ol.Overlay(options)``<br/>
``options.id`` 浮层元素id，设置id后可通过 ``getOverlayById(id)``获取该浮层元素<br/>
``options.element`` 浮层元素，普通的html元素，例如 ``element: document.createElement('div')``<br/>
``options.offset`` 定位浮层元素的偏移量，默认[0, 0]<br/>
``options.position``  浮层元素的地理坐标<br/>
``options.positioning`` 定位方式，可选：'bottom-left', 'bottom-center', 'bottom-right', 'center-left', 'center-center', 'center-right', 'top-left', 'top-center'<br/>
``options.stopEvent`` 是否阻止overlay元素上的事件冒泡到map上，默认true，阻止冒泡<br/>
``options.autoPan`` 是否自动移动地图使得overlay元素始终在可见范围<br/>

方法：<br/>
* getElement 获取overlay元素
* getMap 获取关联的map对象
* getOffset 获取相对定位坐标偏移量
* getPosition 获取地理坐标
* getPositioning 获取定位方式
* setElement 设置浮层元素
* setMap 设置关联的地图
* setOffset 设置偏移量
* setPosition 设置坐标
* setPositioning 设置定位方式

## 9.2 使用`overlay`在地图上放置标注[demo](https://jiafengz.github.io/openlayer4/demo/olOverlay/olOverlay.html)
```javascript
var source =  new ol.source.Vector({
        url: 'https://openlayers.org/en/v4.6.5/examples/data/geojson/countries.geojson',
        format: new ol.format.GeoJSON()
  });
  var layer = new ol.layer.Vector({
      source: source
  });

  var map = new ol.Map({
    layers: [layer],
    target: 'map',
    view: new ol.View({
      center: [0, 0],
      zoom: 2
    })
  });


  source.on('change', function() {  //监听属性改变事件
    if (source.getState() == 'ready') {  //当source加载完毕后获取feature的属性值，设置overlay
        source.getFeatures().forEach(function(feature) {
        var text = feature.get('name');
        var geom = feature.getGeometry();
        if (geom instanceof ol.geom.Polygon) {
          var coords = geom.getInteriorPoint().getCoordinates();
          createOverlay(text, coords); //生成overlay，显示feature的名字
        }
      })
    }
  })
  

  function createOverlay(text, coords) {
    var ele = document.createElement('div');
    ele.classList.add('ol-popup');
    ele.innerHTML = '<a href="#" class="ol-popup-closer"></a>' +
                                  '<div class="popup-content">'  + text +'</div>'
    var marker = new ol.Overlay({
      position: coords,
      positioning: 'center-center',
      element: ele,
      stopEvent: true
    });
    map.addOverlay(marker);
  }
``` 
# 10 事件
## openlayer 的事件体系
# 11 一些应用的例子
## 11.1 实现测量距离和面积
## 11.2 判断点是否在面内
## 11.3 拾取坐标
## 11.4 点击地图获取点击点关联的 features
## 11.5 根据 features 自适应设置地图extent显示范围
## 11.6 热力图
## 11.7 features 聚合显示
## 11.8 移动鼠标高亮显示 feature 
