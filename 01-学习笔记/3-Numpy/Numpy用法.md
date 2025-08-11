# `Numpy`

NumPy 是 Python 进行科学计算和数据分析的基石，几乎所有的数据科学库（如 Pandas、Scikit-learn、TensorFlow）都构建在它之上。

### **解析您代码中的 NumPy 用法**

```python
# 1. 创建一个 NumPy 数组
X_train = np.array([[1.0], [2.0]], dtype=np.float32)

# 2. 从数组中索引一个元素
X_train[0] # 这会得到第一个元素 [1.0]

# 3. 改变元素的形状
.reshape(1,1)
```

1.  `np.array([...])`: 这是 NumPy 最核心的命令，用于**创建一个 NumPy 数组**。
    *   它的输入是一个 Python 的列表 `[[1.0], [2.0]]`。这是一个包含两个列表的列表。
    *   NumPy 会将这个二维列表转换成一个二维数组（也就是一个矩阵）。`X_train` 现在看起来像这样：
        ```
        [[1.0],
         [2.0]]
        ```
    *   这个数组有 2 行 1 列。

2.  `dtype=np.float32`: 这是一个非常重要的参数，它明确地告诉 NumPy：
    *   `dtype` 代表 "data type" (数据类型)。
    *   我们希望数组中所有元素的**数据类型都是 32 位的浮点数** (`float32`)。
    *   在机器学习中，特别是使用 TensorFlow 或 PyTorch 时，经常需要指定精确的数据类型（如 `float32`）来匹配模型的要求并优化内存使用。

3.  `X_train[0]`: 这是在进行**数组索引**。
    *   它获取 `X_train` 数组中**索引为 0 的元素**，也就是第一行。所以 `X_train[0]` 的结果是 `[1.0]`。
    *   注意，此时 `[1.0]` 是一个一维数组，它的形状 (shape) 是 `(1,)`。

4.  `.reshape(1,1)`: 这是在**改变数组的形状**。
    *   `.reshape()` 是 NumPy 数组的一个方法，它可以在不改变数组数据的情况下，赋予它一个新的形状。
    *   我们将形状为 `(1,)` 的一维数组 `[1.0]` 重新塑形为 `(1, 1)`，也就是 1 行 1 列的二维数组（矩阵）。它现在变成了 `[[1.0]]`。
    *   在机器学习模型中，输入的维度通常要求是二维或更高维的（例如，(样本数, 特征数)），所以即使只有一个样本和一个特征，也需要将其 `reshape` 成 `(1, 1)` 的形式。
    *   .reshape(1, -1) 的意思是：“把我的数组转换成一个只有 1 行的二维数组，列数你帮我自动算。
    *   .reshape(-1, 1) 的意思是：“把我的数组转换成一个只有 1 列的二维数组，行数你帮我自动算。
    >例如： [10 20 30 40] Reshape(-1, 1)  后的形状: (4, 1)
    > [[10]  
    > [20]  
    > [30]  
    > [40]]  
    
---

### **NumPy 常用用法系统整理**

下面是 NumPy 最核心和最常用功能的分类整理。

#### **1. 创建数组 (Array Creation)**

这是使用 NumPy 的第一步。

*   **从 Python 列表创建**:
    ```python
    # 一维数组
    a = np.array([1, 2, 3, 4])
    # >> [1 2 3 4]

    # 二维数组 (矩阵)
    b = np.array([[1, 2, 3], [4, 5, 6]])
    # >> [[1 2 3]
    #     [4 5 6]]
    ```

*   **创建占位符数组**: 有时需要先创建特定大小和类型的数组，再填充内容。
    *   `np.zeros((rows, cols))`: 创建一个全为 0 的数组。
        ```python
        zeros_array = np.zeros((2, 3)) # 2行3列的全0数组
        # >> [[0. 0. 0.]
        #     [0. 0. 0.]]
        ```    *   `np.ones((rows, cols))`: 创建一个全为 1 的数组。
    *   `np.arange(start, stop, step)`: 类似 Python 的 `range`，但创建的是 NumPy 数组。
        ```python
        c = np.arange(0, 10, 2) # 从0开始，到10结束（不含10），步长为2
        # >> [0 2 4 6 8]
        ```
    *   `np.linspace(start, stop, num)`: 创建一个包含 `num` 个元素的等差数列。
        ```python
        d = np.linspace(0, 10, 5) # 从0到10，均匀取5个数
        # >> [ 0.   2.5  5.   7.5 10. ]
        ```
    *   `np.random.rand(rows, cols)`: 创建一个指定形状的、值为 0 到 1 之间随机数的数组。

#### **2. 数组的属性 (Array Attributes)**

了解数组的基本信息。

假设我们有数组 `b = np.array([[1, 2, 3], [4, 5, 6]])`

*   `b.shape`: 查看数组的形状（维度）。
    ```python
    print(b.shape)
    # >> (2, 3)  表示2行3列
    ```
*   `b.ndim`: 查看数组的维数。
    ```python
    print(b.ndim)
    # >> 2  表示是二维数组
    ```
*   `b.size`: 查看数组中元素的总个数。
    ```python
    print(b.size)
    # >> 6
    ```
*   `b.dtype`: 查看数组元素的数据类型。
    ```python
    print(b.dtype)
    # >> int32  (在我的系统上默认为32位整数)
    ```

#### **3. 数组索引与切片 (Indexing and Slicing)**

这是 NumPy 最强大和最常用的功能之一，与 Python 列表的索引类似但更强大。

假设我们有数组 `a = np.arange(10)`，结果是 `[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]`

*   **单个元素**: `a[2]` 会得到 `2`。
*   **切片**: `a[2:5]` 会得到 `[2, 3, 4]` (从索引2到5，不含5)。
*   **布尔索引 (Boolean Indexing)**: 使用条件来过滤数组，**非常非常有用**！
    ```python
    # 找到 a 中所有大于 5 的元素
    condition = a > 5
    # >> [False False False False False False  True  True  True  True]

    print(a[condition])
    # >> [6 7 8 9]

    # 也可以直接写成一行
    print(a[a > 5])
    ```

对于二维数组 `b = np.array([[1, 2, 3], [4, 5, 6]])`：
*   **单个元素**: `b[row, col]`
    ```python
    print(b[1, 2])
    # >> 6 (第2行第3列，索引从0开始)
    ```
*   **切片**:
    ```python
    # 获取第一行
    print(b[0, :])  # : 代表该维度的所有元素
    # >> [1 2 3]

    # 获取第二列
    print(b[:, 1])
    # >> [2 5]
    ```

#### **4. 数组运算 (Operations)**

NumPy 的核心优势是**元素级运算 (element-wise operations)**，即对数组中的每个元素同时执行操作，比 Python 的 for 循环快得多。

*   **数学运算**:
    ```python
    a = np.array([1, 2, 3])
    b = np.array([4, 5, 6])

    # 元素级加法
    print(a + b)
    # >> [5 7 9]

    # 元素级乘法
    print(a * b)
    # >> [ 4 10 18]

    # 与单个数字运算 (广播 Broadcasting)
    print(a + 100)
    # >> [101 102 103]
    ```

*   **通用函数 (Universal Functions)**:
    ```python
    print(np.sin(a))    # 计算数组a中每个元素的正弦值
    print(np.exp(a))    # 计算数组a中每个元素的指数
    ```

*   **聚合函数 (Aggregation Functions)**:
    ```python
    c = np.array([[1, 2, 3], [4, 5, 6]])

    print(c.sum())      # 所有元素求和 >> 21
    print(c.min())      # 所有元素中的最小值 >> 1
    print(c.max())      # 所有元素中的最大值 >> 6
    print(c.mean())     # 所有元素的平均值 >> 3.5

    # 也可以按轴(axis)进行计算
    # axis=0 表示按列操作
    print(c.sum(axis=0)) # >> [5 7 9] (1+4, 2+5, 3+6)
    # axis=1 表示按行操作
    print(c.sum(axis=1)) # >> [ 6 15] (1+2+3, 4+5+6)
    ```

#### **5. 改变数组形状 (Reshaping)**

除了您用到的 `.reshape()`，还有一个常用的函数是 `.T` (转置)。

```python
b = np.array([[1, 2, 3], [4, 5, 6]])
# b 的 shape 是 (2, 3)

# 转置
print(b.T)
# >> [[1 4]
#     [2 5]
#     [3 6]]
# 转置后的 shape 是 (3, 2)
```

希望这份整理能帮助您更好地理解和使用 NumPy！它是数据科学之旅中最重要的工具之一。