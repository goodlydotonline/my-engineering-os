---
Title: "辨析：安卓手机 AR 硬件支持、ARCore、ARCore Depth API"
Date: 2026-06-24
Tags: ["#vision", "#ar", "#android", "#arcore", "#depth-api", "#verified"]
Status: Verified
Source: "engineering-analysis"
Related: ["inbox/2026/raw-notes/2026-06-23-android-arcore-distance-measurement-accuracy.md"]
---

# 辨析：安卓手机 AR 硬件支持、ARCore、ARCore Depth API

## 三个概念

### 1. 安卓手机 AR 硬件级别支持

指手机本身具备的、可用于 AR 的传感器和计算能力，主要包括：

- **RGB 摄像头**：用于视觉 SLAM、运动追踪、平面检测。
- **IMU（加速度计 + 陀螺仪）**：辅助位姿估计和运动补偿。
- **ToF / iToF 深度传感器**：直接测量场景深度，精度较高。
- **ISP / NPU / GPU**：用于实时图像处理和深度估计加速。
- **系统厂商认证**：Google 官方 ARCore 支持设备列表是硬性门槛。

关键点：硬件是基础，但硬件支持 ≠ 软件能力自动可用。

### 2. ARCore

Google 提供的 Android AR SDK，是软件层，负责把硬件原始数据转换成可用的 AR 能力：

- **运动追踪（Motion Tracking）**：通过视觉 + IMU 估计相机位姿。
- **环境理解（Environmental Understanding）**：检测水平/垂直平面。
- **光照估计（Light Estimation）**：估计环境光照，用于虚拟物体渲染。
- **Cloud Anchors / Geospatial API**（可选）：多人协作或地球尺度定位。

关键点：ARCore 是软件框架。只要设备在官方支持列表里，基础 ARCore 功能（平面检测、hitTest）就能工作。

### 3. ARCore Depth API

ARCore 内部的一个可选子能力，用来获取深度图。

- **作用**：估算画面中每个像素到相机的距离。
- **实现方式**：
  - **硬件版**：调用 ToF / iToF 传感器，精度较高。
  - **软件版**：没有 ToF 时，ARCore 用运动视差算法估算深度，精度一般，近距离误差明显。
- **特点**：不依赖平面，理论上对小物体、非平面、垂直面更友好。
- **限制**：需要设备支持、需要 App 开启 depth mode、API 调用更复杂、精度不如激光测距仪。

关键点：Depth API 是 ARCore 的“可选扩展”，不是默认能力。

---

## 三者关系

```text
安卓手机硬件（摄像头 / ToF / 惯导）
        ↓
   提供原始数据
        ↓
    ARCore SDK（软件层）
        ├─ 基础 Plane Detection / hitTest     ← 当前项目正在用
        └─ Depth API（可选子模块）            ← 当前项目只有接口占位，未启用
```

| 层级 | 代表 | 作用 |
|------|------|------|
| 硬件 | 摄像头、ToF、IMU | 采集原始数据 |
| 软件框架 | ARCore | 运动追踪、平面检测、环境理解 |
| 软件扩展 | Depth API | 获取逐像素深度，增强测量能力 |

---

## 对当前项目的意义

- **ARCore 已经用了**：`ArMeasurementActivity.kt` 直接调用 ARCore 平面检测做测距。
- **Depth API 还没用**：项目里有接口占位，但实现是 stub，没有真实深度调用。
- **硬件支持**：测试用的小米手机大概率支持基础 ARCore；有没有 ToF 决定 Depth API 是硬件深度还是软件深度。

### 当前问题定位

当前在 MacBook、黑色垂直面测不准，不是「没开 ARCore」的问题，而是「只用平面 hitTest 这条路走不通」的问题。

原因：
- 黑色垂直面纹理低，平面检测不稳定。
- hitTest 依赖平面，无法在没有平面的区域工作。
- 单目运动追踪在弱纹理、反光、近距离场景下漂移增大。

### Depth API 是下一条路径，但不是免费午餐

- 优点：不依赖平面，对小物体、非平面、垂直面更友好。
- 代价：
  - 需要设备支持 Depth API。
  - 需要 App 开启 depth mode 并正确调用 API。
  - 软件深度精度有限，近距离误差明显。
  - 增加代码复杂度和运行时开销。

---

## 决策建议

| 路径 | 适用场景 | 精度 | 工作量 |
|------|----------|------|--------|
| 继续优化平面 hitTest | 地面、桌面、墙面等规则平面 | 中等 | 小 |
| 启用 ARCore Depth API | 小物体、非平面、垂直面、弱纹理 | 中等到较高（取决于硬件） | 中到大 |
| 引入外部测距设备（激光/超声波） | 高精度工业场景 | 高 | 大 |
|  hybrid 方案：hitTest + Depth API 融合 | 通用测量 App | 较高 | 大 |

推荐下一步：
1. 先检测测试机型是否支持 Depth API。
2. 如果支持，做一个最小 Depth API 原型，验证在 MacBook、黑色垂直面上的表现。
3. 如果 Depth API 软件深度表现不佳，再评估是否需要限定支持设备或引入其他传感器。

---

## 相关概念速查

- **hitTest**：将屏幕 2D 坐标投射到 AR 世界中的平面或特征点上，返回 3D 交点。
- **Anchor**：AR 世界中的虚拟锚点，用于固定虚拟对象的位置。
- **Anchor Drift**：由于追踪误差累积，Anchor 在世界坐标系中发生偏移的现象。
- **ToF（Time of Flight）**：飞行时间传感器，通过测量光脉冲往返时间获取深度。
- **运动视差深度（Motion Stereo Depth）**：利用相机移动产生的视差来估算深度，无需专用深度传感器。
