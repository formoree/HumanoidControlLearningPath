### 命令行常用选项
> 了解即可，一般我们会用argument模块自定义需求
+ --help 打印出每个示例的命令行选项。、
+ --physx 使用 PhysX 作为模拟的物理后端。
+ --flex 使用 FleX 作为模拟的物理后端。
+ --sim_device 选择用于使用类似 PyTorch 的语法运行模拟的设备。可以是 cpu 或 cuda，具有可选的设备规格。默认值为 cuda：0 。
+ --pipeline 选择 cpu 或 gpu 管道进行张量运算。默认值为 gpu。
+ --graphics_device_id 指定用于图形的设备序号。

同时你可以访问官方网站[Programming Examples](https://junxnone.github.io/isaacgymdocs/examples/simple.html)，里面展示了几个python示例项目，让你熟悉Isaac的用法，项目位于在压缩包IsaacGym/python/examples文件夹下。

### 仿真设置
+ Isaac Gym 的 API 设计为过程性和面向数据，而不是面向对象，这样可以更高效地在用 C++ 编写的核心部分和用 Python 编写的脚本之间传递信息。
+ Isaac Gym 通过简单的数据数组而非复杂的对象结构来表示模拟状态和控制，这使得使用 Python 的 NumPy 库处理数据变得容易且高效。此外，模拟数据可以作为 CPU 或 GPU 的张量访问，从而能与常用的深度学习框架（如 PyTorch）配合使用，简化模型训练和推理。
```python
# 核心 API（包括支持的数据类型和常量）
from isaacgym import gymapi 
# 所有 Gym API 函数都可以作为启动时获取的单例 Gym 对象的方法进行访问
gym = gymapi.acquire_gym() 
```

#### 创建模拟
gym对象本身作用不大，它仅用作 Gym API 的代理。要创建模拟，您需要调用 create_sim 方法：
```python
sim_params = gymapi.SimParams()
sim = gym.create_sim(compute_device_id, graphics_device_id, gymapi.SIM_PHYSX, sim_params)
```
+ sim 对象包含物理和图形上下文，允许您加载资产、创建环境并与模拟交互。
+ gymapi.SIM_PHYSX：指定您希望使用的物理后端。目前，选择是 SIM_PHYSX 或 SIM_FLEX
  + PhysX 后端提供可在 CPU 或 GPU 上运行的强大刚体和关节模拟。它是目前唯一完全支持新 tensor API 的后端。
  + Flex 后端提供完全在 GPU 上运行的软体和刚体模拟，但它尚不完全支持 tensor API。
+ sim_params：仿真参数允许您配置**物理仿真的详细信息**。这些设置可能会因物理引擎和要模拟的任务的特征而异。选择正确的参数对于仿真稳定性和性能非常重要。以下代码段显示了物理参数的示例：
```python
# get default set of parameters
sim_params = gymapi.SimParams()

# set common parameters
sim_params.dt = 1 / 60
sim_params.substeps = 2
sim_params.up_axis = gymapi.UP_AXIS_Z
sim_params.gravity = gymapi.Vec3(0.0, 0.0, -9.8)

# set PhysX-specific parameters
sim_params.physx.use_gpu = True
sim_params.physx.solver_type = 1
sim_params.physx.num_position_iterations = 6
sim_params.physx.num_velocity_iterations = 1
sim_params.physx.contact_offset = 0.01
sim_params.physx.rest_offset = 0.0

# set Flex-specific parameters
sim_params.flex.solver_type = 5
sim_params.flex.num_outer_iterations = 4
sim_params.flex.num_inner_iterations = 20
sim_params.flex.relaxation = 0.8
sim_params.flex.warm_start = 0.5

# create sim with these parameters
sim = gym.create_sim(compute_device_id, graphics_device_id, physics_engine, sim_params)
```

##### 创建地平面 Ground Plane
大多数模拟都需要地平面，除非它们在零重力下进行。我们可以按如下方式配置和创建接地层：
```python
# configure the ground plane
plane_params = gymapi.PlaneParams()
plane_params.normal = gymapi.Vec3(0, 0, 1) # z-up!
# the distance of the plane from the origin
plane_params.distance = 0
plane_params.static_friction = 1
plane_params.dynamic_friction = 1
# control the elasticity of collisions with the ground plane (amount of bounce)
plane_params.restitution = 0

# create the ground plane
gym.add_ground(sim, plane_params)
```

##### 加载资源 Assets
Gym 目前支持加载 URDF 和 MJCF 文件格式。加载资源文件将创建一个 GymAsset 对象，该对象包括所有实体、碰撞形状、视觉附件、关节和自由度的定义。某些格式也支持柔体和粒子。
```pyhton
# 路径的拆分是必要的，方便导入的时候搜索外部参考资料
asset_root = "../../assets"
asset_file = "urdf/franka_description/robots/franka_panda.urdf"
asset = gym.load_asset(sim, asset_root, asset_file)
```
+ load_asset 方法使用文件扩展名来确定资产文件格式。支持的扩展名包括用于 URDF 文件的 .urdf 和用于 MJCF 文件的 .xml。
+ 我们也可以通过 AssetOptions 参数将额外的信息传递给 asset inmporter。
+ 加载资产不会自动将其添加到模拟中。GymAsset 用作 actor 的**蓝图**，可以在具有不同姿势和个性化属性的模拟中多次实例化。
```python
asset_options = gymapi.AssetOptions()
asset_options.fix_base_link = True
asset_options.armature = 0.01

asset = gym.load_asset(sim, asset_root, asset_file, asset_options)
```

##### 环境和Actor
**环境由一组一起模拟的 actor 和传感器组成。** 环境中的 Actor 彼此进行物理交互。它们的状态由物理引擎维护，并且可以使用稍后讨论的控制 API 进行控制。放置在环境中的传感器（如摄像机）将能够捕获该环境中的 Actor。
Isaac Gym 提供了一个简单的过程 API，用于创建环境并在其中填充 actor。这比仅从文件加载静态场景更有用，因为它允许您在将所有 actor 添加到场景中时控制它们的位置和属性。在添加角色之前，您必须创建一个环境：
```python
spacing = 2.0
lower = gymapi.Vec3(-spacing, 0.0, -spacing)
upper = gymapi.Vec3(spacing, spacing, spacing)
# create_env 的最后一个参数表示每行要打包多少个 env。
env = gym.create_env(sim, lower, upper, 8)
```
> PS：Vec3(spacing, spacing, spacing) 表示一个三维向量，其中所有三个坐标（x、y、z）都被赋值为 spacing 变量的值。这通常用来设置一个物体的尺寸或在模拟环境中指定某个区域的边界。
> 
Actor 只是 GymAsset 的一个实例。要将角色添加到环境中，我们必须指定源资产、所需的姿势和一些其他详细信息：
```python
pose = gymapi.Transform()
pose.p = gymapi.Vec3(0.0, 1.0, 0.0)
pose.r = gymapi.Quat(-0.707107, 0.0, 0.0, 0.707107) # quaternion
# pose.r = gymapi.Quat.from_axis_angle(gymapi.Vec3(1, 0, 0), -0.5 * math.pi)
actor_handle = gym.create_actor(env, asset, pose, "MyActor", 0, 1)

```
gym.create_actor 末尾的两个整数是 collision_group 和 collision_filter。这些值在 actor 的物理模拟中起着重要作用。
+ collision_group 是一个整数，用于标识角色的刚体将分配到的碰撞组。
  + **仅当两个物体属于同一个碰撞组时，它们才会相互碰撞**
  + 每个环境通常有一个碰撞组，在这种情况下，组ID对应于环境索引。这可以**防止不同环境中的 actor 彼此进行物理交互**。在某些情况下，您可能希望为每个环境设置多个碰撞组，以便进行更精细的控制。
  + 值 -1 用于与所有其他组碰撞的特殊碰撞组。这可用于创建可以与所有环境中的 Actor 进行物理交互的 “共享” 对象。
+ collision_filter是一个位掩码，可以让你过滤物体之间的碰撞。
  + 如果两个物体的碰撞过滤器有共同的位被设置，则这两个物体将不会发生碰撞。
  + **这个值可以用来过滤多体演员中的自碰撞，或防止场景中某些类型的物体进行物理交互。**

单个循环中初始化所有环境的代码框架：
```python
# set up the env grid
num_envs = 64
envs_per_row = 8
env_spacing = 2.0
env_lower = gymapi.Vec3(-env_spacing, 0.0, -env_spacing)
env_upper = gymapi.Vec3(env_spacing, env_spacing, env_spacing)

# cache some common handles for later use
envs = []
actor_handles = []

# create and populate the environments
for i in range(num_envs):
    env = gym.create_env(sim, env_lower, env_upper, envs_per_row)
    envs.append(env)

    height = random.uniform(1.0, 2.5)

    pose = gymapi.Transform()
    pose.p = gymapi.Vec3(0.0, height, 0.0)

    actor_handle = gym.create_actor(env, asset, pose, "MyActor", i, 1)
    actor_handles.append(actor_handle)

```
在上面的代码段中，所有环境都包含相同类型的 actor，但 actor 在随机高度生成。请注意，分配给每个 actor 的碰撞组对应于 environment index，这意味着来自不同环境的 actor 不会进行物理交互。在构建环境时，我们还会在列表中缓存一些有用的信息，这些信息可以在后续代码中轻松访问。

##### 运行模拟
设置环境网格和其他参数后，您可以开始模拟。这通常在一个循环中完成，其中循环的每次迭代对应于一个时间步：
```
while True:
    # step the physics
    gym.simulate(sim)
    gym.fetch_results(sim, True)
```

### 本文参考链接
[IsaacGym](https://junxnone.github.io/isaacgymdocs/programming/simsetup.html)