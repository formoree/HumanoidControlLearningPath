> 当状态是连续的，值迭代就变得不太实际

![image-20240722121534152](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407221215550.png)

+ 我们尝试将整个过程离散化

![image-20240722122023701](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407221220808.png)

### Discretization

+ 将复杂的连续问题简化为可解的离散问题,但可能会损失一些精度。
+ 如果(状态,动作)对可能导致无限多(或非常多)的下一个状态：建议从下一状态分布中采样（以此来处理连续MDP问题）。

![image-20240722123235575](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407221232674.png)

+ 另一种离散化方法：这种方法比简单的"吸附到最近顶点"更灵活，因为它允许状态在多个邻近顶点之间进行概率分布。这可以更好地捕捉连续状态空间中的不确定性和动态特性。

![image-20240722123451229](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407221234000.png)

![image-20240722123531552](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407221235441.png)

+ 在上面我们介绍了两种离散化连续MDP问题的方法，但是现在存在两个问题：
  + 如果一个状态不在离散状态中，我们如何执行动作？
  + 获得的策略和价值函数与最优解有多接近？
+ 我们可以使用（1）最近邻居的方法和（2）随机插值的方法【对于不在离散化集合中的状态s，我们通过周围的离散状态来近似表示它。】

![image-20240722124200714](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407221242840.png)

![image-20240722124341365](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407221243243.png)

![image-20240722124414150](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407221244260.png)

### Cross-entropy method（CEM）

![image-20240722124743322](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407221247480.png)