## MDP

![image-20240721152918595](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407211529982.png)

## Exact Solution Methods

### Value Iteration

![image-20240721162028035](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407211620449.png)

![image-20240721165843627](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407211658755.png)

![image-20240721165908064](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407211659008.png)

+ 给出两个定义：最大范数、最大范数的gamma收缩。然后我们得到定理：无论初始化如何，我们都可以收缩到固定的点。
+ 额外的事实：如果说每次的更新较小，那么应该是接近收敛了。

### Policy Evaluation

+ 下面所有的都是确定的。

![image-20240721171519565](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407211715715.png)

+ 我们将情况转到随机策略，policy evaluation的更新公式为2

![image-20240721171737436](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407211717189.png)

![image-20240721172150729](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407211721619.png)

### Linear Programming

![image-20240721180656790](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407211806701.png)

+ 对偶线性规划。如果其有最优解，那么原问题也有最优解。
+ 等式2中 状态概率函数 = 当前状态的概率 + 转移为当前状态概率。
+ 约束条件：我们采取任意一个策略，我们都会有对应的概率$\lambda(s',a')$

![image-20240721181052220](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407211810540.png)

##  Maximum Entropy Formulation

+ 值迭代最终会找到最佳决策，但是当world发生变化，比如我们新增了一个障碍，那么按照值迭代，之前所做的努力都不复存在。因此我们希望能够找到近似最优解的分布情况。
+ 因此我们引入熵这个数学量概念，它用来衡量变量X的不确定性。我们的最优策略会有较高的熵值。信息熵越大,表示信息越不确定,含有的信息量也越大。

![image-20240721173530574](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407211735502.png)

![image-20240721173622685](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407211736634.png)

![image-20240721173812574](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407211738416.png)

+ 其本质是一个约束优化问题，我们用拉格朗日乘子法进行解决

![image-20240721174110456](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407211741867.png)

+ 我们将上述式子扩展到Max-ent，因此我们能解出$\pi(a)$

![image-20240721174420580](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407211744607.png)

+ softmax = a soft way to find maximum

![image-20240721174720612](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407211747640.png)

+ 在这里的推导我们定义了Q值，并且在最终式子发现与上一页的式子相同，因此我们直接带入。

![image-20240721175842453](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407211758368.png)