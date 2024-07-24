# Randomization and the reality gap:  how to transfer robotic policies from  sim to real

## Robotics is really hard

+ 很难获取现实世界精确的State
+ 对于不同任务，reward也很难设置
+ 动作的控制更难

![image-20240724150024421](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407241500637.png)

## Can deep learning + RL make robotics easy?

> 如果我们并不是理解我们的环境，而是简单地收集经验，并让算法来处理剩下的部分

挑战：

+ 数据需求高
  + Robot cost,Safety,Labeling
+ 高效RL
  + Model-based，meta-learning，learning from demonstrations

## Simulated data

> It’s hard to accurately & efficiently model sensors & physical systems

![image-20240724150506975](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407241505877.png)

![image-20240724150526024](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407241505834.png)

![image-20240724150549916](C:/Users/HP/AppData/Roaming/Typora/typora-user-images/image-20240724150549916.png)

![image-20240724150725462](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407241507254.png)

### Why is it so hard to use simulated training data?

![image-20240724150930215](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407241509760.png)

![image-20240724151006987](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407241510984.png)

![image-20240724151044379](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407241510646.png)

> Small modeling errors lead to large control errors

![image-20240724151215801](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407241512789.png)

### How can you use simulation without solving sim2real?

![image-20240724151255474](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407241512322.png)

![image-20240724151312882](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407241513579.png)

![image-20240724151326001](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407241513885.png)

![image-20240724151341318](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407241513762.png)

![image-20240724151432631](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407241514476.png)

![image-20240724152250203](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407241522038.png)

## Building a good simulation

![image-20240724152453231](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407241524974.png)

### Simulator design 

+ Pieter’s lecture from earlier in CS287 
+ In practice, most people pick Bullet / PyBullet or MuJoCo and use the  models provided by the developer of their robot

### World design

![image-20240724152610059](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407241526895.png)

![image-20240724152627344](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407241526193.png)

### Collect data & improve simulation

![image-20240724162332720](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407241623589.png)

![image-20240724162447641](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407241624426.png)

## Domain Adaptation (in sim2real)

### Supervised Domain Adaptation

![image-20240724162852725](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407241628570.png)

![image-20240724163110086](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407241631969.png)

## Domain Randomization

核心观点，如果我们在simulation中见识到了足够多的情况，那么我们在进入现实世界便会很容易处理其中的误差，现实世界就像是simulation的一种。

### Application

![image-20240724183918952](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407241839600.png)

### How does it work?

+ More data = better
+ More textures = better
+ Pre-training is not necessary

![image-20240724184111140](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407241841228.png)

![image-20240724184127342](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407241841209.png)

![image-20240724184310848](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407241843653.png)

![image-20240724184318584](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407241843520.png)

![image-20240724191826296](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407241918986.png)

### What is randomized?

What is randomized? 

+ Physical parameters 
+ Correlated and uncorrelated noise applied to policy inputs 
+ Sensor dropout 
+ Physics discretization timestep 
+ Backlash 
+ Random forces applied to the object 
+ Visual appearance

### Why does domain randomization work?

+ Intuition 1: training data comes from a **covering distribution**  
+ Intuition 2: DR is a way of specifying to the model **what to ignore**  
+ Intuition 3: DR is **meta-learning**

+ 需要实现下面的效果，我们需要大量的数据来实现。
+ 一些现实世界的影响无法在simulator中建模。

![image-20240724192217533](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407241922705.png)

![image-20240724192429671](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407241924485.png)

![image-20240724192556061](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407241925896.png)

### Challenges

![image-20240724192639407](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407241926280.png)

![image-20240724192715317](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407241927209.png)

### Extensions of domain randomization

![image-20240724192833581](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407241928490.png)

## What’s next?

![image-20240724192951134](https://raw.githubusercontent.com/formoree/PicGO-Picture/master/202407241929031.png)