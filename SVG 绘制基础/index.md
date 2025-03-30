## SVG 绘制基础 
### 简介

SVG（Scalable Vector Graphics）是一种基于 XML 的图形格式，用于描述二维矢量图形。

### 坐标系统

SVG（Scalable Vector Graphics）的坐标系统是理解和使用 SVG 的关键部分。它定义了如何在 SVG 画布上定位和绘制图形。以下是 SVG 坐标系统的一些基本概念：

#### 1. SVG 画布

- SVG 画布可以被视为一个你在其中绘制图形的区域。
- 画布的大小由 SVG 元素的 `width` 和 `height` 属性定义。

#### 2. 坐标系

- 在 SVG 中，默认的坐标原点（0,0）位于画布的`左上角`。
- x 轴水平向右延伸，y 轴垂直向下延伸。
- 坐标值定义了相对于原点的位置。

#### 3. `viewBox` 属性

- `viewBox` 属性定义了 SVG 画布内的一个矩形区域，你的 SVG 图形将在这个区域内绘制。
- 它由四个值组成：`min-x`, `min-y`, `width`, `height`。
- `viewBox` 允许你指定一个不同于 SVG 画布实际大小的逻辑坐标系统。这对于制作响应式 SVG 非常有用。

#### 4. 用户坐标系和物理坐标系

- 当没有使用 `viewBox` 时，SVG 的用户坐标系与物理坐标系相同。
- 使用 `viewBox` 后，用户坐标系可以独立于 SVG 画布的实际物理尺寸。

#### 5. 坐标变换

- SVG 允许通过 `transform` 属性（如平移 `translate`、缩放 `scale`、旋转 `rotate` 等）对图形进行坐标变换。
- 这些变换可以改变图形的位置、大小和方向。

#### 示例

无 viewBox

``` xml
<svg width="200" height="200" xmlns="http://www.w3.org/2000/svg">
  <circle cx="50" cy="50" r="40" fill="blue" />
</svg>
```

viewBox="0 0 100 100"

``` xml
<svg width="200" height="200" viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg">
  <circle cx="50" cy="50" r="40" fill="blue" />
</svg>
```

viewBox="50 50 100 100"

``` xml
<svg width="200" height="200" viewBox="50 50 100 100" xmlns="http://www.w3.org/2000/svg">
  <circle cx="50" cy="50" r="50" fill="blue" />
</svg>
```

viewBox="50 50 50 100"

``` xml
<svg width="200" height="200" viewBox="50 50 50 100" xmlns="http://www.w3.org/2000/svg">
  <circle cx="50" cy="50" r="50" fill="blue" />
</svg>
```

viewBox="0 0 300 300"

``` xml
<svg width="200" height="200" viewBox="0 0 300 300" xmlns="http://www.w3.org/2000/svg">
  <circle cx="50" cy="50" r="50" fill="blue" />
</svg>
```

<svg width="200" height="200" xmlns="http://www.w3.org/2000/svg">
  <circle cx="50" cy="50" r="50" fill="blue" />
</svg>

<svg width="200" height="200" viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg">
  <circle cx="50" cy="50" r="50" fill="blue" />
</svg>

<svg width="200" height="200" viewBox="50 50 100 100" xmlns="http://www.w3.org/2000/svg">
  <circle cx="50" cy="50" r="50" fill="blue" />
</svg>

<svg width="200" height="200" viewBox="50 50 50 100" xmlns="http://www.w3.org/2000/svg">
  <circle cx="50" cy="50" r="50" fill="blue" />
</svg>

<svg width="200" height="200" viewBox="0 0 300 300" xmlns="http://www.w3.org/2000/svg">
  <circle cx="50" cy="50" r="50" fill="blue" />
</svg>

width 和 height 可以使用百分比，相对于父元素来决定画布大小：

``` svg
<svg width="100%" height="200" viewBox="0 0 100 100"
  xmlns="http://www.w3.org/2000/svg" version="1.1">
  <circle cx="50" cy="50" r="40" fill="blue" />
</svg>

```

<svg width="100%" height="200" viewBox="0 0 100 100"
  xmlns="http://www.w3.org/2000/svg" version="1.1">
  <circle cx="50" cy="50" r="40" fill="blue" />
</svg>

### 通用属性

1. **`width` 和 `height`**:
   - 定义 SVG 画布的宽度和高度。
2. **`fill`**:
   - 定义图形的填充颜色。默认是黑色，可以设为任何颜色或 `none`。
3. **`stroke`**:
   - 定义图形轮廓的颜色。
4. **`stroke-width`**:
   - 定义图形轮廓的宽度。

### 样式

- SVG 元素可以通过内联样式或 CSS 文件进行样式化。样式属性如 `fill`, `stroke`, `stroke-width` 可以直接应用在 SVG 元素上，或者通过 CSS 类或 ID 选择器设置。

通过这些属性，你可以创建各种图形和效果。SVG 提供了广泛的功能集，可以用于复杂的图形设计和交互式应用。

### 根元素 (`svg`)

SVG 根元素是 `<svg>`，它是所有 SVG 内容的容器。这个元素拥有多个属性，这些属性控制着 SVG 图形的整体表现和行为。以下是一些常见的 `<svg>` 根元素属性：

#### `width` 和 `height`

- **定义**: 设置 SVG 画布的宽度和高度。
- **示例**: `<svg width="100" height="100">...</svg>`

#### `viewBox`

- **定义**: 定义 SVG 画布的视窗，即你希望展示的 SVG 区域。它通常包含四个值：`min-x`, `min-y`, `width`, 和 `height`。
- **示例**: `<svg viewBox="0 0 100 100">...</svg>`
- **用途**: 用于实现响应式 SVG，允许 SVG 图形在不同大小的容器中自适应缩放。

#### `xmlns`

- **定义**: 定义 SVG 的`命名空间`。对于内联 SVG，这个属性是必须的，因为它告诉浏览器这是 SVG 内容。
- **示例**: `<svg xmlns="http://www.w3.org/2000/svg">...</svg>`

> 虽然不添加在浏览器也能渲染，但在某些系统的`预览`功能会因为缺少该属性而不可用。

#### `preserveAspectRatio`

- **定义**: 控制当 SVG 缩放以适应其容器时如何保持其宽高比。
- **示例**: `<svg preserveAspectRatio="xMidYMid meet">...</svg>`
- **用途**: 确保在缩放时 SVG 图形按照一定的方式保持其原始宽高比。

#### `fill` 和 `stroke`

- **定义**: 设置 SVG 图形的默认填充(`fill`)和描边(`stroke`)颜色。
- **示例**: `<svg fill="red" stroke="blue">...</svg>`
- **注意**: 这些属性通常在单独的图形元素上设置，如 `<circle>`、`<path>` 等。

#### `version`

- **定义**: 指定 SVG 的版本。在 HTML5 中嵌入 SVG 时，通常不需要这个属性。
- **示例**: `<svg version="1.1">...</svg>`

#### 使用时的注意事项

- `width` 和 `height` 用于定义 SVG 画布的物理大小，而 `viewBox` 用于定义画布内的逻辑坐标系统。
- 如果没有指定 `width` 和 `height`，SVG 画布可能会根据页面的其他元素自动调整大小。
- `preserveAspectRatio` 的不同值会影响 SVG 在响应式设计中的表现，尤其是在 SVG 尺寸与容器尺寸不一致时。
- `xmlns` 属性对于在 HTML 文档中正确渲染 SVG 是必需的。

通过合理使用这些属性，你可以有效地控制 SVG 的显示和行为，使其适应各种不同的设计需求。

### SVG 元素

#### 矩形（`<rect>`）

``` svg
<svg width="100" height="100" xmlns="http://www.w3.org/2000/svg">
  <rect width="50" height="50" x="25" y="25" fill="blue" />
</svg>
```

<svg width="100" height="100" xmlns="http://www.w3.org/2000/svg">
  <rect width="50" height="50" x="25" y="25" fill="blue" />
</svg>

1. **`x` 和 `y`**:
   - 矩形左上角的 x 和 y 坐标。
2. **`width` 和 `height`**:
   - 矩形的宽度和高度。

#### 圆形（`<circle>`）

``` svg
<svg width="100" height="100" xmlns="http://www.w3.org/2000/svg">
  <circle cx="50" cy="50" r="40" fill="green" />
</svg>

```

<svg width="100" height="100" xmlns="http://www.w3.org/2000/svg">
  <circle cx="50" cy="50" r="40" fill="green" />
</svg>

1. **`cx` 和 `cy`**:
   - 圆心的 x 和 y 坐标。
2. **`r`**:
   - 圆的半径。

#### 椭圆（`<ellipse>`）

``` svg
<svg width="100" height="100" xmlns="http://www.w3.org/2000/svg">
  <ellipse cx="50" cy="50" rx="40" ry="20" fill="red" />
</svg>
```

<svg width="100" height="100" xmlns="http://www.w3.org/2000/svg">
  <ellipse cx="50" cy="50" rx="40" ry="20" fill="red" />
</svg>

1. **`cx` 和 `cy`**:
   - 椭圆中心的 x 和 y 坐标。
2. **`rx` 和 `ry`**:
   - 椭圆的水平半径（rx）和垂直半径（ry）。

#### 线条（`<line>`）

``` svg
<svg width="100" height="100" xmlns="http://www.w3.org/2000/svg">
  <line x1="10" y1="10" x2="90" y2="90" stroke="black" stroke-width="2" />
</svg>
```

<svg width="100" height="100" xmlns="http://www.w3.org/2000/svg">
  <line x1="10" y1="10" x2="90" y2="90" stroke="black" stroke-width="2" />
</svg>

1. **`x1`, `y1`, `x2`, `y2`**:
   - 线条的起点（x1, y1）和终点（x2, y2）坐标。

#### 折线（`<polyline>`）

``` svg
<svg width="100" height="100" xmlns="http://www.w3.org/2000/svg">
  <polyline points="10,10 40,30 70,70 90,10" stroke="orange" fill="none" stroke-width="2" />
</svg>
```

<svg width="100" height="100" xmlns="http://www.w3.org/2000/svg">
  <polyline points="10,10 40,30 70,70 90,10" stroke="orange" fill="none" stroke-width="2" />
</svg>

1. **`points`**:
   - 一系列点，定义折线或多边形的顶点。

#### `多边形`（`<polygon>`）

``` svg
<svg width="100" height="100" xmlns="http://www.w3.org/2000/svg">
  <polygon points="50,10 10,90 90,90" fill="purple" />
</svg>
```

<svg width="100" height="100" xmlns="http://www.w3.org/2000/svg">
  <polygon points="50,10 10,90 90,90" fill="purple" />
</svg>

#### `路径`（`<path>`）

``` svg
<svg width="100" height="100" xmlns="http://www.w3.org/2000/svg">
  <path d="M10 10 H 90 V 90 H 10 L 10 10" fill="none" stroke="brown" stroke-width="2" />
</svg>
```

<svg width="100" height="100" xmlns="http://www.w3.org/2000/svg">
  <path d="M10 10 H 90 V 90 H 10 L 10 10" fill="none" stroke="brown" stroke-width="2" />
</svg>

1. **`d`**:
   - 描述路径的形状。`d` 属性包含一系列命令和参数，如 `M` (moveto), `L` (lineto), `C` (curveto)， `Z (closepath)`。
   - `C`：cubic bezier curve，三次贝塞尔曲线。

`移动`指令：

1. **M (moveto)**：将绘图游标移动到指定的坐标点，但不绘制任何线条或曲线。例如，`M 10 10` 将游标移动到坐标(10, 10)。

绘制`直线`指令：

2. **L (lineto)**：从当前位置绘制一条直线到指定的坐标点。例如，`L 20 20` 将从当前位置绘制一条直线到坐标(20, 20)。
3. **H (horizontal lineto)**：绘制一条水平线，只需指定x坐标。例如，`H 30` 将绘制一条水平线到x坐标30，y坐标保持不变。
4. **V (vertical lineto)**：绘制一条垂直线，只需指定y坐标。例如，`V 40` 将绘制一条垂直线到y坐标40，x坐标保持不变。

绘制`曲线`指令：

5. **C (curveto)**：绘制三次贝塞尔曲线，需要指定两个控制点和终点。例如，`C 50 50, 60 60, 70 70` 将绘制一条从当前位置到(70, 70)的三次贝塞尔曲线，其中(50, 50)和(60, 60)是控制点。
6. **S (smooth curveto)**：绘制平滑的三次贝塞尔曲线，只需要指定一个控制点和终点，前一个控制点会自动根据上一段曲线的控制点计算。例如，`S 80 80, 90 90` 将绘制一条从当前位置到(90, 90)的平滑三次贝塞尔曲线。
7. **Q (quadratic curveto)**：绘制二次贝塞尔曲线，需要指定一个控制点和终点。例如，`Q 100 100, 110 110` 将绘制一条从当前位置到(110, 110)的二次贝塞尔曲线，其中(100, 100)是控制点。
8. **T (smooth quadratic curveto)**：绘制平滑的二次贝塞尔曲线，只需要指定终点，前一个控制点会自动根据上一段曲线的控制点计算。例如，`T 120 120` 将绘制一条从当前位置到(120, 120)的平滑二次贝塞尔曲线。

绘制`圆弧`指令：

9. **A (elliptical arc)**：绘制椭圆弧，需要指定椭圆的半径、旋转角度、弧的大/小方向和终点坐标。例如，`A 30 20 45 1 0 150 100` 将绘制一个椭圆弧。

`闭合`指令：

10. **Z (closepath)**：将当前路径闭合，连接到路径的起点，创建一个封闭的形状。

#### `<text>` (文本)

``` svg
<svg width="300" height="200" xmlns="http://www.w3.org/2000/svg">
  <text x="10" y="50" fill="black">Hello, SVG!</text>
</svg>
```

<svg width="300" height="100" xmlns="http://www.w3.org/2000/svg">
  <text x="10" y="50" fill="black">Hello, SVG!</text>
</svg>

1. **`x` 和 `y`**:
   - 文本开始的 x 和 y 坐标。
2. **`font-family`**:
   - 定义文本的字体。
3. **`font-size`**:
   - 定义文本的大小。

### 工具和库

 Adobe Illustrator、Inkscape（免费）等工具来创建 SVG。
 D3.js 这样的 JavaScript 库来动态生成和操作 SVG。

参考资料：

\> [https://jwatt.org/svg/authoring/](https://jwatt.org/svg/authoring/)

\> [https://pjchender.blogspot.com/2017/03/svg-viewport-viewbox-zoomdrag.html](https://pjchender.blogspot.com/2017/03/svg-viewport-viewbox-zoomdrag.html)
