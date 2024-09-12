> 因为上一篇介绍太长，因此拆分为两篇。
> 这篇主要是对[IsaacGym](https://junxnone.github.io/isaacgymdocs/programming/simsetup.html)阅后记录，部分会加上自己的理解。

### 添加Viewer
添加一个简单的Viewer，可以让我们在测试和运行中查看发生的情况。我们可以这样创建Viewer：
```python
#  向CameraProperties导入变量，改变窗口参数
cam_props = gymapi.CameraProperties()
viewer = gym.create_viewer(sim, cam_props)

# 更新Viewer
gym.step_graphics(sim);
gym.draw_viewer(viewer, sim, True)

# 将视觉更新频率与实时同步 loop结束之后添加
gym.sync_frame_time(sim)
```
+ step_graphics 方法将仿真的视觉表示与物理状态同步。
+ draw_viewer 方法在查看器中呈现最新快照。
+ 它们是单独的方法，因为即使在没有查看器的情况下也可以使用step_graphics，例如在渲染相机传感器时。

如果您希望在查看器窗口关闭时终止模拟，则可以将循环条件设置为 query_viewer_has_closed 方法，该方法将在用户关闭窗口后返回 True。Viewer的基本模拟循环如下：
```python
while not gym.query_viewer_has_closed(viewer):

    # step the physics
    gym.simulate(sim)
    gym.fetch_results(sim, True)

    # update the viewer
    gym.step_graphics(sim);
    gym.draw_viewer(viewer, sim, True)

    # Wait for dt to elapse in real time.
    # This synchronizes the physics simulation with the rendering rate.
    gym.sync_frame_time(sim)
```

### 认识Viewer GUI
当查看器处于焦点状态时，可以使用 F11 切换查看器的全屏模式。
创建查看器后，屏幕左侧将显示一个简单的图形用户界面。可以使用 'Tab' 键打开和关闭 GUI 的显示。
+ Actors 选项卡提供了选择环境和该环境中的参与者的功能。当前选定的角色有三个单独的子选项卡。
  + Bodies 子选项卡提供有关活动角色的刚体的信息。它还允许更改角色实体的显示颜色和切换实体轴的可视化。
  + DOFs（自由度）子选项卡显示有关活动角色的自由度的信息。DOF 属性可以使用用户界面进行编辑，请注意，这是一个实验性功能。
  + Pose Override（姿势覆盖）子选项卡可用于使用其自由度手动设置角色的姿势。启用此功能后，将使用滑块在用户界面中设置的值覆盖所选角色的姿势和驱动目标。它可以是一个有用的工具，用于以交互方式探索或操纵 actor 的自由度。
+ Sim 选项卡显示物理模拟参数。这些参数因模拟类型（PhysX 或 Flex）而异，并且可以由用户修改。
+ Viewer 选项卡允许自定义常见的可视化选项。一个值得注意的功能是能够在查看物体的图形表示和物理引擎使用的物理形状之间切换。这在调试物理特性行为时非常有用。
+ Perf （性能） 选项卡显示Gym内部测量的性能。顶部滑块“Performance Measurement Window（性能测量窗口）”指定了测量性能的帧数。帧速率报告上一个测量窗口的平均每秒帧数 （FPS）。其余性能度量报告为指定帧数上每帧的平均值。
    + Frame Time（帧时间）是从一步开始到下一步开始的总时间
    + Physics simulation time（物理模拟时间）是物理求解器运行的时间。
    + Physics Data Copy（物理数据复制）是复制仿真结果所花费的时间。
    + Idle time 通常在 gym.sync_frame_time（sim）范围内。
    + Viewer Rendering Time 是渲染和显示查看器所花费的时间。
    + Sensor Image Copy time 是将传感器图像数据从 GPU 复制到 CPU 所花费的时间。
    + Sensor Image Rendering time 是将相机传感器（不包括查看器相机）渲染到 GPU 缓冲区所花费的时间。