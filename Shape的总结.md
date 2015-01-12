
#### shape的使用总结

> shape在Android中是比较常用的，一些简单的图形效果，都可以由shape绘制，可以绘制：rectangle(矩形)、oval(椭圆)、ring(环)、line(线)。。这几种图形效果。

##### corners节点

> 这个属性可以用来设置shape的圆角属性。

	<corners android:radius="5dp"/>

以上就是设置5dp的圆角弧度。

##### gradient节点

> 渐变属性，可以通过设置开始、中间、结束颜色，来实现渐变效果

	<gradient 
        android:startColor="#5546F546"
        android:endColor="#227EFC7E"
        android:angle="90"        
        />

上面的angle属性重点说一下：

> angle属性用来设置渐变的方向，值是坐标系中的一个角度

* 0 ： 表示向右渐变
* 90： 表示向上渐变
* 180：表示向左渐变
* 270：表示向下渐变

##### solid节点

> 用来设置shape图形中的填充色，它只有一个color属性

	<solid android:color="#ccc"/>

##### stroke节点

> 用来设置shape的边框，可以指定边框的宽度，和颜色

	<stroke android:width="1dp" android:color="#ccc" />

##### padding节点

> 设置内边距，可以设置上下左右的padding值

	<padding android:left="1dp" android:top="1dp" android:right="1dp" android:bottom="1dp"/>