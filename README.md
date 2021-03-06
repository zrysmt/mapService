# mapService
使用百度地图API和datatables扩展整体后台模块，主要包括查看模块和新增模块
editor修改模块可以单独参看另一个仓库 editService   

mapService和editService两个仓库几乎涵盖了使用百度地图API的增删改查功能，均实现。

注意 需要使用PHP的框架模板才能运行，其实你也可以忽略PHP文件，PHP文件主要是为了从数据库中读取数据，存数据。
   editService模块的PHP还使用的datatablesd库的扩展，可以实时改正更新到数据库中。

# 功能清单

- 查看地图模块可以分级显示不同的形式；搜索定位
- 【添加房源模块】能够根据地址定位到位置，也可以使用marker(可以拖动)自动获取详细地址信息
- 【添加房源模块】中增加测试距离和面积(百度API没有直接提供，使用GeoUtils.js)，响应式设计，绘图(使用DrawingManager.js扩展库)和保存，定位  
- 2016/12/13 【添加房源模块】自动提交绘制的多边形，可以选择删除多边形   
- 2016/12/28 【添加房源模块】可以选择颜色保存颜色配置到数据库中，可以选择marker删除(注意这里的坐标转换)
- 2017/01/17 【添加房源模块】增加全景，全屏和点聚合功能
- 2017/01/18 【添加房源模块】百度地图和高德地图切换，根据屏幕宽度决定加载点聚合图
- 2017/01/19 【添加房源模块】新增绘制点的类型，如矩形、菱形、五角星。
- 2017/01/20 【添加房源模块】按M切换到高德地图，并且能够将覆盖物加载到高德地图上(经过转码)

# 重要技术：

1.六种类型的坐标系 [点击查看](http://blog.csdn.net/lanximu/article/details/16964967) 
经纬度：通过经度（longitude）和纬度（latitude）描述的地球上的某个位置。
平面坐标：投影之后的坐标（用x和y描述），用于在平面上标识某个位置。
像素坐标：描述不同级别下地图上某点的位置。
图块坐标：地图图块编号（用x和y描述）。
可视区域坐标：地图可视区域的坐标系（用x和y描述）。
覆盖物坐标：覆盖物相对于容器的坐标（用x和y描述）。

百度地图的marker的 left top值是可视区域坐标
marker转化为百度地理坐标

```javascript
if (event.type == "click" && nodeName == "SPAN" && event.target.className=="BMap_Marker BMap_noprint") {//markers
    var left = parseInt(event.target.style['left']);
    var top = parseInt(event.target.style['top']);
    var pixel = new BMap.Pixel(left+10,top+25);//问题：加减多少还不一定？？
   // var newPoint = map.pixelToPoint(pixel);//有问题
    var newPoint2 = map.overlayPixelToPoint(pixel);
}
```
2.地图切换
需要记录地图zoom和中心点坐标信息

```javascript
getMapInfo:function(mapName,map){
   if(mapName=="bMap"){
       return {
           zoom:map.getZoom(),
           mapType:map.getMapType().getName(),
           center:map.getCenter()
        } 
   }else if(mapName=="aMap"){
        return {
           zoom:map.getZoom(),
           // mapType:map.getMapType().getName(),
           center:map.getCenter()
        } 
   }
   
}
```
3.百度坐标和高德坐标(火星坐标)转换

```javascript
bd2gcj:function(bdLat,bdLng){
    if (outOfChina(bdLat, bdLng)) {
        return {"lat": bdLat, "lng": bdLng};
    }
    var x = bdLng - 0.0065, y = bdLat - 0.006;
    var z = Math.sqrt(x * x + y * y) - 0.00002 * Math.sin(y * Math.PI);
    var theta = Math.atan2(y, x) - 0.000003 * Math.cos(x * Math.PI);
    var gcjLng = z * Math.cos(theta);
    var gcjLat = z * Math.sin(theta);
    return {"lat": gcjLat, "lng": gcjLng};
},
gcj2bd:function(gcjLat, gcjLng){
    if (outOfChina(gcjLat, gcjLng)) {
        return {"lat": gcjLat, "lng": gcjLng};
    }
    var x = gcjLng, y = gcjLat;
    var z = Math.sqrt(x * x + y * y) + 0.00002 * Math.sin(y * Math.PI);
    var theta = Math.atan2(y, x) + 0.000003 * Math.cos(x * Math.PI);
    var bdLng = z * Math.cos(theta) + 0.0065;
    var bdLat = z * Math.sin(theta) + 0.006;
    return {"lat": bdLat, "lng": bdLng};
}
```
参考阅读：

- [百度地图API详解之地图坐标系统](http://blog.csdn.net/lanximu/article/details/16964967)   