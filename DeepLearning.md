# 构建一个神经网络
![layer](/images/DeepLearning/image-0.png)  
![输入数据](/images/DeepLearning/image-1.png)
``` 
#定义layer：第一种方法
layer_1 = Dense(units=3,activation="sigmoid")  
layer_2 = Dense(units=1,activation="sigmoid")  
model=Sequential([layer_1,layer_2])

#定义layer：第二种方法 
model=Sequential([
    Dense(units=3,activation="sigmoid"),
    Dense(units=1,activation="sigmoid")])

#数据集
x=np.array([[200,17],[120,5],[425,20],[212,18]])
y=np.array([1,0,0,1])

#调用
model.compile(...)
model.fit(x,y)

#预测
model.predict(x_new)  
```

# 前向传播 Numpy
![前向传播](/images/DeepLearning/image-3.png)
```  
#layer实现 
def dense(a in,w,b, g):
    units =w.shape[1]
    a_out = np.zeros(units)
    for j in range(units):
        w = W[:,j]
        z=np.dot(w,a_in)+ b[j]
        a_out[j]= g(z)
    return a out

#顺序实现
def sequential(x):
    a1=dense(x，W1，b1)
    a2 =dense(a1,W2,b2)
    a3=dense(a2，W3，b3)
    a4= dense(a3,W4,b4)
    f_x= a4
    return f_x
```

# 神经网络密集层计算
```
x=np.array([200，17])
W=np.array([[1,-3,5],[-2,4,-6]])
b=np.array([-1，1，2])

#for 循环实现
def dense(a in,W,b):
    a_out = np.zeros(units)
    for j in range(units):
    w = W[:,j]
    z= np.dot(w,x)+ b[j]
    a[j]=g(z)
    return a

#向量化实现   
def dense(A in,W,B):
    z=np.matmul(A_in,W)+ B
    A_out=g(Z)
    return out
```

# TensorFlow 实现
```
import tensoxflow as tf
from tensorflow.keras import Sequential
from tensorflow.keras.layers import Dense

#编译
model=Sequential([
    Dense(units=25,activation='sigmoid'),
    Dense(units=15,activation='sigmoid'),
    Dense(units=1,activation='sigmoid')])

#损失函数（稀疏分类交叉熵）
from tensorflow.keras.losses import BinaryCrossentropy
model.compile(loss=BinaryCrossentropy())

#使用模型及损失函数
model.fit(X,Y,epochs=100)
```

# 训练过程
![训练过程](/images/DeepLearning/image-4.png)

# 激活函数选择  
## 输出层（out layer）
取决于任务类型和标签Y的格式,核心目标是将网络的原始输出转换为我们需要的最终格式。
1. 二分类问题 (Binary Classification)
- 场景: 判断一个样本是否属于某个类别，例如邮件是否为垃圾邮件,标签 y 通常是 0 或 1。
- 激活函数: Sigmoid 函数
- 原因: Sigmoid 函数能将任意实数输入压缩到 (0, 1) 区间内。这个输出值可以被直观地解释为“样本属于正类（标签为1）的概率”。例如，输出 0.8 意味着模型有 80% 的把握认为样本是正类。
- 常用损失函数: 二元交叉熵 (Binary Cross-Entropy)
  
2. 多分类问题 (Multi-class Classification)
- 场景: 从多个互斥的类别中选择一个，例如手写数字识别（0-9）、图像分类（猫、狗、鸟）。
- 激活函数: Softmax 函数
- 原因: Softmax 函数可以将一个包含任意实数的向量，转换为一个所有元素之和为 1 的概率分布向量。向量中的每个值代表了样本属于对应类别的概率，非常适合“多选一”的场景。
- 常用损失函数: 分类交叉熵 (Categorical Cross-Entropy)
3. 回归问题 (Regression)
- 场景: 预测一个连续的数值，而不是一个类别。
- 激活函数:
  1. 预测任意范围的数值:
   - 例子: 预测股价变化（可正可负）、温度变化。
   - 激活函数: 线性函数 (Linear Activation)，或者说不使用激活函数。
   - 原因: 模型需要能够输出任意大小的正数或负数，线性函数 f(x) = x 不会对输出做任何改变，正好满足这个需求。
   2. 仅预测非负数值:
   - 例子: 预测房价、商品销量、股票价格（价格本身）。
   - 激活函数: ReLU 函数 (Rectified Linear Unit)
   - 原因: ReLU 函数 f(x) = max(0, x) 的特性是，当输入大于0时，输出等于输入；当输入小于或等于0时，输出为0。这确保了模型的最终预测值永远不会是负数。 
  
![out layer:activation](/images/DeepLearning/image-5.png)
## 隐藏层（hidden layer）
对于隐藏层，选择的自由度更大，但经过多年的实践，业界已经形成了强烈的共识。在现代深度学习中，ReLU 是隐藏层的默认和首选激活函数（除非y输出是2值选择sigmoid）。
原因如下：
1. 计算效率高
   - ReLU: 其计算仅仅是一个取最大值的操作 max(0, x)，计算速度非常快。
   - Sigmoid/Tanh: 涉及到指数运算 (e^x)，计算成本远高于 ReLU。在深层网络中，这种效率差异会被放大。
2. 有效缓解梯度消失问题
   - 问题描述: 在深度网络中，使用 Sigmoid 或 Tanh 函数时，如果输入值过大或过小，函数曲线会进入“饱和区”（变得非常平坦）。在这些区域，函数的导数（梯度）接近于0。在反向传播过程中，这些接近于0的梯度会层层相乘，导致传到浅层网络的梯度变得无限小，使浅层网络的参数几乎无法更新，这就是“梯度消失”。
   - Sigmoid 的问题: 两端都有饱和区，很容易导致梯度消失。
   - ReLU 的优势:
        * 当输入大于0时，其导数恒为1。这使得梯度可以顺畅地在网络中流动，不会因为连乘而衰减。
        * 虽然在输入小于0时，其梯度为0（也会导致神经元“死亡”的问题，但有 Leaky ReLU 等变体来解决），但它只有一个饱和区，相比 Sigmoid 已经极大地改善了梯度消失问题，使得训练深层网络成为可能。
   
![hidden layer:activation](/images/DeepLearning/image-6.png)

# Softmax 
![softmax](/images/DeepLearning/image-7.png)
![softmax-cost](/images/DeepLearning/image-8.png) 

# 手写数字识别（MNIST with softmax） 
1. 基础实现  
![MNIST](/images/DeepLearning/image-9.png)  
2. Softmax改进
![softmax改进](/images/DeepLearning/image-10.png)