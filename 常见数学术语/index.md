## 常见数学术语 
### 线性方程（linear equations）

线性方程也称`一次方程式`。指未知数都是一次的方程。其一般的形式是：$ax+by+...+cz+d=0$。

### 线性方程组（system of linear equations）

线性方程组为`线性方程`的集合。

$$
\cases{
2x +  y + 3z = 10 \\\ 
x + y + z = 6 \\\ 
x + 3y + 2z = 13
}
$$

### 向量方程（vector equation）

一个`解`（solution）是$x$，$y$，$z$ 值的集合，这些值使等式成立。

上面的线性方程组，也可以写成下面`向量方程`形式：

$$
\begin{bmatrix}2 \\\ 1 \\\ 1\end{bmatrix}x +
\begin{bmatrix}1 \\\ 1 \\\ 3\end{bmatrix}y +
\begin{bmatrix}3 \\\ 1 \\\ 2\end{bmatrix}z =
\begin{bmatrix}10 \\\ 6 \\\ 13\end{bmatrix}
$$

### 齐次方程（homogeneous）

如果右边的常量都为 0 ，则该方程组是`齐次的`（homogeneous），如下面方程：

$$
\begin{aligned}
3x_1 − 7x_2 + 4x_3 = 0 \\\ 
5x_1 + 8x_2 − 12x_3 = 0 \\\ 
\end{aligned}
$$

#### 零解（trivial solution）

$$
\begin{bmatrix}x_1 \\\ x_2 \\\ x_3\end{bmatrix} =
\begin{bmatrix}0 \\\ 0 \\\ 0\end{bmatrix}
$$

对于齐次方程，如果变量都为 0 ，我们称为零解，也可以叫平凡解，基本没啥功能意义。与之对应的解叫`非平凡解`（non-trivial solution）。

这个解没有任何意义，但是齐次方程在`线性代数`里确实是非常重要的.

#### 线性无关（linearly independent）

给定 $v_1,..., v_n$ 是一组向量（都在同一维度），若 $x_1v_1 + ··· + x_nv_n = 0$ `只有零解`，则这组向量可以叫做`线性无关`（linearly independent）。

例如方程组 $\begin{aligned}x_1 + 3x_2 = 0 \\\ x_1 + 2x_2 = 0\end{aligned}$中，向量 $\begin{bmatrix}1 \\\ 1\end{bmatrix}$ 和 $\begin{bmatrix}3 \\\ 2\end{bmatrix}$ 是线性无关的。

反之，对于方程组 $\begin{aligned}x_1 + 3x_2 = 0 \\\ 2x_1 + 6x_2 = 0\end{aligned}$，我们不仅可以找到`零解`，同时也能找到其它解 $x_1 = 3,\ x_2 = −1$，那么我们可以认为向量 $\begin{bmatrix}1 \\\ 2\end{bmatrix}$ 和 $\begin{bmatrix}3 \\\ 6\end{bmatrix}$ 是`线性相关`的。

### 解向量（solution vector）


解向量是`线性方程组`的一个解。

因为一组解在空间几何里可以表示为一个向量，所以叫做解向量。
