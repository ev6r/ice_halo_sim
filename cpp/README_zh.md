# C++ 代码

*提示: 从开发方便的角度, 我会以英文版文档为优先, 中文版内容可能滞后. 个人精力有限, 无法全面照顾, 敬请谅解*

[English version](README.md)

matlab 代码只是一个原型, 并没有充分考虑性能, 因此我专门写了一个 C++ 项目, 以期获得高性能表现.
目前 C++ 版本没有用户界面, 只能从命令行启动.


## 构建, 运行, 测试

本项目使用 [CMake](https://cmake.org/) 构建. 本项目简单的构建过程如下:

1. `cd cpp`
2. `mkdir build && cd build`
3. `cmake .. && make -j4`, 或者设置 `CMAKE_BUILD_TYPE` 为 `release` 以获得高性能可执行程序.

生成的可执行程序位于 `build/bin`. 我在工程中引入了 [GoogleTest](https://github.com/google/googletest)
框架来帮助进行单元测试. 在进行 cmake 配置的过程中, GoogleTest 的代码会自动下载, 通常不用去管他.
测试相关的代码在 `test` 文件夹中, 一般情况下不必改动, 也不需要运行. 如果你对我的单元测试感兴趣, 可以查看对应代码.

### 仿真

在命令行输入 
`./bin/IceHaloSim <path-to-your-config-file>` 即可运行.
这里有一个 [`config-example.json`](./config-example.json) 文件作为输入配置的样本, 可供参考.
在运行程序之后, 你将得到一些 `.bin` 文件, 这些文件包含了所有光线追踪的结果. 
这些数据文件位于配置文件中指定的数据路径中.

### 可视化

运行仿真程序后你将得到一些 `.bin` 文件, 以及一些晶体的形状信息. 项目中我准备了几个小工具来做可视化相关的工作.

* 仿真结果图  
`matlab/src/read_binary_result.m` 用于读取 `.bin` 文件并输出光晕效果图.  
此外也有一个 C++ 小工具来做效果渲染, 可以直接运行 `./bin/IceHaloRender <config-file>`.
使用模拟时相同的配置文件即可, 
在配置文件样本中可以找到渲染相关的配置选项. 我个人更推荐使用 C++ 版本的可视化工具, 速度更快.

* 冰晶形状  
`matlab/src/plot_crystal_main.m` 用于绘制冰晶的形状. 你可以把程序运行后在屏幕输出的结果作为数据, 其中,
`V:` 开头的那些代表顶点 (verte) 数据, `F:` 开头的那些代表面 (face) 数据.

## 配置文件

配置文件中包含了用于模拟的所有参数, 配置文件使用 JSON 格式, 
本项目中我选择了 [Rapidjson](http://rapidjson.org/index.html)
对 JSON 文件进行解析.

### 模拟的基本设置

* `sun`:
有两个属性, 
  * `altitude`, 用于定义太阳的地平高度.  
  * `diameter`, 定义太阳的直径, 单位是度. 对于真实太阳, 这里设置为 0.5 即可

* `ray`:
定义了用于模拟的光线相关的属性, 有两个, 
  * `number`, 用于模拟的光线总数量. 请注意, 这里光线数量并不是最终输出的光线数量, 而是输入光线数量. 
    由于光线在晶体内部进行折射和反射, 在模拟中对所有的折射和反射光线都进行记录, 因此最终的输出光线数量将大于这里定义的值.
  * `wavelength`, 用于模拟的光线波长. 用一个数组来表示, 单位为 nm. 冰的折射率数值来源于
    [Refractive Index of Crystals](https://refractiveindex.info/?shelf=3d&book=crystals&page=ice).

* `max_recursion`:
定义了在模拟中光线与晶体表面相交的最多次数. 如果模拟中光线与晶体表面相交次数超过这个值, 而仍然没有离开晶体,
那么对这条光线的模拟将终止, 这条光线的结果将被舍弃.

* `multi_scatter`:
定义了有关多晶折射相关的属性, 有两个,
  * `repeat`, 定义多晶折射的次数, 对于普通日晕模拟, 设置为 1 即可; 大多数多晶情况只需要设置为 2 即可模拟出效果.
  * `probability`, 在每次穿过晶体之后有多少比例继续进入下一个晶体进行折射.

### 渲染设置

* `camera`:
定义了与相机相关的设置,
  * `azimuth`, `elevation`, `rotation`: 这几个定义了相机的指向. 单位是度.
  * `fov`: 相机的(半)视场角, 从视场中心到边缘的角度. 单位是度.
  * `width`, `height`: 输出图像的尺寸. 单位是像素.
  * `lens`: 镜头类型, 可以是 `fisheye` (鱼眼镜头) 或者 `linear` (普通广角镜头) 其中之一.
  
* `render`:
定义了与最后输出效果相关的设置:
  * `visible_semi_sphere`, 定义了全空间中哪一部分光线最终被渲染. 默认设置为 `upper` (上半球), 也即是普通的场景,
    只能看见地平线上方的晕. 如果设置为 `lower`, 那么只有下半球 (地平线下方) 的光线会被渲染,
    可以看到一些特殊的晕, 比如说, [下映日](https://www.atoptics.co.uk/halo/subpars.htm).
    这个参数可以取 `upper`, `lower`, `camera`, `full` 等几个值.
  * `intensity_factor`, 控制输出图像的亮度.
  * `offset`, 输出图像本身的偏移量.

### 晶体设置

* `axis` and `roll`:
这两个参数定义了晶体的朝向, 其中 `axis` 定义了 c-轴 的指向,`roll` 定义了晶体自身绕 c-轴 的旋转.

  这两个参数都有 3 个属性, `mean`, `std`, `type`. 其中 `type` 定义了随机分布的类型. 要么是 `Gauss`, 
代表高斯分布, 要么是 `Uniform`, 代表平均分布. `mean` 定义了随机分布的均值. 对 `axis` 来说, 
这个值代表天顶角, 从天顶开始度量.
`std` 定义了随机分布的范围. 对高斯分布来说, 这个值就是高斯分布的标准差,
对平均分布来说, 这个值定义了取值范围的大小.

  所有角度单位都是度.

* `population`:
这个参数定义了用于模拟的晶体数量. 请注意, 这里并不是实际数量, 而是一个比例. 比如两种晶体一种是 2.0 一种是 3.0,
那么这等价于一种是 20 另一种是 30.

* `type` 和 `parameter`:
用于设置晶体的类型和形状参数. 目前我支持 5 种晶体,
`HexCylinder`, `HexPyramid`, `HexPyramidStackHalf`, `TriPyramid`, `CubicPyramid`.
每种晶体都有自己的形状参数.

  * `HexCylinder`: 六棱柱形冰晶.
  只有 1 个参数, `h`, 定义为 `h / a`, 其中 `h` 是柱体的高, `a` 是底面直径.  
  <img src="figs/hex_cylinder_01.png" width="400">.

  * `HexPyramid`: 六棱锥形冰晶.
  可能有 3, 5, 或者 7 个参数.  
    * 3 个参数的情况, 分别表示 `h1 / H1`, `h2 / a`, `h3 / H3`, 其中 `H1` 代表第一段锥体最大可能高度, `H3` 类似.  
      <img src="figs/hex_pyramid_01.png" width="400">.  
    * 5 个参数的情况. 最后 3 个参数含义同上, 开头 2 个参数用于定义晶体表面的方向, 这 2 个参数必须是整数.
      这里使用 [Miller index](https://en.wikipedia.org/wiki/Miller_index) 来表示晶体表面方向. 
      举个例子, 2 个参数为 `a`, `b`, 那么表示 Miller index (`a`, 0, `-a`, `b`). 对于一个正常的冰晶,
      比如编号 13 的那个表面, 对应的 Miller index 是 (1, 0, -1, 1), 那么这里参数就写成 1, 1.  
    * 7 个参数的情况. 最后 3 个参数含义同上, 开头 4 个参数用于定义晶体表面方向, 前 2 个定义上面一段锥体的表面,
      后 2 个定义下面一段锥体表面, 定义方式同样是基于 Miller index, 与之前的相同.
      请注意, 对不同 Miller index 的表面, 其最大可能高度 `H` 是不一样的.

  * `HexPyramidStackHalf`:
  有 7 个参数. 与前面六棱锥形冰晶参数类似, 开头 4 个参数用于定义锥面的角度, 后面 3 个参数用于定义 3 段的长度,
  对锥体段表示 `h / H`, 对柱体段表示 `h / a`.  
  <img src="figs/hex_pyramid_stack_half_01.png" width="400">. 

  * `TriPyramid`:
  有 5 个参数. 与前面六棱锥形冰晶参数类似, 开头 2 个参数用于定义锥面的角度. 这里上下两个锥体段角度是一样的.
  (其他情况的我稍后有空再加进程序里).  
  <img src="figs/tri_pyramid_01.png" width="400">. 

  * `CubicPyramid`:
  有 2 个参数. 与上面的情形类似, 2 个参数定义了上下两段锥体的长度. 请注意, 这里是立方晶系.  
  <img src="figs/cubic_pyramid_01.png" width="400">
  
  * `Custom`:
  有 1 个参数, 代表自定义模型的文件名.  
  自定义模型支持 [obj 文件格式](https://www.wikiwand.com/en/Wavefront_.obj_file). 
  这种 3D 文件格式是基于 ASCII 编码的, 因此使用任意一款文本编辑器即可直接阅读和修改, 十分方便.
  当然, 最好还是在 3D 建模软件里进行编辑和修改, 比如 Maya, 3DMax, Blender, 等等.  
  *注意:* obj 文件格式本身允许存在多边形, 也即某一个表面拥有多余 3 个顶点, 但我的程序里无法识别这种情况,
  只能处理三角形. 并且我的程序无法处理纹理和法线信息. 在使用建模软件创建模型的时候请不要包含这些.  
  所以的自定义模型必须放在 `models` 的文件夹内, 这个文件夹必须放在配置文件相同的目录下,
  否则程序无法找到模型文件.


## 未来的工作

* 使用 OpenCL / OpenGL / CUDA 等对程序进行加速. 不过, 目前使用一个我自己实现的线程池做并行化计算,
  结果已经很好, 进一步加速的必要性并不大.
* 增加更多方便的内置模型, 比如异形六边形的柱晶和锥晶.
* 写一个用户界面, 可能是基于网页的.
