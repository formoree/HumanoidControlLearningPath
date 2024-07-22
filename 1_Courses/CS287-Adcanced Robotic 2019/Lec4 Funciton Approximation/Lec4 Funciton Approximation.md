## Function Approximation

![image-20240722140755198](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407221407289.png)

+ 因此他可以看成是一种监督学习，我们用神经网络来表示Function $V_\theta$

![image-20240722141025907](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407221410802.png)

![image-20240722141104541](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407221411421.png)

+ Universal Function Approximation Thnorem：当神经网络中的神经元越多，你想要的函数变化就越多，但是也同样会带来过拟合的问题。

![image-20240722141602214](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407221416043.png)

![image-20240722142404448](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407221424329.png)

## Value iteration with function approximation

![image-20240722143201959](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407221432422.png)

![image-20240722143234358](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407221432634.png)

+ 下面是上面例子的板书【下面的$v^{(1)}(x_2)$式子写错了，应该是$x_2$，在下面一张PPT中说明了】
+ 这个例子说明了：**function approximation存在着发散的可能。**

![image-20240722143129898](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407221431929.png)

![image-20240722143937645](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407221439677.png)

![image-20240722144454878](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407221444838.png)

+ 总的来说，这个图例说明了线性回归在处理复杂、非线性关系时的局限性。它强调了在选择模型时需要考虑数据的实际特征，并在必要时使用更复杂的非线性模型。

![image-20240722145214349](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407221452711.png)

## Policy iteration with function approximation

+ 这张图片解释了近似策略评估(Approximate Policy Evaluation)是一种收缩映射(contraction)，也就是说能够保证收敛。

+ 收缩映射：简单来说,一个映射f被称为收缩映射,如果满足以下条件:

  1. f是一个函数,将一个空间(比如欧几里得空间或者函数空间)中的点映射到另一个点。
  2. 对于空间中任意两点x和y,f(x)和f(y)之间的距离small于x和y之间的距离。

  也就是说,收缩映射会压缩空间中点与点之间的距离。

+ 在强化学习的策略评估中,Bellman更新和加权线性回归都被证明是收缩映射,这就保证了算法的收敛性。这就是图中提到的"收敛保证"。

![image-20240722145922806](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407221459748.png)

+ 下图主要是在说明不同的范数（如∞-范数和可能的2-范数或其他范数）在某些情况下可能会产生不兼容的结果。特别是，在某些方向上（如图中的对角线方向），使用不同的范数可能会导致计算发散或得到不同的结果。
+ 这张图展示了不同范数（如∞-范数和可能的2-范数）在某些情况下会产生不兼容的结果：
  1. 在∞-范数下，等距点形成正方形（如图中的方框）。
  2. 而在2-范数下，等距点会形成圆形（虽然图中没有明确画出）。
  3. 对角线方向上的点显示，使用不同范数可能导致不同的收敛行为，甚至发散。

![image-20240722150330195](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407221503087.png)

## Infinite Horizon Linear Program

+ 这张图片展示了一个无限时域线性规划问题，用于求解强化学习中的最优值函数。
+ 这个线性规划问题的目的是找到最优值函数V*。通过最小化初始状态分布下的期望值函数，同时满足Bellman方程的约束，我们可以得到最优策略对应的值函数。这种方法提供了一种理论上的方式来计算最优值函数，但在实际中，由于状态空间可能非常大或无限，直接求解这个线性规划问题可能在计算上是不可行的。

![image-20240722150758310](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407221507527.png)

+ 这张图片展示了无限时域线性规划问题的一个扩展和近似方法。
+ 函数近似：图片引入了函数近似的概念：Let V(s) = θ^T φ(s)，and consider S' rather than S。这里，V(s) 被表示为特征向量 φ(s) 和参数向量 θ 的内积。S' 可能是原始状态空间 S 的一个子集或采样。因此问题可以进行重新表述。

![image-20240722151002255](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407221510275.png)

+ 这张图片展示了近似线性规划方法的保证和收敛性质。

![image-20240722151042310](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407221510400.png)