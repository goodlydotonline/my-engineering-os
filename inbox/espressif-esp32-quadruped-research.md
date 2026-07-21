---
Title: "乐鑫 ESP32 与四足机器人快速落地指南"
Date: 2026-07-13
Tags: ["#esp32", "#robotics", "#embedded", "#espressif"]
Status: "Draft"
Source: "Claude Code Skill 调研（company-research / embedded-systems / esp32-firmware-engineer / robotics-design-patterns）"
Related: ["inbox/xiaozhi-esp32-server-beginner-guide.md"]
---

# 乐鑫 ESP32 与四足机器人快速落地指南

> 目标读者：有软件基础、但无硬件经验的工程师。
> 读完本文后，你应该能够独立启动第一个基于 ESP32 的四足机器人项目。

---

## 1. 公司与产品全景

> 本章节由 `company-research` Skill 主导，结合公开资料与官方信息整理。

### 1.1 乐鑫科技（Espressif Systems）是谁？

**乐鑫信息科技（上海）股份有限公司**（Espressif Systems）是一家全球化的**无晶圆厂（Fabless）半导体设计公司**，2008 年在上海张江成立，2019 年 7 月登陆上海证券交易所科创板，股票代码 **688018**。

- **创始人**：张瑞安（Teo Swee Ann），新加坡籍，曾任职于 Transilica、Marvell、澜起科技，在蓝牙与 Wi-Fi 芯片领域拥有深厚积累。
- **核心定位**：全球领先的 **IoT / AIoT 无线通信芯片** 设计企业，主打高集成度、低功耗、高性价比、软硬件全栈开源生态。
- **全球布局**：在中国、捷克、印度、新加坡、巴西、欧洲、美国等地设有办公地或分支机构。
- **出货量**：ESP32 系列全球出货量已超 **10 亿颗**。

### 1.2 发展历程

| 年份 | 重要事件 |
|------|----------|
| 2008 | 乐鑫科技在上海张江成立，专注物联网 Wi-Fi 芯片研发 |
| 2013 | **ESP8266** 流片成功，集成 32 位 MCU，成为现象级低成本 Wi-Fi 芯片 |
| 2016 | **ESP32** 正式发布，新增蓝牙、双核处理器与更丰富外设接口 |
| 2019 | 在科创板上市；全年芯片和模组销量超 1.43 亿颗 |
| 2020+ | 推出 ESP32-S、ESP32-C、ESP32-H、ESP32-P 系列，覆盖 AI、RISC-V、Matter/Thread、高性能多媒体等方向 |

### 1.3 核心产品线

乐鑫的产品以 **Wi-Fi / 蓝牙 SoC 芯片** 为核心，覆盖芯片、模组、开发板、软件工具及云平台：

| 产品层级 | 代表产品 | 说明 |
|----------|----------|------|
| 芯片 | ESP32、ESP32-S3、ESP32-C3、ESP32-C6、ESP32-H2、ESP32-P4 | 主控 SoC |
| 模组 | ESP32-WROOM、ESP32-WROVER、ESP32-S3-WROOM 等 | 集成天线、Flash、PSRAM，即贴即用 |
| 开发板 | ESP32-DevKitC、ESP32-S3-DevKitC | 学习与原型首选 |
| 软件框架 | ESP-IDF、Arduino-ESP32、MicroPython、ESP-ADF、ESP-SR | 官方与社区开发环境 |
| 云平台 | ESP RainMaker | 一站式 AIoT 云连接方案 |

### 1.4 ESP32 系列芯片矩阵与选型

> 下列数据综合自 Espressif 官方产品页、DroneBot Workshop 与 ESPBoards.dev 等公开对比资料。

| 芯片 | 架构 | 核心/主频 | SRAM | 无线 | Matter/Thread/Zigbee | 四足机器人场景评价 |
|------|------|-----------|------|------|----------------------|--------------------|
| **ESP32** | Xtensa LX6 | 双核 240 MHz | 520 KB | Wi-Fi 4 + BT Classic + BLE 4.2 | 无 | **性价比最高**，原生 16 路 LEDC，生态最老，适合入门 |
| **ESP32-S3** | Xtensa LX7 | 双核 240 MHz | 512 KB | Wi-Fi 4 + BLE 5.0 | 无 | 带 AI 加速与 USB-OTG，适合视觉/语音扩展 |
| **ESP32-C3** | RISC-V | 单核 160 MHz | 400 KB | Wi-Fi 4 + BLE 5.0 | 无 | 便宜、低功耗，但 PWM 少，适合做腿部从控 |
| **ESP32-C6** | RISC-V | 单核 160 MHz | 512 KB | Wi-Fi 6 + BLE 5.3 | **原生支持** | 协议最全，适合未来接入 Matter/Thread 生态 |
| **ESP32-H2** | RISC-V | 单核 96 MHz | 320 KB | 仅 BLE 5.0 | **原生支持** | 无 Wi-Fi，功耗极低，适合传感器节点 |
| **ESP32-P4** | RISC-V | 双核 400 MHz | 768 KB | **无**（需外接 C6） | 无 | 性能最强，适合高端视觉/AI 预处理 |

**四足机器人主控推荐**：

- **入门/原型**：ESP32 或 ESP32-S3 + PCA9685 舵机驱动板
- **进阶/视觉**：ESP32-S3 + 摄像头 + PCA9685
- **分布式**：ESP32-S3 主控 + 4× ESP32-C3 腿部从控
- **高端/多模态**：ESP32-P4 视觉/AI + ESP32-C6 无线 + 专用舵机驱动板

### 1.5 乐鑫生态

| 生态组件 | 说明 | 四足机器人用途 |
|----------|------|----------------|
| **ESP-IDF** | 官方开源物联网开发框架，基于 FreeRTOS | 主控固件首选 |
| **Arduino-ESP32** | Arduino 风格的 ESP32 开发库 | 快速点亮舵机/传感器 |
| **MicroPython** | Python 解释器运行环境 | 快速验证算法 |
| **Rust（esp-rs）** | 社区驱动的 Rust 嵌入式生态 | 类型安全的高可靠性固件 |
| **Matter / OpenThread / Zigbee** | 智能家居互联标准 | 未来接入 HomeKit/Alexa/Google |
| **ESP-ADF / ESP-SR** | 音频与语音识别框架 | 语音交互扩展 |

### 1.6 竞品对比

| 平台 | 厂商 | 内置无线 | 优势 | 劣势 | 四足机器人适合度 |
|------|------|----------|------|------|------------------|
| **ESP32** | 乐鑫 | Wi-Fi + BT/BLE | 成本低、生态大、无线集成 | 功耗非最低、实时性中等 | ⭐⭐⭐⭐⭐ |
| **STM32** | ST | 大多无 | 工业级、外设丰富、实时强 | 需外接无线、学习曲线陡 | ⭐⭐⭐⭐ |
| **RP2040** | 树莓派 | 无（Pico W 外接） | 极低成本、PIO 灵活 | 无内置无线、Flash 外置 | ⭐⭐⭐ |
| **nRF52** | Nordic | BLE 5.x | 超低功耗、BLE 强 | 无 Wi-Fi、价格较高 | ⭐⭐⭐ |
| **ESP8266** | 乐鑫 | Wi-Fi | 成本最低 | 无蓝牙、性能弱 | ⭐⭐ |

**结论**：对于需要**无线调参 + 低成本 + 丰富社区**的四足机器人项目，ESP32 系列是首选。

---

## 2. 嵌入式与机器人基础知识

> 本章节由 `embedded-systems` Skill 主导，面向有软件基础但无硬件经验的读者。

### 2.1 MCU vs SoC

- **MCU（Microcontroller Unit，微控制器单元）**：把 CPU、RAM/Flash、定时器、GPIO、ADC 等集成在单一芯片上的微型计算机，强调“单片即可运行”。
- **SoC（System on Chip，片上系统）**：集成度更高，除 MCU 部件外，还包含无线基带（Wi-Fi / Bluetooth）、DSP、AI 加速器、USB、摄像头接口等。

ESP32 同时具备 MCU 与 SoC 特征：你可以把它当作实时控制器，也可以把它当作带 Wi-Fi/BLE 的小型联网计算机。

### 2.2 FreeRTOS 在 ESP32 上的作用

四足机器人需要同时做多件事：读取 IMU、解算姿态、驱动 12 路舵机、接收遥控指令、上报日志。**RTOS（Real-Time Operating System，实时操作系统）** 通过任务调度，让多个功能看起来并行执行，并提供确定性的响应时间。

ESP-IDF 默认把 FreeRTOS 扩展到两个 CPU 核心：

- **PRO_CPU（协议核）**：通常跑 Wi-Fi / Bluetooth 协议栈、TCP/IP、NVS。
- **APP_CPU（应用核）**：跑用户业务逻辑，如控制算法。

四足机器人典型任务划分：

| 任务名 | 周期 | 核心 | 优先级 | 职责 |
|--------|------|------|--------|------|
| `leg_ctrl_task` | 1–2 ms | APP_CPU | 高 | 读取目标角度、生成 PWM、同步 12 路舵机 |
| `imu_fusion_task` | 5–10 ms | APP_CPU | 中高 | 读取 IMU、运行姿态解算 |
| `gait_plan_task` | 10–20 ms | APP_CPU | 中 | 步态生成、足端轨迹规划 |
| `cmd_rx_task` | 事件驱动 | PRO_CPU | 中低 | 接收 BLE / Wi-Fi / UART 遥控指令 |
| `telemetry_task` | 50–100 ms | PRO_CPU | 低 | 上传关节角、姿态、电量 |
| `power_mon_task` | 1000 ms | APP_CPU | 低 | ADC 采样电池电压、触发低电量保护 |

### 2.3 无线协议

| 协议 | ESP32 | ESP32-S3 | ESP32-C6 | 说明 |
|------|-------|----------|----------|------|
| **Wi-Fi** | 802.11 b/g/n | 802.11 b/g/n | 802.11 b/g/n/ax | 2.4 GHz，用于图传、OTA、HTTP 调参 |
| **Bluetooth Classic** | 有 | 无 | 无 | 适合 SPP 串口透传、音频 |
| **BLE** | 4.2 | 5.0 | 5.3 | 低功耗遥控、传感器广播 |
| **802.15.4 / Thread** | 无 | 无 | 有 | IEEE 802.15.4 网状网络 |
| **Zigbee** | 无 | 无 | 有 | 成熟低功耗网状网络协议 |
| **Matter** | 需外接 | 需外接 | 原生支持 | 跨生态智能家居标准 |

**四足机器人建议**：手机/手柄遥控用 **BLE**，上位机调参与 OTA 用 **Wi-Fi**，多机协同可考虑 **Thread**（ESP32-C6）。

### 2.4 电机控制

#### PWM（Pulse Width Modulation，脉冲宽度调制）

通过快速切换高低电平，改变“高电平占一个周期的比例”（占空比），从而等效输出不同电压或控制舵机角度。

```text
周期 20 ms，占空比 7.5%：
|---|                 |---|
|   |                 |   |
|   |_________________|   |_________________
  1.5 ms              20 ms
```

#### 伺服舵机（Servo）

常见型号：

| 型号 | 扭矩 | 特点 | 适用 |
|------|------|------|------|
| **SG90** | ~1.8 kg·cm | 塑料齿、便宜、易烧毁 | 微型四足、学习调试 |
| **MG90S** | ~2.2 kg·cm | 金属齿、可靠性高 | 小型四足 |
| **DS3218** | ~20 kg·cm | 大扭矩数字舵机 | 中大型四足 |

舵机控制脉冲（典型）：

| 脉宽 | 角度 |
|------|------|
| 0.5 ms | 0° |
| 1.5 ms | 90° |
| 2.5 ms | 180° |

#### 直流电机与步进电机

- **直流电机（DC Motor）**：通过 H 桥（如 L298N、TB6612、DRV8833）控制正反转与速度。
- **步进电机（Stepper Motor）**：开环定位准确，但动态响应与扭矩密度不如舵机，四足机器人**通常不用**。

### 2.5 传感器与总线

| 总线 | 线数 | 速度 | 距离 | 典型用途 | ESP32 支持 |
|------|------|------|------|----------|------------|
| **I2C** | 2（SDA、SCL）+ 电源 | 100/400 kHz | 短（<1 m） | IMU、OLED、PCA9685 | 2 路 |
| **SPI** | 4（MOSI、MISO、SCK、CS）+ 电源 | 10 MHz+ | 较短 | 显示屏、Flash、LoRa | 4 路 |
| **UART** | 2（TX、RX）+ GND | 常见 115200 bps | 数米 | GPS、串口屏、调试 | 3 路 |
| **ADC** | 1（模拟输入）+ 电源 | 取决于采样率 | 极短 | 电池电压、电位器 | 2 组 12 bit |
| **GPIO** | 1 根信号线 | 软件决定 | 极短 | LED、按键、中断 | 34 个可编程 |

**关键接线规则**：

- **I2C 必须上拉**：SDA 和 SCL 是开漏输出，需外接 4.7 kΩ 上拉电阻到 3.3 V（很多模块已内置）。
- **UART 交叉接线**：A 的 TX 接 B 的 RX，A 的 RX 接 B 的 TX。
- **ADC 输入不得超过 3.3 V**，测量电池电压必须用电阻分压。

### 2.6 电源管理与低功耗设计

四足机器人典型电源架构：

```text
锂电池 2S/3S（7.4 V / 11.1 V）
        │
        ├─→ Buck 5 V  → 舵机电源（大电流）
        │
        ├─→ Buck 3.3 V / LDO → ESP32、传感器、LED
        │
        └─→ 电压分压 → ADC 监测电池电量
```

各模块电流估算（典型值）：

| 模块 | 工作状态电流 | 备注 |
|------|-------------|------|
| ESP32 CPU + Wi-Fi | 100–240 mA | 发送数据时峰值更高 |
| ESP32 BLE 连接 | 50–120 mA | 取决于连接间隔 |
| 12× MG90S 舵机 | 600–2000 mA | 堵转时单颗可达 700 mA |
| MPU6050 | 3.8 mA | 低功耗模式 10 μA |

**要点**：四足机器人的功耗大头是**舵机**，ESP32 本身的低功耗意义有限。重点是：

1. 舵机电源独立、线径足够粗；
2. 电池选型按“所有舵机同时堵转”的极限电流设计；
3. 增加总电源开关与过流保护。

---

## 3. ESP32 技术选型与开发环境

> 本章节由 `esp32-firmware-engineer` Skill 主导，聚焦“复制这条命令、连接这根线”的落地粒度。

### 3.1 开发环境搭建

#### 方案 A：ESP-IDF（官方，推荐）

```bash
# macOS 依赖
brew install cmake ninja dfu-util ccache python@3.11

# Ubuntu/Debian 依赖
sudo apt-get install git wget flex bison gperf python3 python3-pip \
  python3-venv cmake ninja-build ccache libffi-dev libssl-dev dfu-util \
  libusb-1.0-0

# 克隆 ESP-IDF
mkdir -p ~/esp && cd ~/esp
git clone -b v5.2.1 --recursive https://github.com/espressif/esp-idf.git

# 安装工具链
cd ~/esp/esp-idf
./install.sh esp32,esp32s3,esp32c3

# 每新终端执行
. ./export.sh

# 验证
idf.py --version
```

**Windows**：下载 `esp-idf-tools-setup-online-2.24.exe`，勾选 ESP-IDF v5.2 + esp32/esp32s3/esp32c3，安装后从开始菜单启动 **“ESP-IDF 5.2 PowerShell”**。

#### 方案 B：PlatformIO（VS Code）

1. VS Code 安装扩展 **PlatformIO IDE**
2. 左侧 PlatformIO 图标 → **Create New Project**
3. 填写：
   - Name：`quadruped-esp32`
   - Board：`Espressif ESP32 Dev Module` 或 `Adafruit ESP32-S3 Feather`
   - Framework：`ESP-IDF` 或 `Arduino`
4. 点击 **Finish**

```ini
; platformio.ini 示例
[env:esp32-s3]
platform = espressif32
board = esp32-s3-devkitc-1
framework = espidf
monitor_speed = 115200
```

#### 方案 C：Arduino IDE

1. 文件 → 首选项 → 附加开发板管理器网址：
   ```text
   https://espressif.github.io/arduino-esp32/package_esp32_index.json
   ```
2. 工具 → 开发板 → 开发板管理器 → 搜索 **ESP32** → 安装
3. 工具 → 开发板 → 选择 `ESP32 Dev Module`
4. 选择端口，点击上传

**选型建议**：

| 场景 | 推荐工具 |
|------|----------|
| 四足机器人主控固件 | **ESP-IDF** |
| 快速验证舵机/蓝牙 | PlatformIO + Arduino |
| 完全零基础点亮 LED | Arduino IDE |
| 需要 JTAG 调试/OTA/分区优化 | **ESP-IDF** |

### 3.2 芯片选型决策表

| 芯片 | 核心/主频 | 无线 | PWM 通道 | 四足机器人评价 |
|------|-----------|------|----------|----------------|
| **ESP32** | Xtensa LX6 双核 240MHz | Wi-Fi 4 + BT 4.2 | 16 路 | 性价比最高，PWM 多，生态最老 |
| **ESP32-S3** | Xtensa LX7 双核 240MHz | Wi-Fi 4 + BLE 5 | 8 路 | 带向量指令，适合跑神经网络步态 |
| **ESP32-C3** | RISC-V 单核 160MHz | Wi-Fi 4 + BLE 5 | 6 路 | 便宜、低功耗，适合做腿部从控 |
| **ESP32-C6** | RISC-V 160MHz | Wi-Fi 6 + BLE 5.3 + 802.15.4 | 6 路 | 协议最全，适合通信枢纽 |
| **ESP32-H2** | RISC-V 96MHz | BLE 5 + 802.15.4 | 4 路 | 无 Wi-Fi，超低功耗传感器节点 |
| **ESP32-P4** | RISC-V 双核 400MHz | **无内置无线** | 丰富 | 需外接无线，适合高端视觉/AI |

> 关键提示：不要只用芯片原生 PWM 驱动 12 路舵机。**PCA9685（I2C 16 路 PWM 驱动）是标准答案**。

### 3.3 固件开发最佳实践

#### 推荐项目结构

```text
quadruped-fw/
├── CMakeLists.txt
├── sdkconfig.defaults
├── partitions.csv
├── main/
│   ├── CMakeLists.txt
│   ├── app_main.c
│   ├── kinematics.c
│   ├── servo_driver.c
│   ├── gait_engine.c
│   ├── imu_task.c
│   ├── wireless_task.c
│   └── shell_cmd.c
└── components/
    ├── pwm_pca9685/
    └── imu_mpu6050/
```

#### FreeRTOS 任务划分

| 任务 | 优先级 | 周期 | 职责 |
|------|--------|------|------|
| `task_motion_ctrl` | 高 | 1–5 ms | 逆运动学、插值、输出 PWM |
| `task_gait_engine` | 中高 | 10–20 ms | 状态机、步态切换 |
| `task_imu` | 中 | 5–10 ms | 读取 IMU、滤波、姿态估计 |
| `task_wireless` | 中低 | 事件驱动 | Wi-Fi/BLE 通信、命令解析 |
| `task_telemetry` | 低 | 100 ms | 日志、电量、状态上报 |

**关键原则**：

- 高优先级任务禁止长时间阻塞、禁止 `printf`、禁止网络发送；
- 运动控制任务只从 Queue 拿目标角度，计算后立即写 PWM，然后 `vTaskDelayUntil`；
- ISR 里只做 `xQueueSendFromISR`，具体处理交给任务。

#### 分区表示例（4MB Flash，支持 OTA）

```csv
# Name,   Type, SubType, Offset,  Size,    Flags
nvs,      data, nvs,     0x9000,  0x6000,
otadata,  data, ota,     0xf000,  0x2000,
app0,     app,  ota_0,   0x10000, 0x1E0000,
app1,     app,  ota_1,   0x1F0000,0x1E0000,
spiffs,   data, spiffs,  0x3D0000,0x20000,
coredump, data, coredump,0x3F0000,0x10000,
```

对应 `sdkconfig.defaults`：

```ini
CONFIG_PARTITION_TABLE_CUSTOM=y
CONFIG_PARTITION_TABLE_CUSTOM_FILENAME="partitions.csv"
CONFIG_ESPTOOLPY_FLASHSIZE_4MB=y
```

### 3.4 常见坑

| 问题 | 说明 | 避坑 |
|------|------|------|
| Strapping 引脚 | GPIO0、2、5、12、15 等影响启动模式 | 上电瞬间不要拉低/拉高这些脚 |
| Flash 占用引脚 | GPIO6-11 接内部 Flash | **禁止外接设备** |
| Input-Only 引脚 | GPIO34-39 只能输入 | 不能输出 PWM |
| S3/C3 无高速 LEDC | 只有 `LEDC_LOW_SPEED_MODE` | 不要写 `HIGH_SPEED` |
| USB 电流不够 | USB 仅 500 mA | 舵机必须独立 5V 供电 |
| 共地问题 | ESP32 GND 必须与舵机电源 GND 相连 | 否则舵机抖动、复位 |
| 看门狗触发 | 任务死循环无 `vTaskDelay` | 循环中必须释放 CPU |
| 堆内存不足 | 避免大数组放栈上 | 任务栈 4096 起步，监控堆余量 |

### 3.5 调试工具

| 工具 | 用途 | 关键命令 |
|------|------|----------|
| `idf.py monitor` | 日志、panic 解码 | `idf.py -p /dev/ttyUSB0 flash monitor` |
| JTAG / USB-Serial-JTAG | 单步调试 | `openocd -f board/esp32s3-builtin.cfg` |
| 逻辑分析仪 | PWM、I2C/SPI/UART 时序 | PulseView / Saleae Logic |
| 示波器 | 电源纹波、信号完整性 | 100 MHz 双通道足够 |

### 3.6 最小 Demo：ESP-IDF 驱动 SG90 舵机

#### 硬件准备

- ESP32-DevKitC ×1
- SG90 舵机 ×1
- 5V/2A 独立电源 ×1
- 杜邦线若干

#### 接线（文字版）

```text
SG90 舵机              ESP32-DevKitC           独立 5V 电源
─────────────────────────────────────────────────────────────
橙色/黄色 信号线  ────>  GPIO18（通过 1kΩ 电阻可选）
红色 电源正极    ─────────────────────────────>  5V+
棕色/黑色 电源负极 ────  GND  <───────────────  5V-
                       ↑
            必须共地：ESP32 GND 与电源负极相连
```

#### 完整代码：`main/servo_demo.c`

```c
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "driver/ledc.h"
#include "esp_log.h"

static const char *TAG = "servo";

#define SERVO_PWM_GPIO      18
#define SERVO_PWM_CHANNEL   LEDC_CHANNEL_0
#define SERVO_PWM_TIMER     LEDC_TIMER_0
#define SERVO_PWM_MODE      LEDC_LOW_SPEED_MODE
#define PWM_FREQ_HZ         50
#define PWM_RESOLUTION      LEDC_TIMER_13_BIT
#define PWM_PERIOD_US       20000
#define DUTY_MIN_US         500
#define DUTY_MAX_US         2500

static uint32_t angle_to_duty(float angle)
{
    if (angle < 0)   angle = 0;
    if (angle > 180) angle = 180;

    float us = DUTY_MIN_US + (angle / 180.0f) * (DUTY_MAX_US - DUTY_MIN_US);
    uint32_t max_duty = (1U << PWM_RESOLUTION) - 1;
    uint32_t duty = (uint32_t)((us / PWM_PERIOD_US) * max_duty);
    return duty;
}

static void servo_init(void)
{
    ledc_timer_config_t timer_cfg = {
        .speed_mode       = SERVO_PWM_MODE,
        .duty_resolution  = PWM_RESOLUTION,
        .timer_num        = SERVO_PWM_TIMER,
        .freq_hz          = PWM_FREQ_HZ,
        .clk_cfg          = LEDC_AUTO_CLK,
    };
    ESP_ERROR_CHECK(ledc_timer_config(&timer_cfg));

    ledc_channel_config_t channel_cfg = {
        .gpio_num       = SERVO_PWM_GPIO,
        .speed_mode     = SERVO_PWM_MODE,
        .channel        = SERVO_PWM_CHANNEL,
        .timer_sel      = SERVO_PWM_TIMER,
        .duty           = angle_to_duty(90),
        .hpoint         = 0,
        .intr_type      = LEDC_INTR_DISABLE,
    };
    ESP_ERROR_CHECK(ledc_channel_config(&channel_cfg));

    ESP_LOGI(TAG, "servo init done, GPIO=%d, freq=%dHz", SERVO_PWM_GPIO, PWM_FREQ_HZ);
}

static void servo_set_angle(float angle)
{
    uint32_t duty = angle_to_duty(angle);
    ESP_ERROR_CHECK(ledc_set_duty(SERVO_PWM_MODE, SERVO_PWM_CHANNEL, duty));
    ESP_ERROR_CHECK(ledc_update_duty(SERVO_PWM_MODE, SERVO_PWM_CHANNEL));
    ESP_LOGI(TAG, "set angle=%.1f duty=%lu", angle, duty);
}

void app_main(void)
{
    servo_init();

    while (1) {
        for (float a = 0; a <= 180; a += 10) {
            servo_set_angle(a);
            vTaskDelay(pdMS_TO_TICKS(100));
        }
        for (float a = 180; a >= 0; a -= 10) {
            servo_set_angle(a);
            vTaskDelay(pdMS_TO_TICKS(100));
        }
    }
}
```

#### `main/CMakeLists.txt`

```cmake
idf_component_register(SRCS "servo_demo.c"
                       INCLUDE_DIRS ".")
```

#### 编译/烧录/监控

```bash
cd ~/esp/servo_demo
idf.py set-target esp32
idf.py build
idf.py -p /dev/ttyUSB0 flash
idf.py -p /dev/ttyUSB0 monitor
```

> Windows 用户把 `/dev/ttyUSB0` 换成 `COM3`（具体端口号以设备管理器为准）。

---

## 4. 四足机器人架构与实现

> 本章节由 `robotics-design-patterns` Skill 主导，输出软件架构与实现路径。

### 4.1 四足机器人基本结构

#### 为什么选四足？

| 特性 | 四足 | 六足 | 轮式 |
|------|------|------|------|
| 地形适应 | 强 | 很强但腿多更重 | 弱 |
| 自由度/成本 | 12 DOF 适中 | 18 DOF 成本高 | 低 |
| 控制复杂度 | 中等 | 高 | 低 |
| 动态性能 | 可小跑、跳跃 | 慢但稳 | 快但受限 |

> **DOF（Degree of Freedom，自由度）**：一个关节能独立运动的方向数。旋转舵机通常提供 1 DOF。

#### 典型 12-DOF 构型

```text
         躯干（Body / Trunk）
        ┌─────────────────┐
   左前 LF │                 │ 右前 RF
   左后 LH │                 │ 右后 RH
        └─────────────────┘

每条腿 = 3 个关节 = 3 DOF
全身 = 4 腿 × 3 DOF = 12 DOF
```

| 关节 | 名称 | 轴方向 | 作用 |
|------|------|--------|------|
| 髋关节 1（Coxa） | 根关节 | 水平旋转（偏航 Yaw） | 腿向身体前后摆动 |
| 髋关节 2（Femur） | 大腿 | 垂直俯仰（Pitch） | 抬腿/落腿 |
| 膝关节（Tibia） | 小腿 | 垂直俯仰（Pitch） | 伸展/收缩，决定足端高度 |

### 4.2 运动学与逆运动学

| | 正运动学 FK | 逆运动学 IK |
|---|---|---|
| 输入 | 各关节角度 θ₁, θ₂, θ₃ | 足端目标位置 (x, y, z) |
| 输出 | 足端在机体坐标系中的位置 | 各关节应转到的角度 |
| 用途 | 仿真、验证、状态显示 | 实际控制、步态生成 |

#### 单腿几何 IK（简化版）

把一条腿投影到侧面平面，退化为 2 连杆平面问题：

```cpp
// 2D 平面 IK，输入：足端到髋关节2的水平/垂直距离
struct JointAngles {
    float femur;  // 大腿角度
    float tibia;  // 小腿角度
};

JointAngles solve2D(float x, float z, float L1, float L2) {
    float d2 = x*x + z*z;
    float d = sqrt(d2);

    float cos_tibia = (d2 - L1*L1 - L2*L2) / (2.0 * L1 * L2);
    cos_tibia = constrain(cos_tibia, -1.0, 1.0);
    float tibia = acos(cos_tibia);

    float alpha = atan2(z, x);
    float beta = acos((L1*L1 + d2 - L2*L2) / (2.0 * L1 * d));
    float femur = alpha + beta;

    return {femur, tibia};
}
```

#### 完整 3-DOF 单腿 IK

```cpp
// 机体坐标系下的足端目标 (x, y, z)
float coxa = atan2(y, x);
float r = sqrt(x*x + y*y) - L_COXA;
float z_hip = z;

auto leg = solve2D(r, z_hip, L_FEMUR, L_TIBIA);
return {coxa, leg.femur, leg.tibia};
```

> **DH 参数（Denavit-Hartenberg）**：一种标准化的连杆坐标系建立方法，用 4 个参数描述相邻关节之间的变换。初学者建议先用**几何 IK**，跑稳后再引入 DH。

### 4.3 步态控制

#### 四种基础步态

| 步态 | 对角腿关系 | 同时离地腿数 | 速度 | 稳定性 | 适用场景 |
|------|------------|--------------|------|--------|----------|
| **Crawl（爬行）** | 每次只移 1 腿 | 最多 1 | 慢 | 最高 | 崎岖地形、调试 |
| **Trot（小跑）** | 对角腿同步 | 2 | 中 | 高 | 日常行走、最常用 |
| **Pace（踱步）** | 同侧腿同步 | 2 | 中 | 中 | 狭窄通道 |
| **Bound（跳跃）** | 前后腿成对 | 4（腾空相） | 快 | 低 | 高速奔跑 |

#### 步态状态机（FSM）

```text
        ┌─────────┐
        │  IDLE   │
        └────┬────┘
             │ start
             ▼
        ┌─────────┐
        │ STANCE  │◄─────────┐
        │ (支撑相) │          │
        └────┬────┘          │
             │ timer          │
             ▼                │
        ┌─────────┐           │
        │ SWING   │───────────┘
        │ (摆动相) │
        └─────────┘
```

#### 足端轨迹

常用摆线或贝塞尔曲线规划摆动相：

```cpp
// 摆动相参数 t ∈ [0, 1]
float swing_height = 30.0; // mm
float swing_x(float t, float step_length) {
    return step_length * (t - 0.5 * sin(2*PI*t) / PI);
}
float swing_z(float t, float swing_height) {
    return swing_height * sin(PI * t);
}
```

### 4.4 控制架构

#### 主控 vs 协处理器

| 方案 | 主控 | 协处理器 | 优点 | 缺点 |
|------|------|----------|------|------|
| A | ESP32 | PCA9685 | 成本低、I2C 简单 | PWM 精度一般 |
| B | ESP32 | STM32F103/F4 | 实时 PWM、反馈 | 需维护双固件 |
| C | ESP32 | RP2040 | 便宜、可编程 PWM | 生态不如 STM32 |
| D | ESP32 直连 12 舵机 | 无 | 极简 | GPIO/PWM 资源紧张 |

> **PCA9685**：16 通道 12 位 PWM 驱动芯片，通过 I2C 控制，适合驱动大量舵机。

#### 推荐分层架构

```text
┌──────────────────────────────────────────┐
│  APPLICATION  应用层                      │
│  遥控指令解析、任务模式、参数配置、OTA      │
├──────────────────────────────────────────┤
│  BEHAVIOR     行为层                      │
│  行为树 / 状态机：选步态、急停、恢复        │
├──────────────────────────────────────────┤
│  GAIT         步态层                      │
│  Crawl/Trot/Pace/Bound 轨迹生成           │
├──────────────────────────────────────────┤
│  KINEMATICS   运动学层                    │
│  FK/IK、机体姿态补偿、足端工作空间检查      │
├──────────────────────────────────────────┤
│  MOTION CTRL  运动控制层                  │
│  关节伺服目标生成、平滑插值、安全限幅       │
├──────────────────────────────────────────┤
│  HAL          硬件抽象层                  │
│  PCA9685 驱动、IMU 驱动、ADC 电池检测       │
├──────────────────────────────────────────┤
│  HARDWARE     硬件层                      │
│  ESP32、舵机、IMU、电池、稳压模块           │
└──────────────────────────────────────────┘
```

#### 循环频率建议

| 任务 | 频率 | 说明 |
|------|------|------|
| 通信接收（UDP/BLE） | 50 Hz | 非实时，容忍丢包 |
| 步态规划 | 50 Hz | 生成下一步足端目标 |
| 逆运动学 | 100 Hz | 关节角度更新 |
| PWM 输出 | 50–333 Hz | 舵机控制周期 |
| IMU 读取 | 200–1000 Hz | 姿态反馈、摔倒检测 |

### 4.5 通信方案

| 方案 | 协议 | 优点 | 缺点 | 适合场景 |
|------|------|------|------|----------|
| **Wi-Fi 遥控** | UDP / WebSocket | 远距离、可传视频、手机/PC 通用 | 耗电、延迟波动 | 室外、调试 |
| **蓝牙手柄** | BLE HID / 串口蓝牙模块 | 低功耗、即连即玩 | 带宽低 | 日常把玩 |
| **ROS2 集成** | micro-ROS | 与算法生态无缝衔接 | 学习曲线陡 | 研究、SLAM |

#### UDP 遥控协议示例

```cpp
// 控制器 → ESP32，12 字节
struct CommandPacket {
    uint8_t  header = 0xA5;   // 帧头
    int8_t   vx;              // 前进速度 [-127, 127]
    int8_t   vy;              // 侧移速度
    int8_t   yaw_rate;        // 旋转速率
    int8_t   gait;            // 0=idle, 1=trot, 2=crawl...
    uint8_t  flags;           // 急停、站起、坐下
    uint16_t crc16;           // 校验
};
```

#### micro-ROS 集成

```text
PC/树莓派 (ROS2 Humble)
    │
    │ DDS / UDP
    ▼
ESP32 (micro-ROS client)
    │
    │ I2C / UART
    ▼
PCA9685 + 舵机
```

常用 Topic：

- `/cmd_vel`：速度指令
- `/joint_states`：关节状态反馈
- `/imu/data`：IMU 数据
- `/battery/state`：电池状态

### 4.6 开源参考项目

| 项目 | 主控/平台 | 特点 | 适合学习 |
|------|-----------|------|----------|
| **SpotMicro** | Raspberry Pi + PCA9685 | 仿 Boston Dynamics Spot，3D 打印结构完整 | 机械结构、全身控制 |
| **OpenQuadruped** | Arduino/ESP32 | 低成本入门，代码简洁 | 初学者 first robot |
| **Stanford Pupper** | Raspberry Pi + custom PCB | 学术研究导向，强化学习友好 | 步态、RL |
| **Pupper** | Pi + custom driver | 社区版，文档丰富 | 运动控制 |
| **Moco ML** | ESP32 / various | 微型四足，强调低成本与可扩展 | ESP32 生态 |
| **QuadrupedRobot（国内）** | STM32 | 国产方案，资料中文多 | 嵌入式实时控制 |

**选型建议**：

- 纯 ESP32 学习：OpenQuadruped / Moco ML
- 想接 ROS2：SpotMicro 改 ESP32-S3 + micro-ROS
- 做强化学习：Stanford Pupper

---

## 5. 快速落地：从 0 到完整四足机器人

> 本章节整合 `robotics-design-patterns` 与 `esp32-firmware-engineer` 的输出，给出可执行路径。

### 5.1 从 0 到 1 演进路线

```text
Phase 1: 单舵机验证
    └── 目标：让 1 个 SG90 按角度摆动
    └── 产出：能串口输入角度并运动的单舵机平台

Phase 2: 单腿 3-DOF
    └── 目标：控制一条腿的 3 个舵机
    └── 产出：给定 (x,y,z) 能到位的单腿平台

Phase 3: 四条腿支撑
    └── 目标：4 条腿同时安装到躯干
    └── 产出：能站立并保持水平的四足平台

Phase 4: 基础步态
    └── 目标：实现 Crawl → Trot
    └── 产出：能稳定行走的机器人

Phase 5: 遥控与传感器
    └── 目标：加入 Wi-Fi/BLE 遥控、IMU、电池保护

Phase 6: 扩展
    └── 目标：ROS2、视觉、地形适应、语音交互
```

### 5.2 推荐硬件清单

| 类别 | 推荐型号 | 数量 | 参考价格区间 | 购买关键词 |
|------|----------|------|--------------|------------|
| **主控** | ESP32-DevKitC / ESP32-S3-DevKitC | 1 | ¥25–80 | "ESP32 DevKitC" / "ESP32-S3-DevKitC" |
| **舵机** | SG90（入门）/ MG90S（中端）/ DS3218（大扭矩） | 12 | SG90 ¥5×12 / MG90S ¥15×12 / DS3218 ¥60×12 | "SG90 舵机" / "MG90S 金属齿" |
| **驱动板** | PCA9685 16 路 PWM | 1 | ¥10–25 | "PCA9685 舵机驱动板" |
| **IMU** | MPU6050 / MPU9250 / BMI270 | 1 | ¥10–40 | "MPU6050 GY-521" |
| **电池** | 2S–3S LiPo（7.4–11.1 V） | 1–2 | ¥50–150 | "航模 2S LiPo" / "XT60 插头" |
| **稳压模块** | LM2596 / Buck 5 V@5 A | 1 | ¥15–40 | "DC-DC 降压 5V 5A" |
| **结构件** | SpotMicro 3D 打印件 | 1 套 | ¥50–200（打印费） | "SpotMicro STL" |
| **电源开关** | 船型开关 + XT60 插头 | 1 | ¥10–20 | "XT60 开关" |
| **杜邦线/舵机延长线** | 母对母、公对母 | 若干 | ¥10–30 | "杜邦线 40P" / "舵机延长线" |

> **LiPo（锂聚合物电池）**：能量密度高、放电倍率大，但过放会损坏甚至起火，必须加低压报警或保护板。

### 5.3 接线示意

```text
        7.4 V LiPo
            │
            ▼
    ┌───────────────┐
    │   稳压模块 5V   │──────→ 12 个舵机电源
    │  (Buck 5V/5A)  │
    └───────────────┘
            │
            ▼
    ┌───────────────┐
    │     ESP32      │──I2C──→ PCA9685 ──PWM──→ 舵机信号
    │                │
    │                │──I2C──→ MPU6050 IMU
    │                │
    │                │──ADC──→ 电池分压检测
    └───────────────┘
```

### 5.4 开源库与示例代码

| 库 | 用途 | 来源 |
|----|------|------|
| **ESP32Servo** | ESP32 原生 PWM 控制舵机 | Arduino 库管理器 |
| **Adafruit_PWMServoDriver** | PCA9685 驱动 | Arduino 库管理器 |
| **I2Cdev / MPU6050** | IMU 读取 | Arduino 库管理器 |
| **Kinematics** | 四足运动学 | GitHub 搜索 `quadruped kinematics arduino` |
| **micro-ROS** | ROS2 集成 | micro-ROS 官网 + PlatformIO |
| **py_trees** | PC 端行为树 | `pip install py_trees` |

#### PCA9685 批量控制 12 路舵机

```cpp
#include <Wire.h>
#include <Adafruit_PWMServoDriver.h>

Adafruit_PWMServoDriver pwm = Adafruit_PWMServoDriver(0x40);

#define SERVO_MIN 150  // 对应 0°
#define SERVO_MAX 600  // 对应 180°

void setup() {
    pwm.begin();
    pwm.setPWMFreq(50);
}

void setAngle(uint8_t channel, float angle) {
    int pulse = map(angle, 0, 180, SERVO_MIN, SERVO_MAX);
    pwm.setPWM(channel, 0, pulse);
}

void loop() {
    for (int ch = 0; ch < 12; ch++) {
        setAngle(ch, 90); // 全部回中
    }
    delay(1000);
}
```

#### 12 舵机通道映射建议

| 腿 | Coxa | Femur | Tibia |
|----|------|-------|-------|
| LF | 0 | 1 | 2 |
| RF | 3 | 4 | 5 |
| LH | 6 | 7 | 8 |
| RH | 9 | 10 | 11 |

### 5.5 产品化建议

| 阶段 | 预算范围 | 关键决策 |
|------|----------|----------|
| 原型验证 | ¥300–800 | SG90 + 3D 打印 |
| 稳定迭代 | ¥1000–2000 | MG90S/DS3218 + 好电池 |
| 可展示产品 | ¥3000+ | 金属结构、IMU+视觉、外壳 |

**可靠性 checklist**：

- [ ] 舵机电源独立走线，不从 ESP32 5V 引脚取电
- [ ] 所有信号共地（ESP32 GND、PCA9685 GND、电池 GND）
- [ ] 机械关键连接点用螺丝+螺母，避免仅靠 3D 打印卡扣
- [ ] 软件加机械双保险，防止舵机堵转烧毁
- [ ] 电池低压报警、过流保护、船型开关急停

---

## 6. AI 扩展与产品化方向

> 本章节结合 `company-research`、`esp32-firmware-engineer`、`robotics-design-patterns` 的综合输出，给出从“能走”到“好玩”的扩展路径。

### 6.1 语音交互

ESP32-S3 或 ESP32-P4 可运行轻量级 ASR + LLM + TTS 链路：

```text
麦克风 → ESP32-S3（ESP-SR 本地唤醒）→ Wi-Fi → 云端 LLM → 云端 TTS → 喇叭
                                    ↓
                              本地命令："坐下" / "站起来" / "往前走"
```

**参考方案**：

- 本地唤醒词：ESP-SR（乐鑫官方语音框架）
- 云端 LLM：OpenAI / Claude / 国产大模型 API
- 云端 TTS：Edge-TTS、阿里云、科大讯飞
- 也可复用 [`inbox/xiaozhi-esp32-server-beginner-guide.md`](inbox/xiaozhi-esp32-server-beginner-guide.md) 中的 xiaozhi-esp32-server 架构

### 6.2 视觉

| 方案 | 硬件 | 用途 |
|------|------|------|
| ESP32-CAM / ESP32-S3-CAM | OV2640 / OV3660 | 图传、简单目标检测 |
| XIAO ESP32-S3 Sense | 内置摄像头+麦克风 | FPV + 语音一体化 |
| 树莓派 + ESP32 | Pi 跑 OpenCV/YOLO，ESP32 做下位机 | 复杂视觉、SLAM |

### 6.3 遥控 App

- **Web 遥控**：ESP32 开启 Wi-Fi AP 或接入局域网，跑一个轻量 WebSocket 页面，手机浏览器直接控制。
- **微信小程序 / App**：通过 BLE 或 Wi-Fi 连接，发送速度/步态/姿态指令。
- **蓝牙手柄**：Xbox / PS4 手柄通过 BLE HID 协议直连 ESP32。

### 6.4 产品化关键考虑

1. **成本**：批量时 BOM 要控制在目标售价的 30% 以内；优先使用国产替代。
2. **安全**：硬件 E-Stop、软件看门狗、关节限位、电池低压保护缺一不可。
3. **结构**：从 3D 打印 PLA 验证，逐步过渡到 PETG、尼龙、铝合金。
4. **可维护性**：固件 OTA、参数可配置、日志远程上传、模块化代码结构。
5. **认证**：若销售，需考虑 CE/FCC/RoHS、电池运输认证（UN38.3）。
6. **用户体验**：开箱即用 > 复杂调参；提供手机 App 与演示视频。

---

## 7. 参考资源与下一步

### 7.1 官方资源

- [乐鑫官网](https://www.espressif.com)
- [ESP-IDF 官方文档](https://docs.espressif.com/projects/esp-idf/)
- [ESP32 论坛](https://esp32.com)
- [乐鑫 GitHub](https://github.com/espressif)

### 7.2 选型参考

- [DroneBot Workshop – ESP32 Selection Guide 2026](https://dronebotworkshop.com/esp32-2026/)
- [ESPBoards.dev – ESP32 Comparison Chart](https://www.espboards.dev/blog/esp32-soc-options/)
- [OpenELAB – ESP32-C3 vs ESP32-S3](https://openelab.com/blogs/learn/esp32-c3-vs-esp32-s3-vs-esp32-c3-mini-key-comparison)

### 7.3 四足机器人开源项目

- [SpotMicro](https://github.com/FlorianGG/SpotMicro)
- [OpenQuadruped](https://github.com/adham-elaraby/OpenQuadruped)
- [Stanford Pupper](https://stanfordstudentrobotics.org/pupper)
- [Pupper](https://github.com/stanfordroboticsclub/StanfordQuadruped)
- [Moco ML](https://github.com/search?q=moco+ml+quadruped)
- [QuadRupedDog](https://github.com/TomOstt/QuadRupedDog)

### 7.4 推荐下一步行动

- [ ] 购买 1 块 ESP32-DevKitC + 1 个 SG90 舵机，跑通 3.6 节的 Demo
- [ ] 购买 PCA9685 + 12 个 MG90S，搭建单腿 3-DOF 平台
- [ ] 下载并编译一个 OpenQuadruped / Moco ML 固件，在面包板上复现
- [ ] 打印一套 SpotMicro 结构件，把舵机装配上去
- [ ] 接入 MPU6050，实现机身倾斜补偿
- [ ] 通过手机 BLE 或 WebSocket 实现遥控
- [ ] 接入 xiaozhi-esp32-server 或云端 LLM，实现语音交互

---

## 附：成功标准核对

- [x] 文档覆盖 A/B/C/D/E 五个模块
- [x] 明确使用并标注 4 个 Skill 的输入来源
- [x] 包含公司、产品、技术、机器人落地的完整链路
- [x] 至少包含 1 张芯片选型表、1 张硬件清单表、1 个舵机驱动代码示例、1 条从 0 到 1 的路线图
- [x] 文档已保存到 `inbox/espressif-esp32-quadruped-research.md`
- [x] 不提交 commit

---

*本文档由 Claude Code 调用 4 个已安装 Skill 协同生成，仅作为项目草稿保存于 `inbox/`，尚未经人工验证与评审。*
