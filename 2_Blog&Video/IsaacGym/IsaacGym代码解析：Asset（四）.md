> 想了下，Asset还是不翻译比较好，更加保持原意。

### 加载Assets
这个部分在系列第二篇文章中说过，这里不进行赘述。

### 在Assets中加载网格
Gym 使用 Assimp 加载资产中指定的网格。某些网格可以直接在网格文件中指定材质或纹理。如果资源文件还为该网格指定了材质，则资源中的材质优先。要改用网格材质，请使用：
`asset_options.use_mesh_materials = True.`
Gym 尝试直接从网格加载法线。如果网格具有不完整的法线，则 Gym 将生成平滑的顶点法线。要强制Gym始终生成平滑的顶点法线/面法线，请使用：
```python
asset_options.mesh_normal_mode = gymapi.COMPUTE_PER_VERTEX
or
asset_options.mesh_normal_mode = gymapi.COMPUTE_PER_FACE
```
PS:“法线”（normal）指的是与三维物体表面相垂直的向量。法线在计算光照、碰撞检测和物体表面属性时非常重要，因为它们帮助确定光线如何与表面交互。

### 覆盖惯性属性
刚体的惯性属性对于仿真精度和稳定性非常重要。每个刚体都有一个质心和一个惯性张量。URDF 和 MJCF 允许指定这些值，但有时这些值不正确。例如，有许多 URDF 资产具有看似任意的惯性张量，这可能会导致不需要的模拟伪影。为了克服这个问题，Isaac Gym 允许使用从碰撞形状的几何计算的值显式覆盖质心和惯性张量：
```python
# 默认情况下，这些选项均为 False，这意味着导入器将使用原始资产中给定的值。
asset_options.override_com = True
asset_options.override_inertia = True
```

### 凸面分解 Convex Decomposition
> “凸面分解”是计算几何和计算机图形学中的一个概念，指的是将一个复杂的多面体（通常是非凸的）分解成多个简单的凸多面体的过程。

Isaac Gym 支持用于碰撞形状的三角形网格的自动凸分解。只有在使用 PhysX 时才需要这样做，因为 PhysX 需要凸面网格进行碰撞（Flex 能够直接使用三角形网格）。如果不进行凸面分解，则每个三角形网格形状都使用单个凸包进行近似处理。这很有效，但单个凸包可能无法准确表示原始三角形网格的形状。**凸面分解可创建由多个凸面形状组成的更精确的形状近似值**。这对于抓取等应用程序特别有用，因为在这些应用程序中，物理对象的精确形状很重要。

默认情况下，凸面分解处于禁用状态。在资源导入期间，您可以使用 AssetOptions 中的标志启用它：`asset_options.vhacd_enabled = True`

Gym 使用第三方库 （V-HACD） 来执行凸分解。有许多参数会影响分解速度和结果的质量。它们在 isaacgym.gymapi.VhacdParams 类中向 Python 公开，该类包含在 AssetOptions 中：
```python
asset_options.vhacd_params.resolution = 300000
asset_options.vhacd_params.max_convex_hulls = 10
asset_options.vhacd_params.max_num_vertices_per_ch = 64
```
凸面分解可能需要很长时间，具体取决于参数和网格的数量。为了避免每次都重新计算凸分解，Gym 使用了简单的缓存策略。首次分解网格时，分解结果将存储在缓存目录中，以便在后续运行时快速加载。默认情况下，缓存目录为`${HOME}/.isaacgym/vhacd`。缓存目录及其包含的任何文件都可以随时安全删除。
我们可以通过单击查看器 GUI 中的 Viewer 选项卡并启用“Render Collision Meshes”复选框来查看凸面分解的结果。

### 程序化资产 Procedural Assets
可以按程序创建简单的几何资源，如盒子、胶囊体和球体：
```pyhton
asset_options = gym.AssetOptions()
asset_options.density = 10.0

box_asset = gym.create_box(sim, width, height, depth, asset_options)
sphere_asset = gym.create_sphere(sim, radius, asset_options)
capsule_asset = gym.create_capsule(sim, radius, length, asset_options)
```

### 资产自省 Asset Introspection
我们可以检查每个资产中的组件集合，包括刚体、关节和 DOF。有关示例用法，请参阅 examples/asset_info.py。

### 本文参考链接
[IsaacGym](https://junxnone.github.io/isaacgymdocs/programming/simsetup.html)