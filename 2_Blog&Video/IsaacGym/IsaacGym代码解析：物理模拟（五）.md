### 创建actor
actor是 GymAsset 的一个实例。该函数create_actor将 actor 添加到环境中，并返回一个 actor 句柄，该句柄可用于稍后与该 actor 交互。出于性能原因，最好在角色创建期间保存handles，而不是每次在模拟运行时都查找它们。许多随附的示例都执行如下操作：
```python
# cache useful handles
envs = []
actor_handles = []

print("Creating %d environments" % num_envs)
for i in range(num_envs):
    # create env
    env = gym.create_env(sim, env_lower, env_upper, num_per_row)
    envs.append(env)

    # add actor
    actor_handle = gym.create_actor(env, asset, pose, "actor", i, 1)
    actor_handles.append(actor_handle)
```
角色手柄特定于在其中创建角色的环境。对 actor 进行操作的 API 函数需要环境引用和 actor 句柄，因此它们通常一起缓存。

有相当多的函数可以对 actor 进行操作。它们被命名为 get_actor_xx、set_actor_xx 或 apply_actor_xx。使用 API reference 获取完整列表。

#### 聚合 Aggregates
注意：聚合仅适用于 PhysX 后端。创建聚合对 Flex 没有影响。

### Actor组件
每个actor都有一组刚体、关节和 DOF。我们可以像这样获取计数：
```
num_bodies = gym.get_actor_rigid_body_count(env, actor_handle)
num_joints = gym.get_actor_joint_count(env, actor_handle)
num_dofs = gym.get_actor_dof_count(env, actor_handle)
```
此时，在创建 Actor 后，无法添加或删除 Actor 组件。

#### 刚体
每个刚体由一个或多个刚体形状组成。我们可以自定义每个角色的刚体和形状属性，如 body_physics_props.py 示例中所示。

#### 关节
Gym 导入器保留了 URDF 和 MJCF 模型中定义的关节。固定、旋转和棱柱关节经过充分测试并得到全面支撑。使用这些关节类型，可以创建各种各样的模型。计划对球形和平面关节提供额外的支持。

#### 自由度 DOF
每个自由度都可以独立驱动。您可以将控件应用于单个 DOF，或使用数组同时处理所有角色 DOF，如下一节所述。

### 控制actors
控制 actor 是使用自由度完成的。对于每个自由度，我们可以设置驱动模式、限制、刚度、阻尼和目标。您可以为每个角色设置这些值，并覆盖从资源加载的默认设置。

### 缩放 Actor
Actor 可以在运行时缩放其大小。缩放角色将更改其碰撞几何体、质量属性、关节位置和棱柱关节限制。有关示例用法，请参阅 examples/actor_scaling.py。

缩放 Actor 后，可能必须手动调整其某些属性才能获得所需的结果。例如，其 DOF 属性（刚度、阻尼、最大力、电枢等）、驱动力、肌腱属性等可能必须进行调整。我们可以访问`get_asset_dof_properties`和`get_actor_dof_properties/set_actor_dof_properties`的DOF属性数组，最终会返回numpy数组。

DOF_MODE_EFFORT允许使用 apply_actor_dof_efforts 将 efforts 应用于 DOF。如果 DOF 是线性的，则 efforts 是以牛顿为单位的力。如果 DOF 是有角度的，则 efforts 是以 Nm 为单位的扭矩。DOF 属性只需设置一次，但必须每帧都应用 efforts 。在每一帧中多次应用努力是累积的，但在下一帧开始时，它们将重置为零：
```python
# configure the joints for effort control mode (once)
props = gym.get_actor_dof_properties(env, actor_handle)
props["driveMode"].fill(gymapi.DOF_MODE_EFFORT)
props["stiffness"].fill(0.0)
props["damping"].fill(0.0)
gym.set_actor_dof_properties(env, actor_handle, props)

# apply efforts (every frame)
efforts = np.full(num_dofs, 100.0).astype(np.float32)
gym.apply_actor_dof_efforts(env, actor_handle, efforts)
```
DOF_MODE_POS 和 DOF_MODE_VEL 与 PD 控制器接合，该控制器可以使用刚度和阻尼参数进行调整。控制器将施加与 `posError * stiffness + velError * damping` 成比例的 DOF 力。

另外我们用以下方法设置目标位置：
```
targets = np.zeros(num_dofs).astype('f')
gym.set_actor_dof_position_targets(env, actor_handle, targets)
```
+ DOF_MODE_POS 用于位置控制，通常刚度和阻尼都设置为非零值
+ DOF_MODE_VEL 用于速度控制。刚度参数应设置为零。
我们可以使用 set_actor_dof_velocity_targets 设置速度目标。如果 DOF 是线性的，则目标以米/秒为单位。如果 DOF 是角度的，则目标以弧度/秒为单位：
```
vel_targets = np.random.uniform(-math.pi, math.pi, num_dofs).astype('f')
gym.set_actor_dof_velocity_targets(env, actor_handle, vel_targets)

```
**与 efforts 不同，位置和速度目标不需要每帧都设置，只需在更改目标时设置即可。**

除了使用执行组件 DOF 数组外，还可以独立控制每个 DOF，如 dof_controls.py 示例所示。使用单个 DOF 更灵活，但**由于来自 Python 的多个 API 调用会产生开销，因此效率可能会降低。**

### 物理状态
Gym 提供了一个 API，用于获取和设置结构化 Numpy 数组的物理状态。

#### 刚体状态
刚体状态包括位置 （Vec3）、方向 （Quat）、线速度 （Vec3） 和角速度 （Vec3）。这允许我们在最大坐标中处理仿真状态。我们可以获取 actor、env 或整个模拟的刚体状态数组：
```python
body_states = gym.get_actor_rigid_body_states(env, actor_handle, gymapi.STATE_ALL)
body_states = gym.get_env_rigid_body_states(env, gymapi.STATE_ALL)
body_states = gym.get_sim_rigid_body_states(sim, gymapi.STATE_ALL)
# 这些方法返回结构化的 numpy 数组。
# 最后一个参数是一个位字段，用于指定应返回的状态类型。
# STATE_POS 表示应计算位置，STATE_VEL 表示应计算速度，STATE_ALL 表示两者都应计算。

# 切片访问数组
body_states["pose"]             # all poses (position and orientation)
body_states["pose"]["p"])           # all positions (Vec3: x, y, z)
body_states["pose"]["r"])           # all orientations (Quat: x, y, z, w)
body_states["vel"]              # all velocities (linear and angular)
body_states["vel"]["linear"]    # all linear velocities (Vec3: x, y, z)
body_states["vel"]["angular"]   # all angular velocities (Vec3: x, y, z)

# 编辑和设置状态
gym.set_actor_rigid_body_states(env, actor_handle, body_states, gymapi.STATE_ALL)
gym.set_env_rigid_body_states(env, body_states, gymapi.STATE_ALL)
gym.set_sim_rigid_body_states(sim, body_states, gymapi.STATE_ALL)

# 确定状态数组中特定刚体的偏移量
i1 = gym.find_actor_rigid_body_index(env, actor_handle, "body_name", gymapi.DOMAIN_ACTOR)
i2 = gym.find_actor_rigid_body_index(env, actor_handle, "body_name", gymapi.DOMAIN_ENV)
i3 = gym.find_actor_rigid_body_index(env, actor_handle, "body_name", gymapi.DOMAIN_SIM)
```

#### DOF状态
我们也可以使用缩减的坐标来处理actor：
```python
dof_states = gym.get_actor_dof_states(env, actor_handle, gymapi.STATE_ALL)
gym.set_actor_dof_states(env, actor_handle, dof_states, gymapi.STATE_ALL)

# 访问属性
dof_states["pos"]   # all positions
dof_states["vel"]   # all velocities
```
DOF 状态数组包括 position 和 velocity 作为单个浮点数。对于线性自由度，位置以米为单位，速度以米/秒为单位。对于角度自由度，位置以弧度为单位，速度以弧度/秒为单位。

