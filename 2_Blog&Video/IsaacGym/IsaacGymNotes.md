> 部分参考[IsaacGym报错](https://zhuanlan.zhihu.com/p/679327032#:~:text=Not%20connec)
## no module isaacgym
+ cd isaacgym/pthon
+ pip install -e .


## Forcing CPU pipline

测试Isaacgym最简单的demo后发现问题：
```
(NIE) nie@nie-Yz:~/code/Locomotion_Baseline/isaacgym/python/examples$ python 1080_balls_of_solitude.py 
Importing module 'gym_38' (/home/nie/code/Locomotion_Baseline/isaacgym/python/isaacgym/_bindings/linux-x86_64/gym_38.so)
Setting GYM_USD_PLUG_INFO_PATH to /home/nie/code/Locomotion_Baseline/isaacgym/python/isaacgym/_bindings/linux-x86_64/usd/plugInfo.json
WARNING: Forcing CPU pipeline.
Not connected to PVD
+++ Using GPU PhysX
Physics Engine: PhysX
Physics Device: cuda:0
GPU Pipeline: disabled
段错误 (核心已转储)
```
+ **vulkan图形工具没有配置，注意配置的时候保证各个部件相互兼容！不然还是会出现segmentation fault的问题**
+ 我以为解决了，但是还是没有解决，然后将显卡驱动从560 -> 535 我真的是服了;
+ new: 降低显卡驱动版本，直接ok

### Vscode配置问题
+ 因为Ubuntu20.04中搜狗输入法与pycharm存在冲突【pycharm无法打开，一直闪退】
+ 我采用Vscode进行编码，但值得注意的是切换完虚拟环境之后--记得 ctrl+ shift + ` 生成新终端。这样python包才不会缺失。

### WARNING: lavapipe is not a conformant vulkan implementation
```
python 1080_balls_of_solitude.py 
Importing module 'gym_38' (/home/nie/code/Locomotion_Baseline/isaacgym/python/isaacgym/_bindings/linux-x86_64/gym_38.so)
Setting GYM_USD_PLUG_INFO_PATH to /home/nie/code/Locomotion_Baseline/isaacgym/python/isaacgym/_bindings/linux-x86_64/usd/plugInfo.json
WARNING: Forcing CPU pipeline.
Not connected to PVD
+++ Using GPU PhysX
Physics Engine: PhysX
Physics Device: cuda:0
GPU Pipeline: disabled
WARNING: lavapipe is not a conformant vulkan implementation, testing use only.
Unhandled descriptor set 433
Unhandled descriptor set 1182355248
Unhandled descriptor set 497
Unhandled descriptor set 65536
Segmentation fault (core dumped)

```

+ [vulkan安装 以及NVIDIA显卡驱动问题](https://blog.csdn.net/ggggfff1/article/details/135487322)
+ [测试安装vulkan](https://ubuntuqa.com/article/10482.html)