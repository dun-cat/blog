## 2D 矩阵 (Matrix) 的基础变换 
### 坐标表示

平面上的点坐标的矩阵表示：$P = \begin{bmatrix}x \\\ y\end{bmatrix}$

### 变换矩阵

#### 平移（Translation）

平移变换是个`单列矩阵`：$T=\begin{bmatrix}tx \\\ ty\end{bmatrix}$

如果对点进行平移，只需进行`矩阵加法`运算：
$P' = \begin{bmatrix}x \\\ y\end{bmatrix} + \begin{bmatrix}tx \\\ ty\end{bmatrix} = \begin{bmatrix}x + tx \\\ y + ty\end{bmatrix}$

#### 缩放（Scale）

`缩放矩阵`：
$S=\begin{bmatrix}
  sx & 0 \\\
  0 & sy
\end{bmatrix}$

如果对点进行缩放，只需进行`矩阵乘法`运算：
$P'= \begin{bmatrix}
  sx & 0 \\\
  0 & sy
\end{bmatrix} \times \begin{bmatrix}x \\\ y\end{bmatrix} = \begin{bmatrix}
  sx \times x \\\
  sy \times y
\end{bmatrix}$

#### 旋转（Rotation）

θ 为用于表示绕`原点`的`顺时针`旋转角度，则`旋转矩阵`为：
$R = \begin{bmatrix}
  cos(θ) & -sin(θ) \\\
  sin(θ) & cos(θ)
\end{bmatrix}$，$R$ 是一个`正交矩阵`。因此，我们也可以直接通过转置获取其`逆矩阵`：$R^{-1} = \begin{bmatrix}
  cos(θ) & sin(θ) \\\
   -sin(θ) & cos(θ)
\end{bmatrix}$ 来进行`逆时针`旋转。

如果对点进行顺时针旋转，通过矩阵乘法运算：
$P'= \begin{bmatrix}
  cos(θ) & -sin(θ) \\\
  sin(θ) & cos(θ)
\end{bmatrix} \times \begin{bmatrix}x \\\ y\end{bmatrix} = \left[ \begin{alignat}{1}
(cos(θ)  \times x)\ &+&\ &(-&sin(θ) \times y) \\\
(sin(θ)  \times x)\ &+&\ &(&cos(θ) \times y)
\end{alignat}\right]$

#### 倾斜（skew）

倾斜（Skew）是一种剪切（Shearing）变换，它是改变图形的角度而不改变其面积的`仿射变换`。在二维空间中，剪切通常沿 X 轴或 Y 轴进行，对应的剪切矩阵分别如下：

**X轴剪切矩阵**: 当沿 X 轴进行剪切时，Y 轴上的点会沿 X 轴方向移动，移动的距离与其在 Y 轴上的位置成比例。

X 轴剪切矩阵表示为：
$
\begin{bmatrix}
1 & \text{skewX} \\\
0 & 1
\end{bmatrix}
$，
$P'= \begin{bmatrix}
1 & \text{skewX} \\\
0 & 1
\end{bmatrix} \times \begin{bmatrix}x \\\ y\end{bmatrix} = \begin{bmatrix}
x\ +skewX \times y \\\
y \\\
\end{bmatrix}$

**Y轴剪切矩阵**: 当沿 Y 轴进行剪切时，X 轴上的点会沿 Y 轴方向移动，移动的距离与其在 X 轴上的位置成比例。

Y 轴剪切矩阵表示为：
$
\begin{bmatrix}
1 & 0 \\\
\text{skewY} & 1
\end{bmatrix}
$，
$P'= \begin{bmatrix}
1 & 0 \\\
\text{skewY} & 1
\end{bmatrix} \times \begin{bmatrix}x \\\ y\end{bmatrix} = 
\begin{bmatrix}
x \\\
skewY \times x + y
\end{bmatrix}$

#### 基于不同原点的变换

上面的变换都是基于`原点`(0,0) 的变换，如果基于其它点进行缩放、旋转等。就需要先进行平移，然后变换，最后平移回来。

例如基于原点 (2,3) 的旋转矩阵变换表示：
$p'=\begin{bmatrix}
  cos(θ) & -sin(θ) \\\
  sin(θ) & cos(θ)
\end{bmatrix}*(\begin{bmatrix}x \\\ y\end{bmatrix} - \begin{bmatrix}2 \\\ 3\end{bmatrix}) + \begin{bmatrix}2 \\\ 3\end{bmatrix}$

### 组合变换和逆变换

用矩阵表示`线性变换`的一个主要动力就是可以很容易地进行`组合变换`以及`逆变换`。

组合可以通过`矩阵乘法`来完成。如果 `A` 与 `B` 是两个`线性变换`，那么对`向量 x` 先进行 `A 变换`，然后进行 `B 变换`的过程为：${\mathbf  {B}}({\mathbf  {A}}{\vec  x})=({\mathbf  {BA}}){\vec  x}$

能够通过两个矩阵相乘将两个变换组合在一起这样的能力就使得可以通过`逆矩阵`进行变换的`逆变换`。$A^{-1}$ 表示 $A$ 的逆变换。

### 仿射变换

`仿射变换`（Affine transformation）是一种保持直线和平行度的几何变换。对于上面的`(缩放/旋转/倾斜)+平移`的组合就是一种仿射变换。

仿射变换的示例包括`平移`、`缩放`、`同质性`、`相似性`、`反射`、`旋转`、`剪切映射`以及它们以任意组合和顺序的组合。

为了表示仿射变换，需要使用`齐次坐标`，即用三维向量 (x, y, 1) 表示二维向量。对于高纬来说也是如此。这样去处理，对于`平移变换`就可以通过`矩阵乘法`实现。

$\begin{bmatrix}
  x' \\\ y' \\\ 1
\end{bmatrix}
=\begin{bmatrix}
1 & 0 & tx \\\
0 & 1 & ty \\\
0 & 0 & 1 \\\
\end{bmatrix} \times \begin{bmatrix}
  x \\\ y \\\ 1
\end{bmatrix}=\begin{bmatrix}
  x +  tx \\\
  y +  ty \\\
  1
\end{bmatrix}$

### 齐次坐标

齐次坐标（Homogeneous Coordinates）会在原来的向量`增加`一个维度，并且它的值永远为 1。 一个二维向量的齐次坐标用矩阵表示为：
$\begin{bmatrix}
  x \\\ y \\\ 1
\end{bmatrix}$

在计算机图形学、计算机视觉和机器人学中，`齐次坐标`被广泛使用，主要原因是它们提供了一种在使用矩阵运算进行几何变换（如旋转、缩放、平移）时的便利性和统一性。

以下是使用齐次坐标的主要原因：

1. **统一各种变换**: 在非齐次坐标系中，`平移`不能用`矩阵乘法`表示，这与旋转和缩放不同。齐次坐标允许将平移、旋转和缩放统一为单一的`矩阵乘法`操作，从而简化了变换的表示和计算。
2. **方便矩阵运算**: 使用齐次坐标，可以将多个变换`组合`成一个矩阵，然后一次性应用于点或向量。这在计算机图形学中是非常有用的，因为它允许高效地处理复杂的场景和动画。
3. **表示无穷远点**: 在齐次坐标系统中，可以方便地表示和处理无穷远点。这对于计算机视觉和 3D 图形中的透视投影等应用至关重要。
4. **简化投影变换**: 齐次坐标使得处理投影变换变得更简单。在 3D 图形中，常常需要将 3D 场景投影到 2D 视图上，齐次坐标提供了一种有效的方式来实现这种投影。
5. **增强数学表达能力**: 在数学上，齐次坐标增强了坐标系统的表达能力，允许更加灵活地表示和处理空间中的点。

{{< video src="Affine_transformations.ogv" width=300 loop="true" >}}

### 组合缩放、平移、旋转三种变换

当使用齐次坐标后，这三种变换就可以通过矩阵乘法进行组合。

我们分别表示出这三种变换的增广矩阵：`平移` ($T$)、`旋转` ($R$)、`缩放` ($S$)。

$T =\begin{bmatrix}
  x \\\ y \\\ 1
\end{bmatrix}
=\begin{bmatrix}
1 & 0 & tx \\\
0 & 1 & ty \\\
0 & 0 & 1 \\\
\end{bmatrix}$，
$R = \begin{bmatrix}
  cos(θ) & -sin(θ) & 0 \\\
  sin(θ) & cos(θ) & 0 \\\
  0 & 0 & 1
\end{bmatrix}$，$S=\begin{bmatrix}
  sx & 0 & 0 \\\
  0 & sy & 0 \\\
  0 & 0 & 1
\end{bmatrix}$，然后组合它们得到了一个复合矩阵: 
$TRS =\displaystyle \left[\begin{matrix}sx \cos{\left(\theta \right)} & - sy \sin{\left(\theta \right)} & tx\\\sx \sin{\left(\theta \right)} & sy \cos{\left(\theta \right)} & ty\\\0 & 0 & 1\end{matrix}\right]$

参考资料：

\> [https://zh.wikipedia.org/wiki/变换矩阵](https://zh.wikipedia.org/wiki/变换矩阵)

\> [https://en.wikipedia.org/wiki/Affine_transformation](https://en.wikipedia.org/wiki/Affine_transformation)
