# 智能家居安防报警系统GuardSystem

> [!TIP]
> 
> 本项目包含以下3个子项目
> 
> - 设备端Openharmony南向项目(https://github.com/LiHuaJZOAQ/GuardSys)
>
> - 设备端Openharmony北向项目(https://github.com/LiHuaJZOAQ/GuardSysAPP)
>
> - 服务端Web项目(https://github.com/LiHuaJZOAQ/GuardSysServer)

## 项目概述

本系统实现了一套完整的**智能家居安防报警解决方案**，包含三个子系统：**鸿蒙南向驱动层**（OpenHarmony 4.0, Union-Pi Tiger A311D）、**鸿蒙北向APP层**（ArkTS, API 10）、**云服务端**（Node.js + Express + Vue 3）。系统通过多传感器联动（红外、烟雾、温湿度），实现陌生人闯入报警、环境危险报警、温湿度监测等功能，并通过蜂鸣器和 RGB LED 灯执行声光报警。用户可通过浏览器访问 Web 端，实时查看环境数据和报警信息，支持操作与报警日志追溯查询。

---

## 项目背景与简介

随着物联网技术的快速发展，智能家居安防系统逐渐成为家庭安全的重要组成部分。传统安防系统存在设备孤立、报警方式单一、无法远程监控等痛点。本项目基于 OpenHarmony 4.0 操作系统，以 Union-Pi Tiger 开发板为核心，集成多种传感器与执行器，构建了一套从底层驱动到云平台再到 Web 前端的完整安防报警解决方案。系统支持多传感器联动检测、声光报警、远程监控、微信推送、历史追溯等功能，具备高可靠性、可扩展性和实用性。

---

## 系统总体架构

```
┌──────────────────────────────────────────────────────────────┐
│                      南向（OpenHarmony 4.0）                   │
│              Union-Pi Tiger 开发板 (A311D)                    │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  HC-SR501 红外传感器 (GPIO)   → 人体探测 / 入侵检测    │  │
│  │  MQ-2 烟雾传感器 (ADC)        → 烟雾浓度检测           │  │
│  │  SHT30 温湿度传感器 (I2C)     → 环境温湿度采集          │  │
│  │  蜂鸣器 + RGB LED (GPIO)      → 声光报警输出            │  │
│  │  摄像头                       → 人脸识别(OpenCV +      │  │
│  │                                    SeetaFace2)          │  │
│  │  ┌─────────────────────────────────────────────────┐   │  │
│  │  │     NAPI 封装层 (libguardsys_napi.z.so)         │   │  │
│  │  │     ArkTS 侧通过 @ohos.guardsys 模块调用         │   │  │
│  │  └─────────────────────────────────────────────────┘   │  │
│  └────────────────────────────────────────────────────────┘  │
└──────────────────────────┬───────────────────────────────────┘
                           │ NAPI 接口调用
┌──────────────────────────▼───────────────────────────────────┐
│                      北向（OpenHarmony APP）                   │
│                     ArkTS + API 10                            │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  Index.ets          → 仪表板 + 传感器卡片 + 自动上报    │  │
│  │  SensorManager.ets  → NAPI 传感器封装(逐个采集+默认值)  │  │
│  │  TCPClient.ets      → WebSocket 客户端(注册+心跳+重连)  │  │
│  │  WifiManager.ets    → WiFi 扫描/连接(API 10 适配)      │  │
│  │  SettingsPage.ets   → 设置页(服务器/WiFi/报警/刷新)    │  │
│  │  LogPage.ets        → 日志查看页(三级分色)             │  │
│  └────────────────────────────────────────────────────────┘  │
└──────────────────────────┬───────────────────────────────────┘
                           │ WebSocket / TCP
┌──────────────────────────▼───────────────────────────────────┐
│                       GuardSys 服务端                         │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  后端 (server/)                                        │  │
│  │  Express + Socket.io + ws + better-sqlite3             │  │
│  │  端口 3000：HTTP + WebSocket(/device) + Socket.io      │  │
│  │  端口 3001：TCP 原始协议 (兼容旧设备)                    │  │
│  │  JWT 认证 + SQLite 持久化 + ServerChan 微信推送         │  │
│  ├────────────────────────────────────────────────────────┤  │
│  │  前端 (web/)                                           │  │
│  │  Vue 3 + Vite + Chart.js + Socket.io-client            │  │
│  │  Login.vue       → 登录 / 注册                          │  │
│  │  Dashboard.vue   → 仪表盘 + 实时数据 + 控制 + 历史曲线   │  │
│  │  ActivityLog.vue → 操作与报警日志追溯查询                │  │
│  └────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────┘
```

---

## 硬件选型与传感器

| 硬件模块 | 型号/规格 | 通信协议 | 功能描述 |
|---------|-----------|---------|---------|
| 主控芯片 | A311D (Union-Pi Tiger) | — | 四核 Cortex-A73 + 双核 Cortex-A53 |
| 操作系统 | OpenHarmony 4.0 Release | — | 开源鸿蒙操作系统 |
| 温湿度传感器 | SHT30 | I2C (GPIO 5, 地址 0x44) | 温度 ±0.3°C, 湿度 ±2% RH |
| 烟雾传感器 | MQ-2 | ADC (GPIO 1, 12bit) | 检测液化气/丙烷/氢气浓度 |
| 红外传感器 | HC-SR501 | GPIO (GPIO 9) | 人体红外感应，感应角度 <120° |
| 蜂鸣器 | 有源蜂鸣器 | GPIO (GPIO 384) | 声光报警发声 |
| RGB LED | 共阴 RGB LED | GPIO (R=381, G=382, B=383) | 三色状态指示 |
| 摄像头 | USB 摄像头 | USB | 人脸图像采集（预留） |

### 引脚定义

| 设备 | 协议 | 引脚/地址 |
|------|------|-----------|
| SHT30 温湿度 | I2C | I2C 5, 地址 0x44 |
| MQ-2 烟雾 | ADC | GPIO 1 |
| HC-SR501 红外 | GPIO | GPIO 9 |
| 蜂鸣器 | GPIO | GPIO 384 |
| RGB LED 红灯 | GPIO | GPIO 381 |
| RGB LED 绿灯 | GPIO | GPIO 382 |
| RGB LED 蓝灯 | GPIO | GPIO 383 |

---

## 南向驱动层设计（传感器驱动）

南向驱动基于 OpenHarmony 4.0 Release，使用 C/C++ 语言开发，直接操作 Linux sysfs 与 IIO 子系统接口，实现各传感器的底层数据采集与执行器控制。

### SHT30 温湿度驱动 (`src/sensor_sht30.c`)

- 基于 Linux I2C-RDWR ioctl 接口读写 `/dev/i2c-x` 设备节点
- 采用单次触发高重复精度模式（命令 `0x2C06`），避免连续采集发热
- 实现 CRC-8 校验（多项式 `0x31`），确保数据完整性
- 温湿度换算公式：
  - `温度 = 175.0 × raw_temp / 65535 - 45.0 (°C)`
  - `湿度 = 100.0 × raw_humi / 65535 (%)`

### MQ-2 烟雾驱动 (`src/sensor_mq2.c`)

- 基于 IIO 子系统 sysfs 节点读取 ADC 原始值（`/sys/bus/iio/devices/iio:device0/in_voltage2_raw`）
- 实现 5 点滑动平均滤波，抑制采样噪声
- R0 校准值根据实测 ADC=1200 校准为 0.4312 kΩ
- 浓度换算链路：`ADC → V_out → Rs → R0/Rs → PPM`
- 拟合公式：`ppm = pow(11.5428 × R0/Rs, 0.6549)`
- 支持运行时 R0 校准（洁净空气环境调用 `mq2_calibrate()`）

### HC-SR501 红外驱动 (`src/sensor_hc_sr501.c`)

- 基于 Sysfs GPIO 接口，支持导出/方向配置/电平读取
- 逻辑引脚到系统 GPIO 映射：`sys_gpio = 379 + logic_pin`
- 实现 3 次采样多数表决去抖（间隔 10ms），防止误触发

### 报警执行器驱动 (`src/alarm_control.c`)

- 通过 Sysfs GPIO 控制蜂鸣器和 RGB LED（三色独立控制）
- 三级报警模式映射：
  - `0` 正常：绿灯亮，蜂鸣器关
  - `1` 警告：黄灯亮（R+G），蜂鸣器关
  - `2` 报警：红灯亮，蜂鸣器开

### 人脸识别模块 (`src/face_recognize.cpp`)

- 集成 OpenCV 图像预处理 + SeetaFace2 人脸识别引擎
- 支持人脸注册（`FaceSearchRegister`）、识别（`FaceSearchRecognize`）、检测（`GetFaceRects`）
- 质量评估过滤低质量人脸（`seeta::QualityAssessor`），相似度阈值 0.7
- **注**：因硬件问题目前无法验证，代码仅供参考

---

## NAPI 接口封装层设计

NAPI（Native API）是 OpenHarmony 提供的一种轻量级原生接口封装框架，允许 ArkTS 应用直接调用 C/C++ 编写的底层驱动代码。本系统采用统一的异步架构设计，将所有传感器操作封装为 Promise 异步接口。

### 架构设计

```
ArkTS 应用代码
     │
     │ import guardsys_napi from '@ohos.guardsys'
     ▼
┌─────────────────────────────────────┐
│      NAPI 中枢注册 (guardsys_napi.cpp)  │
│  聚合 5 个模块的 napi_property_descriptor │
│  注册为单一 @ohos.guardsys 模块         │
├─────────────────────────────────────┤
│  SHT30  │  SR501  │  MQ-2  │  Alarm  │  Face  │
│ napi    │  napi   │  napi  │  napi   │  napi  │
├─────────┴─────────┴────────┴─────────┴────────┤
│      AsyncBaseContext + QueueAsyncWork          │
│  (统一异步基类 + 通用异步工作项提交函数)          │
├────────────────────────────────────────────────┤
│        底层 C/C++ 驱动 (src/)                   │
│    I2C / ADC / GPIO sysfs 直接硬件操作           │
└────────────────────────────────────────────────┘
```

### 统一异步基类 (`napi/guardsys_napi.h`)

所有传感器模块的异步上下文继承自 `AsyncBaseContext` 基类，包含：
- `napi_async_work async_work` — NAPI 异步工作项句柄
- `napi_deferred deferred` — Promise 延迟对象
- `int32_t status_code` — 底层执行状态码

`QueueAsyncWork()` 内联函数封装 `napi_create_async_work` 与 `napi_queue_async_work`，实现零开销异步任务提交。

### 模块注册中枢 (`napi/guardsys_napi.cpp`)

采用"聚合-导出"模式：
1. 各子模块（sht30_napi.cpp, mq2_napi.cpp 等）在命名空间 `guardsys::napi` 内定义 `g_xxx_desc[]` 描述符数组
2. 中枢文件通过 `napi_module_register` 注册为单一 `@ohos.guardsys` 模块
3. 使用 `__attribute__((constructor))` 在 SO 库加载时自动注册

### 导出的 NAPI 接口

| 模块 | ArkTS 接口 | 参数 | 返回值 |
|------|-----------|------|--------|
| SHT30 | `getSht30Data(i2cBus, address)` | `number, number` | `{ temp, humi }` |
| MQ-2 | `getMq2Data(adcChannel)` | `number` | `{ smoke }` |
| HC-SR501 | `getIrStatus(gpioPin)` | `number` | `{ ir }` |
| 报警 | `setAlarmStatus(status, pins)` | `number, object` | `void` |
| 人脸 | `faceInit()` / `faceDeinit()` / `getFaceRects()` / `faceRegister()` / `faceRecognize()` | 按需 | 按需 |

### TypeScript 类型声明 (`@ohos.guardsys_napi.d.ts`)

提供了完整的 ArkTS 类型声明文件，支持 IDE 智能提示和编译期类型检查。

---

## 北向APP设计（数据采集与自动上报）

北向应用基于 OpenHarmony API 10，使用 ArkTS 语言开发，部署于 Union-Pi Tiger 开发板。通过 NAPI 调用南向驱动，经 WebSocket 上报至云服务器。

### 核心模块

#### SensorManager (`SensorManager.ets`)

- 封装所有 NAPI 传感器接口调用
- `getAllData()` 逐传感器 `await` + `try-catch` 读取，单传感器失败不阻塞其他传感器
- `initAlarm()` 启动时点亮绿灯，指示系统正常运行
- `setAlarmMode(mode)` 设置报警模式并驱动蜂鸣器 + RGB LED
- 引脚参数硬编码（SHT30=GPIO5/0x44, MQ-2=GPIO1, SR501=GPIO9, 蜂鸣器=384, RGB=381/382/383）

#### Index (`Index.ets`)

仪表板页面，每秒自动刷新传感器数据，包含：
- **状态指示**：WiFi 连接状态 + IP 地址，WebSocket 服务器连接状态
- **传感器卡片**：温度、湿度、烟雾浓度、红外状态，支持点击单传感器刷新
- **报警状态栏**：实时显示当前报警等级 + 报警原因，报警/警告时显示"撤销警报"按钮
- **红外布防切换**：点击红外卡片标签切换布防/撤防，布防后红外触发自动升级为 mode 2 报警
- **自动上报**：每次 `collectAll()` 采集后自动通过 WebSocket 上报，无论用户在哪个页面

#### TCPClient (`TCPClient.ets`)

WebSocket 单例客户端，基于 `@ohos.net.webSocket`：
- 自动构建 URL（443 端口用 `wss://`，其他端口用 `ws://`，路径 `/device`）
- 连接成功后自动发送 `{"type":"register","mac":"序列号"}` 注册设备
- 30 秒心跳保活（`{"type":"ping"}`）
- 断线 5 秒自动重连
- 发送失败主动断开并触发重连
- 接收服务器下发的控制指令（`setAlarm` / `collect`）

#### WifiManager (`WifiManager.ets`)

WiFi 操作封装，适配 OpenHarmony API 10：
- 扫描可用 WiFi 列表，信号强度 RSSI 转 0-100 等级
- 使用 `addCandidateConfig` + `connectToCandidateConfig` 连接
- 通过 `@ohos.deviceInfo.serial` 获取设备序列号作为设备唯一标识
- 获取本机 IP、当前 WiFi 信号强度 RSSI

#### SettingsPage (`SettingsPage.ets`)

设置页面，提供完整配置能力：
- 服务器地址/端口配置及连接/断开控制
- WiFi 扫描列表、连接/断开操作
- 自动上报开关与间隔控制
- 传感器刷新频率调节（默认 1000ms）
- 调试报警模式设置与蜂鸣器控制

#### LogPage + LogStore (`LogPage.ets`, `LogStore.ets`)

- `LogStore` 单例全局共享日志消息，上限 200 条
- 三级分色日志：`error`（红色）、`warn`（黄色）、`info`（黑色）
- 独立日志页面，500ms 自动轮询刷新

### 报警联动检测逻辑

每次数据采集后按优先级检测，仅输出最高模式：

| 优先级 | 等级 | LED | 蜂鸣器 | 触发条件 |
|--------|------|-----|--------|----------|
| **最高** | 报警 (2) | 红色 | 开启 | 烟雾≥200ppm / 布防+红外有人 / 危险区域有人未撤离 |
| **中** | 警告 (1) | 黄色 | 关闭 | 烟雾≥100ppm / 温度≥40°C / 湿度≥80%或≤20% / 烟雾突增≥50ppm |
| **低** | 正常 (0) | 绿色 | 关闭 | 数值偏高提示 / 红外有人记录 |

---

## 服务端设计（数据存储与实时推送）

服务端基于 Node.js，使用 Express + Socket.io + ws + better-sqlite3，单文件入口 `server/index.js`。

### 服务架构

```
南向设备(开发板)  ──WebSocket/TCP──▶  GuardSys 服务端
                                        │
                                    ┌───┴───┐
                                    │ 端口  │
                                    │ 3000  │── HTTP + WebSocket(/device) + Socket.io
                                    ├───────┤
                                    │ 端口  │── TCP 原始协议 (JSON + \n 分隔)
                                    │ 3001  │
                                    └───┬───┘
                                        │
                                    ┌───▼───┐
                                    │ SQLite │── guardsys.db (4 张表)
                                    └───────┘
                                        │
                                    Socket.io
                                        │
                                    ┌───▼───┐
                                    │ Web   │── Vue 3 前端
                                    │ 前端   │
                                    └───────┘
```

### 双协议支持

| 协议 | 路径/端口 | 说明 |
|------|-----------|------|
| **WebSocket（推荐）** | `wss://host/device`（443）或 `ws://host:port/device` | 共享 HTTP 端口，无需额外暴露端口 |
| **TCP 原始协议** | 端口 3001 | 兼容旧设备，JSON + `\n` 分隔 |

### 数据库设计（SQLite）

| 表名 | 主要字段 | 说明 |
|------|---------|------|
| `users` | id, username, password, created_at | 用户账号（默认 admin/admin123） |
| `devices` | id, name, connected_at, last_seen, online, armed | 设备信息与在线状态 |
| `sensor_logs` | id, device_id, temp, humi, smoke, ir, alarm, rssi, alarm_reason, created_at | 传感器历史数据（保留 7 天） |
| `push_logs` | id, device_id, push_type, pushed_at | 推送日志（防重复） |

### 认证体系

- JWT（JSON Web Token）认证，有效期 24 小时
- 所有 REST API 统一前缀 `/api`
- 认证接口除外均需 `Authorization: Bearer <token>`
- Socket.io 连接需传入 `auth: { token }`

### 报警推送

- 通过 ServerChan 服务推送微信通知（需配置 `SCKEY` 环境变量）
- 触发条件：烟雾≥200ppm 或 温度≥40°C 或 湿度≥80%/≤20% 或 报警等级≥2
- 每台设备每天仅推送一次，防止重复通知

### 自动运维

- 定时任务（每小时）：将超 5 分钟无数据的设备标记离线，清理 7 天前的历史数据
- 设备注册时将 MAC/序列号作为永久 `device_id` 持久化到数据库

---

## Web前端展示（仪表盘与控制面板）

前端基于 Vue 3 + Vite + Chart.js + Socket.io-client，部署于云服务器。

### Login (`Login.vue`)

- 登录/注册双标签页，支持 JWT 自动验证
- 已登录自动跳转仪表盘

### Dashboard (`Dashboard.vue`)

- **实时数据仪表盘**：温度、湿度、烟雾（ppm）、红外状态、WiFi 信号强度、报警状态
- **颜色分级**：正常=绿色，警告=黄色，报警=红色（随阈值变化）
- **设备列表**：展示所有设备 ID、在线/离线状态、最后在线时间，支持单选切换
- **控制面板**：选择设备后可设置报警模式（正常/警告/报警）、触发立即采集
- **撤销报警**：报警/警告状态时显示撤销按钮
- **实时推送**：通过 Socket.io 接收 `sensor:data` 事件，自动更新数据与图表
- **24 小时历史曲线**：使用 Chart.js 绘制温度、湿度、烟雾三条曲线，双 Y 轴
- **连接状态指示**：顶部实时显示 Socket.io 连接状态
- 可配置轮询刷新频率（默认 5000ms）

### ActivityLog (`ActivityLog.vue`)

- **操作与报警日志追溯查询**
- 支持按设备 ID 过滤
- 表格展示时间、设备、温度、湿度、烟雾、红外、报警状态、报警原因
- 按报警等级着色：报警=红色行、警告=黄色行、正常=绿色徽章
- 支持手动刷新

---

## 通信协议与报警联动逻辑

### 连接方式

| 协议 | 路径/端口 | 说明 |
|------|-----------|------|
| **WebSocket（推荐）** | `wss://host/device`（443）或 `ws://host:port/device` | 共享 HTTP 端口 |
| **TCP 原始协议** | 端口 3001 | 兼容旧设备，JSON + `\n` 分隔 |

### 设备注册

```json
{"type":"register","mac":"1234567890"}
```

注册后 MAC 作为永久 `device_id` 持久化到数据库。若未注册则回退使用 `IP:端口` 作为临时 ID。

### 数据上报

```json
{"type":"report","temp":"25.3","humi":"60.1","smoke":12.34,"ir":true,"alarm":0,"rssi":-34}
```

| 字段 | 类型 | 说明 |
|------|------|------|
| `temp` | string | 温度 |
| `humi` | string | 湿度 |
| `smoke` | number | 烟雾浓度（ppm） |
| `ir` | bool | 红外有人 |
| `alarm` | number | 0=正常, 1=警告, 2=报警 |
| `rssi` | number | WiFi 信号强度（可选） |

服务器应答：`{"success":true}`

### 红外布防/撤防

```json
{"type":"arm","value":1}
```

- `value=1` 布防：红外有人自动触发设备端报警
- `value=0` 撤防：红外仅记录日志

### 控制指令（服务器 → 设备）

```json
{"type":"cmd","action":"setAlarm","value":1}
```

| action | value | 说明 |
|--------|-------|------|
| `setAlarm` | 0/1/2 | 设置报警模式 |
| `collect` | — | 立即采集传感器数据 |
| `arm` | 0/1 | 远程布防/撤防 |

### 心跳保活

```json
{"type":"ping"}  →  服务器回复 {"type":"pong"}
```

设备每 30s 发送；超 5 分钟无数据则标记离线。

### Socket.io 事件（Web 前端实时推送）

| 事件 | payload | 说明 |
|------|---------|------|
| `sensor:data` | `{deviceId, temp, humi, smoke, ir, alarm, alarmReason, timestamp}` | 传感器实时数据 |
| `device:online` | `{deviceId, online}` | 设备上线/离线 |
| `device:command` | `{deviceId, action, value}` | 控制指令已下发 |
| `device:arm` | `{deviceId, armed}` | 布防/撤防状态变化 |

### 报警联动逻辑

| 优先级 | 等级 | LED | 蜂鸣器 | 触发条件 | 报警原因 |
|--------|------|-----|--------|----------|---------|
| **最高** | 报警 (2) | 红色 | 开启 | 烟雾≥200ppm | 烟雾过高 |
| | | | | 布防+红外有人 | 入侵报警 |
| | | | | 红外有人+烟雾≥100或温度≥40 | 人员未撤离 |
| | | | | 湿度≥80% | 湿度过高 |
| | | | | 湿度≤20% | 湿度过低 |
| **中** | 警告 (1) | 黄色 | 关闭 | 烟雾≥100ppm | 烟雾偏高 |
| | | | | 温度≥40°C | 温度过高 |
| | | | | 湿度≥80%或≤20% | 湿度过高/过低 |
| | | | | 烟雾单次突增≥50ppm | 烟雾突增 |
| **低** | 正常 (0) | 绿色 | 关闭 | 烟雾≥50或温度≥35 | 数值偏高 |
| | | | | 湿度≥70或≤30 | 湿度异常 |
| | | | | 红外有人（撤防） | 有人活动 |

---

## 系统功能演示（实物/截图）

> 照片存放于 `photos/` 目录

### Web 端仪表盘
- **实时传感器数据卡片**：温度、湿度、烟雾浓度（ppm）、红外状态、WiFi 信号、报警等级一目了然
- **设备列表与控制面板**：支持多设备选择、报警模式设置、远程数据采集
- **24 小时历史曲线**：Chart.js 动态渲染的温度、湿度、烟雾浓度趋势图

### 活动日志追溯
- 完整记录每一条传感器数据与报警事件
- 支持按设备 ID 过滤、报警等级着色
- 字段包含：时间、设备、温度、湿度、烟雾、红外、报警状态、报警原因

### 实物运行
- 开发板连接 SHT30、MQ-2、HC-SR501、蜂鸣器、RGB LED 等模块
- 系统启动自动亮绿灯，WebSocket 连接云服务器
- 烟雾超标或红外入侵时红灯+蜂鸣器报警，Web 端同步推送

---

## REST API 文档

所有 API 前缀 `/api`，需 `Authorization: Bearer <token>`（认证接口除外）。

| 方法 | 路径 | 说明 |
|------|------|------|
| POST | `/api/auth/register` | 注册账号 |
| POST | `/api/auth/login` | 登录获取 JWT |
| GET | `/api/auth/verify` | 验证 token |
| GET | `/api/devices` | 设备列表 |
| GET | `/api/devices/:id/latest` | 设备最新数据 |
| GET | `/api/devices/:id/history?hours=24` | 设备历史数据 |
| POST | `/api/devices/:id/control` | 设备控制指令 |
| GET | `/api/logs?limit=50&deviceId=` | 传感器日志查询 |

---

## 开发环境

| 子系统 | 环境要求 |
|--------|----------|
| **南向** | Ubuntu 20.04, OpenHarmony 4.0 Release 源码, Ninja |
| **北向** | DevEco Studio 4.0+, OpenHarmony SDK API 10 |
| **服务端后端** | Node.js 18+, npm |
| **服务端前端** | Node.js 18+, npm (Vue 3 + Vite) |
| **硬件** | Union-Pi Tiger 开发板 (A311D), 各传感器模块 |

---

## 部署指南

### 服务端部署（Railway）

```bash
cd GuardSysServer/server
npm install
npm run dev               # 本地开发
```

Railway 一键部署：导入仓库后设置环境变量 `SCKEY`（可选）。`postinstall` 脚本自动构建 `web/dist`。

### 前端部署

- **方案一**：Railway 自动托管（`postinstall` 脚本自动构建 web/dist）
- **方案二**：独立部署 Vercel（根目录 `web/`，构建命令 `npm run build`）

### 北向 APP 配置

修改 `SettingsPage.ets` 中的服务器地址：
```typescript
@State serverHost: string = 'your-domain'   // 服务器域名
@State serverPort: string = '443'           // 443=wss, 其他=ws
```
通过 DevEco Studio 编译安装 HAP 至开发板。

### 南向驱动推送

```bash
./build.sh --product-name unionpi_tiger --ccache --build-target guardsys_napi
hdc shell mount -o remount,rw /
hdc file send libguardsys_napi.z.so /system/lib/module
hdc shell reboot
```

---

## 项目亮点与创新点

1. **全栈物联网架构**：从南向硬件驱动 → NAPI 封装 → 北向 APP → 云服务端 → Web 前端，形成完整闭环
2. **统一异步 NAPI 设计**：基于 `AsyncBaseContext` 基类 + `QueueAsyncWork` 辅助函数，5 个硬件模块共享统一异步架构，新增模块仅需追加一行描述符
3. **双协议兼容**：同时支持 WebSocket（推荐）和 TCP 原始协议，兼容新旧设备
4. **多传感器智能联动**：四级报警检测机制（烟雾/温度/湿度/红外），支持布防/撤防模式
5. **三级声光报警**：通过 RGB LED（绿/黄/红）和蜂鸣器实现直观的本地报警反馈
6. **人脸识别能力**：集成 OpenCV + SeetaFace2，预留 AI 人脸识别接口（代码完整，待硬件验证）
7. **实时推送与追溯**：Socket.io 实时推送 + ServerChan 微信推送 + SQLite 持久化存储 + 7 天历史数据追溯
8. **API 10 适配**：北向 APP 适配 OpenHarmony API 10 的 WiFi 与 WebSocket API 变更
9. **故障容错**：单传感器故障使用默认值继续运行，不影响整体采集周期
10. **自动重连与心跳**：WebSocket 30 秒心跳 + 5 秒自动重连 + 20 秒超时看门狗

---

## 总结与展望

本项目基于 OpenHarmony 4.0 平台，构建了一套完整的智能家居安防报警系统，实现了从南向传感器驱动到云服务端再到 Web 前端的全链路打通。系统通过红外、烟雾、温湿度多传感器联动，实现了陌生人入侵检测、环境危险报警等功能，支持蜂鸣器和 RGB LED 现场声光报警，同时通过 WebSocket 实时上报数据至云服务器，用户可通过浏览器远程查看环境数据与报警信息，支持历史日志追溯。

**未来展望**：

1. **AI 增强**：完成人脸识别模块的硬件验证，实现陌生人识别与白名单管理
2. **多设备协同**：支持同一家庭多台开发板组网，实现全屋安防覆盖
3. **智能场景联动**：如烟雾报警自动切断燃气阀门、温度过高自动开启空调等
4. **移动端 APP**：开发独立的鸿蒙手机 APP，替代 Web 浏览器访问
5. **数据分析与预测**：基于历史数据训练模型，实现设备故障预测与环境趋势分析
6. **语音报警**：集成 TTS 语音播报，实现语音报警提示
7. **第三方集成**：对接主流智能家居平台（如 HomeKit、HomeAssistant）

---

## 项目结构总览

```
《综合实训》开发项目/
├── GuardSys/                          # 南向源码（OpenHarmony 4.0）
│   └── sample/guardsys/
│       ├── include/                   # 头文件
│       ├── src/                       # 驱动实现
│       ├── napi/                      # NAPI 封装
│       ├── BUILD.gn                   # 编译配置
│       └── bundle.json                # 模块配置
├── GuardSysAPP/                       # 北向 APP (HarmonyOS)
│   └── entry/src/main/ets/pages/
│       ├── Index.ets                  # 仪表板
│       ├── SensorManager.ets          # 传感器封装
│       ├── TCPClient.ets              # WebSocket 客户端
│       ├── WifiManager.ets            # WiFi 管理
│       ├── SettingsPage.ets           # 设置页
│       └── LogPage.ets                # 日志页
├── GuardSysServer/                    # 服务端
│   ├── server/                        # 后端
│   │   └── index.js                   # Express 单文件入口
│   ├── web/                           # 前端
│   │   └── src/
│   │       ├── components/
│   │       │   ├── Login.vue          # 登录/注册
│   │       │   └── Dashboard.vue      # 仪表盘
│   │       └── views/
│   │           └── ActivityLog.vue    # 日志追溯
│   └── docs/                          # 文档
└── README.md                          # 本文件
```

---

## 许可证

GNU Affero General Public License v3.0
