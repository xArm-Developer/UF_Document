# 机械臂碰撞检测原理及误触发的处理

UFACTORY 机械臂的碰撞检测功能依赖于电流和动力学模型的结合，通过对比各个关节的理论电流和实际电流差异，来判断是否发生了碰撞。以下是对其工作原理及相关设置的详细说明。

#### 1. **碰撞检测原理简述**

机械臂的控制系统会依据动力学模型对每个关节的理论电流进行计算。动力学模型会综合考虑机械臂的当前状态，包括：

* **关节位置**：机械臂的当前位姿。
* **速度和加速度**：每个关节的运动速度与加速度。
* **负载重量与质心**：末端负载的重量和质心位置。
* **安装方向** : 安装机械臂的方式，正装、侧装、倒装或者其他。
* **关节摩擦力**: 每个关节的静摩擦和动摩擦参数。

控制器通过这些参数结合动力学方程，实时推算出在理想情况下各个关节应该消耗的电流。同时，系统会监测每个关节的实际电流，并与理论值进行对比。如果两者之间的差值超过预设的阈值，意味着关节可能遇到外部阻力或碰撞，系统会触发碰撞检测。

#### 2. **误触发碰撞检测的原因**

在实际应用中，机械臂的碰撞检测功能有时可能会被误触发。这种现象通常与机械臂的末端负载与质心、安装方向以及摩擦力参数有关。以下几个设置是导致误触发的常见原因：

**2.1 末端负载设置**

**负载的重量与质心设置**

末端负载的重量和质心位置会直接影响关节的受力和电流。如果负载的重量或质心位置的参数设置错误，动力学模型计算出的理论电流可能会出现偏差。例如，某些工具的质心离机械臂末端较远，会导致末端关节需要消耗更多的电流来平衡重力。如果质心参数设置的是\[0,0,0]，控制器可能会误报电流异常。

如何获取准确的负载重量和质心？一般有2种方法。

* 在UFACTORY Studio-设置-运动-TCP设置界面进行TCP负载参数的自动辨识。
* 通过称量、仿真计算等得出负载的重量和质心。一般在TCP负载辨识不可用的情况，例如机械周围空间狭小，做TCP负载辨识可能会与周围环境发生干涉。

**动态的负载变化** 在抓取-放置应用中，机械臂的末端负载会随着程序执行而动态变化，抓取工件后实际负载变大，放置工件后实际负载变小。所以编写程序时，需要告知控制器末端负载的变化情况。通常在抓取指令之前，设置一次负载；在放置工件指令之后，再设置一次负载。代码逻辑如下

```
设置负载(不包含工件的末端负载的重量和质心)
机械臂运动
抓取工件
设置负载(包含工件的末端负载的重量和质心)
机械臂运动
放置工件
设置负载(不包含工件的末端负载的重量和质心)
```

**2.2 机械臂的安装方向**

UFACTORY 机械臂支持不同的安装方式，正装、侧装、吊装或者自定义安装方式，每种安装方式下机械臂都需要对重力方向进行不同的补偿计来修正关节的理论电流值。如果机械臂实际安装方式是吊装，但是软件设置的是正装，关节电流的理论值就可能出现偏差，系统会认为实际电流与理论电流差距过大，从而报出电流异常。 确保软件设置的机械臂安装方向与实际匹配，特别是在自定义安装方向时，以避免不必要的误差。对于xArm XX1300及以上版本的机械臂，以及所有的Lite6、850 机械臂，在UFACTORY Studio软件中设置安装方向时，系统会根据机械臂自带的传感器测量机械臂实际的安装方向，并与用户设置的安装方向做比对，如果两者不匹配，则会进行弹窗提示。

**2.3 摩擦力参数**

在 UFACTORY 机械臂的控制系统中，摩擦力参数也是碰撞检测的重要影响因素之一。UFACTORY 机械臂在出厂前会进行摩擦力参数辨识，包括每个关节的静摩擦和动摩擦参数。这些参数被存储在控制器中和驱动器，用于计算各个关节在不同负载和运动状态下的理论电流。

当用户更换了控制器后，控制器内的摩擦力参数不再适用，导致系统在计算关节电流时出现误差。由于摩擦力直接影响关节的运动阻力和电流，参数不匹配时，控制器可能会错误地认为关节遇到了外界阻力，进而误触发碰撞检测。

下表是UFACTORY 各个型号机械臂的摩擦参数的存储位置，以及用户自行更换控制器后重新载入摩擦力的的方法。

| 机械臂型号                   | 摩擦力存储位置 | 摩擦力重载方式  |
| ----------------------- | ------- | -------- |
| 850                     | 机械臂     | 拍一次急停，松开 |
| Lite 6                  | 机械臂     | 拍一次急停，松开 |
| xArm 5/6/7              | 机械臂     | 拍一次急停，松开 |
| xArm 5/6/7（XX1300 之前版本） | 控制器     | 联系技术支持   |

如何确保控制器使用了正确的摩擦力参数？ UFACTORY 机械臂的摩擦力参数与机械臂序列号（SN）绑定，可以对比机械臂实际SN与控制器加载的机械臂SN，如二者匹配，则表示控制器使用了正确的摩擦力参数。如不匹配，则需要重新加载摩擦力参数。

控制器加载的摩擦力参数对应的SN在UFACTORY Studio-设置-我的设备-设备信息的“机械臂SN”一栏可以看到。机械臂的实际SN 在机械臂底部的贴纸上。

#### 3. **优化碰撞检测功能的建议**

为了提高 UFACTORY 机械臂碰撞检测的准确性，避免误触发，以下几点建议可以帮助优化使用效果。

* **及时更新负载和质心信息**：
  * 每次更换工具或调整负载时，需要在设置界面更新负载重量和质心位置。
  * 抓取-放置类的程序，需要在程序中的抓取、释放后设置负载。
* **正确设置机械臂安装方向**：确保在UFACTORY Studio或者通过SDK 设置正确的机械臂的安装方向。
* **确保摩擦力参数匹配**：更换控制器后，确保重新加载了机械臂的关节摩擦力参数。

#### 4. **总结**

UFACTORY 机械臂的碰撞检测功能通过对比理论电流与实际电流差异，能在机械臂发生碰撞后立即停止机械臂。然而，实际使用中的一些误触发问题与末端负载和质心以及安装方式设置密切相关。通过准确设置这些参数，可以在保证机械臂安全的前提下有效避免误报碰撞检测。