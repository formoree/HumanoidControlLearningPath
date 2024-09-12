### Tensor API
没事多看看[Tensor API](https://junxnone.github.io/isaacgymdocs/programming/tensors.html)、[Graphics and Camera Sensors](https://junxnone.github.io/isaacgymdocs/programming/graphics.html)、[Terrains](https://junxnone.github.io/isaacgymdocs/programming/terrain.html)。这些在用到的时候看看属性即可。

### 力传感器
#### 刚体力传感器
力传感器可以连接到刚体上，以测量在用户指定的参考系下受到的力和扭矩。读数表示父体所承受的净力和扭矩，包括求解器施加的外力、接触力和内力（例如，由于关节驱动）。位于接地平面上的物体的净力为零。

力传感器是在 asset 上创建的。这样，我们只需定义一次力传感器，它们就会在从 asset 实例化的每个 actor 上创建。要创建力传感器，要指定传感器将附加到的刚体索引以及传感器相对于实体原点的相对姿势。多个传感器可以连接到同一个身体上：
```python
body_idx = gym.find_asset_rigid_body_index(asset, "bodyName")

sensor_pose1 = gymapi.Transform(gymapi.Vec3(0.2, 0.0, 0.0))
sensor_pose2 = gymapi.Transform(gymapi.Vec3(-0.2, 0.0, 0.0))

sensor_idx1 = gym.create_asset_force_sensor(asset, body_idx, sensor_pose1)
sensor_idx2 = gym.create_asset_force_sensor(asset, body_idx, sensor_pose2)

# 为每个传感器传递其他属性，这些属性将确定力的计算方式
sensor_props = gymapi.ForceSensorProperties()
sensor_props.enable_forward_dynamics_forces = True
sensor_props.enable_constraint_solver_forces = True
sensor_props.use_world_frame = False

sensor_idx = gym.create_asset_force_sensor(asset, body_idx, sensor_pose, sensor_props)

```
创建 actor 后，我们可以获取附加的力传感器的数量，并使用其索引访问各个力传感器：
```
actor_handle = gym.create_actor(env, asset, ...)

num_sensors = gym.get_actor_force_sensor_count(env, actor_handle)
for i in range(num_sensors):
    sensor = gym.get_actor_force_sensor(env, actor_handle, i)

# 查询最新的传感器读数
sensor_data = sensor.get_forces()
print(sensor_data.force)   # force as Vec3
print(sensor_data.torque)  # torque as Vec3

```
连接到同一物体的两个传感器将报告相同的力，但如果它们的相对姿势不同，则扭矩通常会有所不同。力是在人体质心处测量的，但扭矩是在传感器的局部参考系处测量的。我们可以通过调用gym.get_sim_force_sensor_count(sim) 来获得模拟中力传感器的总数。

我们通过函数 acquire_force_sensor_tensor 获得Gym的Tensor描述符，并包装为pytorch tensor：
```
_fsdata = gym.acquire_force_sensor_tensor(sim)
fsdata = gymtorch.wrap_tensor(_fsdata)
# 该张量的形状为 （num_force_sensors， 6），数据类型为 float32。对于每个传感器，前三个浮点数是力，后三个浮点数是扭矩。

# 获取最新的传感器读数
gym.refresh_force_sensor_tensor(sim)
```
#### 关节力传感器
如果我们要能读取actor上每个自由度的力，要设置：
```
actor = gym.create_actor(env, ...)
gym.enable_actor_dof_force_sensors(env, actor)

# 函数将返回作用在此 actor 的 DOF 上的总力的 numpy 数组。
# 如果 enable_actor_dof_force_sensors 之前未在 actor 上调用，则返回的力将始终为零。
forces = gym.get_actor_dof_forces(env, actor_handle)

# 获取环境中所有DOF对应的力
_forces = gym.acquire_dof_force_tensor(sim)
forces = gymtorch.wrap_tensor(_forces)

# 获取最新传感器读数
gym.refresh_dof_force_tensor(sim)
```
出于性能原因，默认情况下不启用这些传感器。尽管性能影响通常很小，但最好仅在需要时启用传感器。