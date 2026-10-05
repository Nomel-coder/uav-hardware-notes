# 四旋翼无人机硬件组成与信号链

这份报告从一架搭载机载计算机的四旋翼出发，梳理各个硬件模块如何配合。先明确模块职责，再沿传感、计算、控制和执行的方向查看连接关系，最后检查供电与调试条件。

## 硬件组成

| 模块 | 主要作用 | 与其他模块的关系 |
| --- | --- | --- |
| 机架与螺旋桨 | 支撑设备，将电机转动转化为推力 | 机架固定四组动力部件，桨叶需要匹配电机旋向 |
| 无刷电机 | 驱动螺旋桨旋转 | 由电调的三相输出供电 |
| 电调 | 根据控制指令驱动电机 | 接收飞控信号，从动力电源取电 |
| 飞行控制器 | 估计飞行状态并执行姿态等控制 | 接收传感器、遥控器或机载计算机的数据，向电调分配控制量 |
| 机载计算机 | 运行视觉、建图、路径规划等任务 | 处理相机与雷达数据，与飞控交换状态和目标指令 |
| 雷达、相机与光流传感器 | 感知周围环境和相对运动 | 根据接口和系统设计接入机载计算机或飞控 |
| 遥控器与接收机 | 提供人工操纵入口 | 接收机将通道数据送给飞控 |
| 电池与分电模块 | 为各模块提供电能 | 动力、电源转换和信号连接应分别核对 |

## 一次控制过程

飞手操纵摇杆后，接收机把通道数据送给飞控。飞控结合状态估计结果计算各电机需要的控制量，再交给电调驱动电机。四组螺旋桨推力的大小和分布改变，机体便产生相应的平动与转动。

自主任务中，机载计算机处理相机或雷达的数据，生成位置、速度等目标。飞控仍负责底层飞行控制，因此“机载计算机发出任务目标”和“飞控驱动电机”是两个不同环节。

## 接口与协议

以图示传感配置为例，可以按下表区分接口、消息协议和数据方向。网线、USB、UART 与 MAVLink 不在同一层次；连接图中的双箭头也不能替代对实际遥测支持的核对。

| 连接 | 接口或协议 | 主要信息／能量流 |
| --- | --- | --- |
| MID-360 ↔ Orin NX | 以太网；Livox 协议封装于 UDP | 雷达→主机：点云、IMU；主机↔雷达：配置、状态 |
| 两台 RGB 相机 → Orin NX | USB；具体视频协议待型号确认 | 相机→主机：图像；控制传输依驱动而定 |
| Orin NX ↔ PX4 飞控 | UART 承载 MAVLink | Orin→飞控：控制目标、命令；飞控→Orin：状态、反馈 |
| 优象光流 → PX4 飞控 | 图示 UART／MAVLink | 主要上传光流测量；测距与具体消息待型号确认 |
| AT9S Pro → R12DSM → 飞控 | RadioLink 无线链路；SBUS 输出 | 操作者输入→接收机→飞控 |
| 飞控 → 蓝鸟 50A 电调 | DShot600 | 数字油门与命令；遥测能力须单独确认 |
| 电调 → F90 电机 → 桨叶 | 三相驱动；机械连接 | 电能→电磁转矩→桨叶旋转 |
| 6S 电池 → 分电板 → 用电设备 | 直流供电 | 提供能量；12 V／24 V 支路需核对实际稳压与额定范围 |

其中，DShot600 的 600 表示 600 kbit/s，不是电机转速。普通 DShot 指令与双向 DShot 遥测需要区分；是否支持反馈，还取决于飞控、电调及固件配置。

## 版本记录的阅读方式

报告使用 Markdown 编写，正文就在本文件中。Git 的每次提交记录一次内容快照；后续版本会补充接口、通信方向和调试约束。可以在 GitHub 的提交记录中查看对应差异。

## 参考资料

- PX4 开发文档：[Companion Computers](https://docs.px4.io/main/en/companion_computer/)
- GitHub 文档：[About Git](https://docs.github.com/en/get-started/using-git/about-git)


- [Livox MID-360 技术参数](https://www.livoxtech.com/cn/mid-360/specs)
- [Livox MID-360 通信协议](https://livox-wiki-en.readthedocs.io/en/latest/tutorials/new_product/mid360/livox_eth_protocol_mid360.html)
- [MAVLink Developer Guide](https://mavlink.io/en/)
- [PX4 Using a Companion Computer with Pixhawk Controllers](https://docs.px4.io/main/en/companion_computer/pixhawk_companion)
- [PX4 Optical Flow](https://docs.px4.io/main/en/sensor/optical_flow)
- [RadioLink R12DSM User Manual](https://www.radiolink.com/r12dsm_manual)
- [PX4 DShot ESCs](https://docs.px4.io/main/en/peripherals/dshot)
