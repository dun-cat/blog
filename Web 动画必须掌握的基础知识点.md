## Web 动画必须掌握的基础知识点 
### 介绍

动画里的涉及到角度与坐标的关系，主要通过数学三角学知识来处理。而在物理渲染中，模型的处理主要通过物理学的公式来实现。下面涉及到的数学公式可以参考这篇[《常用的数学公式》](/articles/math-formulas)来理解。

### canvas 里的坐标系（coordinate system）

<img src='coordinate_system.png' style='max-width:700px'>

canvas 里的坐标系中心并不是在画布的正中心，而是在画布的左上角。

这个坐标系有它的历史背景，因为电子枪是从左往右，从上往下扫描屏幕的，后来也成为了图形学编程里屏幕坐标系的习惯，这种习惯也方便了图形编程中某些问题的处理。

### canvas 四象限的角度

在 canvas 里角度的正负表示 如下：

<img src='angle.png' style="background:#fffad0"/>

通常，我们可以通过 Javascript 的反正切函数`Math.atan2(y,x)`来获取对应的角度，这里的角度单位是弧度。

<img src='angle2.png' style="background:#fffad0"/>

在 canvas 坐标系统里的角度计算：

``` javascript
Math.atan2(-1, -2) * 180 / Math.PI
// => -153.43494882292202
```

当我们要逆时针计算角度时，要减去角度大小。而顺时针计算时，要加上角度大小。如果是两个角度向量，那么直接使用向量加法即可，图形编程中可以使用`弧度来表示角度向量`。

### 角速度

在 canvas 里下面的方向的速度分解到 $x$ 轴 $y$ 轴的向量表示如下：

<img src='angular_speed.svg'>

在知道了`角度（angle）`和`速度（speed）`之后，我们利用三角函数可以获取 $vx$， $vy$的速度向量：

``` javascript
vx = Math.cos(angle) * speed
vy = Math.sin(angle) * speed
```

这里的`angle`是弧度，我们可以通过转换公式来转换角度：

``` javascript
vx = Math.cos(degree * Math.PI / 180) * speed
vy = Math.sin(degree * Math.PI / 180) * speed
```

### 角加速度

加速度和速度向量类似，由大小（力的大小）和方向组成，由此可以分解到 $x$ 轴和 $y$ 轴上：

``` javascript
let force = 8
let angle = 450

let ax = Math.cos(angle * Math.PI / 180) * force
let ay = Math.sin(angle * Math.PI / 180) * force
```

把`加速度向量`加入`速度向量`中：

``` javascript
vx += ax;
vy += ay;
```

把`速度向量`加入`坐标`中：

``` javascript
object.x += vx;
object.y += vy;
```

### 坐标旋转

<img src='rotation.png' style='max-width: 500px'/>

**1. 只知道半径（radius）和旋转后的角度（angle）的情况**

通过下面的公式来计算旋转后的坐标：

``` javascript
object.x = centerX + cos(angle) * radius
object.y = centerY + sin(angle) * radius
```

**2. 知道旋转（rotation）角度和起始坐标 (x,y) 的情况**


通过下面的公式来计算旋转后的坐标：

``` javascript
x1 = x * cos(rotation) - y * sin(rotation)
y1 = y * cos(rotation) + x * sin(rotation)
```

如果是相对坐标 (centerX, centerY) 为旋转中心，可以把公式写成下面这样：

``` javascript
x1 = (x - centerX) * cos(rotation) - (y - centerY) * sin(rotation)
y1 = (y - centerY) * cos(rotation) + (x - centerX) * sin(rotation)
```

上面是如何推导出来的呢？

1.首先通过起始点(x,y)和目标点(x1,y1)的`半径`和`角度`，我们可以知道以下等式：

``` javascript
x = radius * cos(angle)
y = radius * sin(angle)

x1 = radius * cos(angle + rotation)
y1 = radius * sin(angle + rotation)
```

2.接下来利用下面数学的**三角恒等式**：

``` javascript
// 余弦和角公式
cos(a + b) = cos(a) * cos(b) - sin(a) * sin(b)
// 正弦和角公式
sin(a + b) = sin(a) * cos(b) + cos(a) * sin(b)
```

3.我们把**1**里的`x1、y1`公式利用三角恒等式展开：

``` javascript
x1 = radius * cos(angle) * cos(rotation) - radius  *  sin(angle) * sin(rotation)
y1 = radius * sin(angle) * cos(rotation) + radius * cos(angle) * sin(rotation)
```

4.最后把**1**里的`x、y`变量代入公式，就得到下面的方程：

``` javascript
x1 = x * cos(rotation) - y * sin(rotation)
y1 = y * cos(rotation) + x * sin(rotation)
```
