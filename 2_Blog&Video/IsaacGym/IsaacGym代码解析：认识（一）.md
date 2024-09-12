### 前言
最近刚好做了IsaacGym相关的项目，借此机会将项目中的一些点整理一下。在做项目的途中，我发现环境配置的博客很多，但是代码分析的比较少，至少从我搜索内容来看是这样。
因此就想依据搜索到的寥寥几篇博客以及NVIDIA官方文档分析一下IsaacGym的代码，包括一些函数接口和重要变量。
在此之前我想吐槽一下代码感受，声明我是第一次做IsaacGym平台的项目，部分想法可能比较稚嫩，有些可能是原代码结构问题，也有可能是博主自己问题。
+ 单智能体大框架的算法逻辑比多智能体简单太多了，基本是[action->step->obs, reward]这样的循环。即使加上仿真环境，也只是花一些时间熟悉标准接口和物理仿真流程（这里就涉及一些机器人学方面的知识）。
+ 我在学习代码最开始感觉到不舒服的主要是函数分布比较凌乱，没有系统的结构：因为环境相关文件继承了两次，每次进行跳转的时候可能不是这个位置，不知道是Vscode设置问题还是本来就这样。然后正常来说在子类或者父类写好就行，但是好几个函数都是超类、子类、父类零散分布，因为是继承函数名称也一样，真的第一次看思路混乱。
+ Debug有的时候找函数也麻烦，如果跳转不到正确的位置，就要去三个文件里面Ctrl + F。

当前配置环境为：Ubuntu 20.04 + IsaacGym preview4[直接在官网下载压缩包解压]。在配置上有比较小众的问题可以看看[IsaacGym 配置问题汇总](https://zhuanlan.zhihu.com/p/719056771)

### 入门理解
[强化学习环境ISAAC GYM入门](https://zhuanlan.zhihu.com/p/679988736)写得很详细，从一个小项目入手介绍整个过程必要的函数以及物理属性。
我简要总结一下整个流程：
+ 模型建立：一般在assets/xml文件夹下面
+ 环境文件建立：重点关注gymapi, gymtorch, gym.acquire_gym()
  + create_sim() 环境基础设定，包括坐标系
  + _create_ground_plane() 建立地面环境
  + _create_envs() 建立环境参数
  + pre_physics_step() 仿真前的计算内容
  + post_physics_step() 最重要，计算奖励、更新智能体状态
    + compute_reward()
    + conpute_observations()
  + reset_idx() 环境重置的内容
+ 添加训练和环境参数
  + 增加yaml文件
  + 利用注册方法进行环境配置+注册[Class task_registry]
+ 训练
  + arguments() 制定执行超参数
  + wandb, swanlab 记录训练信息

### 认识IsaacGym
+ IsaacGym 与其他类似的“健身房”风格系统不同，在 Isaac Gym 中，模拟可以在 GPU 上运行，将结果存储在 GPU 张量中，而不是将它们复制回 CPU 内存。
+ 核心功能
  + 支持导入 URDF 和 MJCF 文件
  + 用于评估环境状态和应用操作的 GPU 加速张量 API
  + 支持各种环境传感器 - 位置、速度、力、扭矩等
  + 物理参数的运行时域随机化
  + 雅可比矩阵/逆运动学支持
+ Isaac Gym 与 Omniverse 和 Isaac Sim 有何关系？
  + Omniverse 是一个开放的虚拟协作平台，支持多种应用，包括 Isaac Sim。Isaac Sim 是基于 Omniverse 的机器人仿真工具，结合了物理仿真和高质量渲染，旨在为机器人开发提供真实的环境。
  + Isaac Gym 中的渲染相对基础，不支持光线追踪或 Omniverse 中提供的更复杂的合成数据传感器。


### 本文参考链接
[IsaacGym](https://junxnone.github.io/isaacgymdocs/programming/simsetup.html)