## Optimal control for Linear Dynamical Systems and Quadratic Cost(LQR)

> 多看，学得很懵

+ 贝尔曼维数灾难
  + 即使使用可变分辨率离散化和高度优化的实现，离散化通常只在5或6维状态空间内在计算上可行。
  + 函数近似可能有效也可能无效，在实践中通常是局部有效的。

![image-20240723175045979](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407231750378.png)

+ 最优控制用于线性动态系统和二次代价(又称LQ设置或LQR设置)
  + 这是一个非常特殊的情况:可以精确解决连续状态空间最优控制问题,且只需执行线性代数运算。
  + 运行时间复杂度: O(H n³),其中H可能代表时间范围,n可能代表状态空间的维度.

![image-20240723175233016](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407231752653.png)

### LQR

+ LQR 是一种用于控制线性动态系统的最优控制方法。其目标是通过选择适当的输入 $𝑢_𝑡$，使得系统在满足动态方程的同时，最小化累积的二次代价函数。

![image-20240723180459589](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407231805494.png)

+ 我们将LQR扩展到非线性动态系统

![image-20240723181642150](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407231816366.png)

![image-20240723181745311](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407231817961.png)

![image-20240723181858403](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407231819032.png)

+ 这个算法通过迭代计算来找到LQR问题的最优解。它从初始猜测开始，逐步改进控制策略和值函数估计，直到收敛到最优解。这种方法特别适用于线性系统和二次代价函数，因为它能保证在满足特定条件时收敛到全局最优解。

![image-20240723182321988](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407231823132.png)

+ 对上述情况进行扩展
+ 重新定义更新状态公式，它跟之前情况非常相似。

![image-20240723182429178](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407231824527.png)

+ 系统中加入噪音

![image-20240723182657684](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407231826767.png)

![image-20240723182747686](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407231827474.png)

## Most General Case

![image-20240723185622640](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407231856445.png)

![image-20240723185656097](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407231856981.png)

![image-20240723185726886](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407231857855.png)

![image-20240723185923797](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407231859863.png)

## DDP

![image-20240723190023133](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407231900032.png)![image-20240723190037626](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407231900895.png)