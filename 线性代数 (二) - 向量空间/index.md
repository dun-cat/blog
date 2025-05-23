## 线性代数 (二) - 向量空间 
### 向量

在代数中，向量是一个基本的数学对象，可以在许多不同的数学领域和应用中找到，从线性代数到物理学，再到工程学。向量的概念可以有多种不同的形式和解释，但在最常见的意义上，它是一个表示在空间中方向和大小的量。

1. **基本定义**:
   - 一个向量通常表示为一组有序的数，称为元素或分量。
   - 在几何上，向量通常表示为一个箭头，其方向和长度（或大小）分别代表向量的方向和大小。

2. **表示方式**:
   - 向量可以在`笛卡尔坐标系`中用`坐标点`表示，如二维向量 $ \mathbf{v} = (v_1, v_2) $ 或三维向量 $ \mathbf{v} = (v_1, v_2, v_3) $。
   - 向量也可以作为从原点到一个特定点的有向线段表示。

3. **操作**:
   - 向量可以通过各种方式操作，包括加法、减法、标量乘法等。
   - 向量的加法遵循平行四边形法则或三角形法则。
   - 标量乘法是指将向量的每个分量乘以一个标量值。

4. **应用**:
   - 在物理学中，向量被用来表示`速度`、`加速度`、`力`等。
   - 在工程和计算机科学中，向量用于表示`方向`、`位置`、`转换`等。

### 向量空间

`向量`的`线性组合`是构建`线性空间`（或`向量空间`）的基础。
一组`向量`的`线性组合`可以生成一个`向量空间`，其中的每个向量都可以通过这组`基向量`的线性组合来表示。

### 向量的矩阵表示

向量的矩阵表示是线性代数中的一个基本概念，它允许使用矩阵运算来处理向量。

在这个框架下，一个向量可以表示为一个矩阵，这个矩阵通常是一列，其中包含向量的各个`分量`。以下是这个概念的具体说明：

**列向量**: 一个 $ n $ 维向量 $ \mathbf{v} $ 可以表示为一个 $ n \times 1 $ 的`列矩阵`（列向量）：$
     \mathbf{v} = \begin{pmatrix} v_1 \\\ v_2 \\\ \vdots \\\ v_n \end{pmatrix}
     $
     其中，$ v_1, v_2, \ldots, v_n $ 是向量的分量。

**行向量**:同样地，向量也可以表示为一个`行矩阵`（行向量），特别是在某些运算中，如点积运算：$
     \mathbf{v}^T = \begin{pmatrix} v_1 & v_2 & \cdots & v_n \end{pmatrix}
     $
     其中 $ \mathbf{v}^T $ 表示 $ \mathbf{v} $ 的`转置`（Transpose）。

**矩阵运算**:将向量表示为矩阵允许使用矩阵运算来执行诸如向量加法、标量乘法、点积、叉积（在三维空间中）等操作。

例如，两个向量的点积可以表示为一个`行向量`与一个`列向量`的`矩阵乘积`。

**向量与矩阵的乘法**: 一个向量可以与一个矩阵相乘，产生另一个向量。这在进行线性变换时特别有用，如旋转、缩放或映射到新的坐标系。

### 转置矩阵

对矩阵进行`行列交换`后的矩阵就是`转置矩阵`。计算一个矩阵的转置矩阵是一个简单直观的过程。转置操作涉及将矩阵的行换成列，或者将其列换成行。

假设有一个矩阵：$
     A = \begin{pmatrix}
     a & b \\\
     c & d \\\
     e & f
     \end{pmatrix}
     $
     它的转置 $ A^T $ 是：
     $
     A^T = \begin{pmatrix}
     a & c & e \\\
     b & d & f
     \end{pmatrix}
     $

### 正交矩阵

**定义**:

- 一个方阵 $ A $ 被称为`正交矩阵`，那么它的`转置矩阵` $ A^T $ 与它自己的`乘积`等于`单位矩阵` $ I $。数学上表示为：$ A^T A = AA^T = I $
- 这意味着矩阵 $ A $ 的行向量和列向量都是`标准正交`的，即它们的长度为 `1`（单位向量）并且相互正交（`内积`为 `0`）。

**性质**:

- **保持长度和角度**：正交矩阵在变换时保持向量的长度和夹角不变。这在物理学和工程学中特别重要，例如，在描述刚体运动时。
- **逆矩阵等于转置矩阵**：正交矩阵的逆矩阵等于它的转置矩阵。这使得计算和理解它们的逆变换变得相对简单。
     $ A^{-1} = A^T $
- **行列式的值**：正交矩阵的`行列式`的绝对值总是 `1`。这意味着它们要么是保持方向（行列式为 1 ），要么是反转方向（行列式为 -1）的变换。
