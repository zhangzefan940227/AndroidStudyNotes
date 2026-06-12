# 目录简介
frameworks 目录是 Android 系统的**核心框架层**，位于 Linux 内核层和应用层之间，扮演着承上启下的关键角色。它实现了 Android 系统的基础服务和应用层 API，是连接底层硬件与上层应用的桥梁。

在 Android 14 中，frameworks 目录的核心价值体现在：

* **API 提供**：定义并实现了 Android 应用开发所需的所有核心 API，包括四大组件（Activity、Service、ContentProvider、BroadcastReceiver）的基础框架
* **系统服务管理**：负责启动、管理和协调系统级服务，如 ActivityManagerService、WindowManagerService、PackageManagerService 等
* **跨进程通信**：实现了 Binder IPC 机制，支持进程间高效通信
* **资源管理**：提供统一的资源管理机制，支持多语言、多分辨率适配
* **图形渲染**：负责 UI 渲染、动画效果、SurfaceFlinger 合成等图形系统功能

frameworks 目录主要由 Java 和 C++ 编写，分为 Java 框架层和 Native 框架层两个部分，体现了 Android 系统的分层设计理念。

# 总览

```java
├── av  //多媒体架构。包含 mediaserver、AudioFlinger（音频流处理）、Stagefright（播放引擎源码）等。
├── base //包含了 Android 系统最关键的代码。
├── compile  //包含渲染脚本（RenderScript）的编译器和一些 Java 相关的编译工具。
├── data-binding  //DataBinding 编译插件和运行库的源码，用于实现 XML 布局与逻辑代码的绑定。
├── ex  //扩展库（Extensions），包含一些可选的、不属于 Android 核心但常用的库，如特殊的相机特效处理、通用设置辅助库等。
├── hardware  //Java 层对硬件抽象层（HAL）的接口定义。例如相机 (Camera)、显示显示控制等。
├── layoutlib  //Android Studio 预览窗口的核心实现。它允许在没有真实手机环境的情况下，在電腦上模拟渲染 XML 布局。
├── libs  //提供给系统使用的通用 C++/Java 库。
├── minikin  //Android 的核心字体排版引擎。负责文本的测量（Measure）、断句和字符排版。
├── multidex  //支持旧版 Android 分包（突破 64k 方法限制）的兼容库实现。
├── native  //包含 C++ 实现的基础功能。
├── opt  //可选组件。包含像 telephony（电话框架）、net/wifi（Wi-Fi 逻辑）、calendar 等独立的功能模块。
├── proto_logging  //基于 Protocol Buffers 的日志记录系统，主要用于统计系统度量数据（Statsd）。
├── rs  //RenderScript 的运行时环境。虽然 Android 12 后已不推荐使用，但在 Android 16 源码中仍保留用于向后兼容和高性能图像处理计算。
└── wilhelm  //Android 对 OpenSL ES 和 OpenMAX AL 标准的实现。主要用于底层原生音频和多媒体 API 的支持。
```

# 1. av：音频视频框架

frameworks/av 目录包含了 Android 的音频视频框架，负责音频/视频的采集、解码、处理、输出以及流媒体协议等相关功能的实现。

## 1.1 av/media/libstagefright：核心播放引擎
Android 最核心的媒体框架，负责多媒体文件的解析、解码和渲染流程。

- ​`NuPlayer`: Android 现行的主力播放器实现类。负责调度数据源、解码器和渲染器。
- ​`MediaCodec`: 开发者最常接触的类。负责调用底层硬件或软件解码器进行编解码。
- ​`GenericSource`: 负责多媒体数据的读取和解封装（Extractor）。
- ​`MediaExtractor`: 负责解析不同的文件格式（如 MP4, MKV, TS）。

## 1.2 av/media/libmediaplayerservice：播放器工厂服务

作为 Binder 服务，管理系统中所有播放器实例的生命周期。

- `MediaPlayerService`: 管理 MediaPlayer、Recorder 实例。
- `StagefrightRecorder`: 负责音视频录制的逻辑控制。

## 1.3 av/media/libmediaformatshaper：格式修整器 - Android 12+ 增强
用于在编码前根据设备能力动态调整媒体格式参数，确保导出的视频符合硬件标准。

## 1.4 av/media/codec2：下一代编解码架构
旨在取代老的 ACodec 架构，提供更模块化、更易于硬件厂商适配的编解码组件。

## 1.5 av/services/audiopolicy：音频策略服务
负责音频路由决策，比如：插上耳机时声音从哪出，音量控制逻辑等。

- `AudioPolicyService`: 系统服务，接收来自 Java 层的音频控制请求。
- `AudioPolicyManager`: 核心逻辑类。负责执行策略（例如：通话时屏蔽媒体音，判断当前最适合的音频输出设备）。

## 1.6 av/services/audioflinger：音频混音与输出
负责将多个应用发送的音频数据进行混音，并写入硬件驱动。

- `AudioFlinger`: 音频框架的核心守护进程。
- `AudioFlinger::PlaybackThread`: 混音线程。负责处理重采样（Resampling）和混音
- ​`AudioTrack` (Native层): 提供给上层写数据的接口。

## 1.7 av/services/camera/libcameraservice：相机服务
连接应用层 Camera API 和底层 Camera HAL 的中转站。

- `CameraService`: 管理相机权限、多应用竞争关系。
- `CameraDeviceClient`: 代表一个打开的相机设备，处理拍照、预览请求。

  

**核心流程总结：

- 播放： `NuPlayer` -> `MediaCodec` -> `AudioFlinger` / `SurfaceFlinger`。
- 录制： `CameraService` -> `StagefrightRecorder` -> `MediaCodec` -> `MPEG4Writer`。
- 策略： `AudioPolicyManager` 决定声音从哪个扬声器出。

# 2 base：Android系统的核心

```
===================================================================================================
frameworks/base/ 目录树总览与模块协同图
=======================================
【 1. 编译配置与 API 边界层 】 ---> 负责编译规则、API 暴露管控和新特性开关
├── config/             <-- 系统编译配置文件、权限白名单（如 preloaded-classes）
├── api/                <-- 存放 API 签名文本（current.txt），由 Metalava 进行兼容性校验
├── android-sdk-flags/  <-- 管理 AConfig 功能旗帜（Flags），控制新特性的动态条件式开关
└── metadata/           <-- 收集并生成相机、传感器等硬件能力的元数据描述文件
=================================================================================================
【 2. 核心架构与运行骨架（纵向分层） 】 ---> 构建系统启动、进程隔离与跨进程通信（IPC）
├── boot/               <-- 引导类路径（Bootclasspath）配置，定义最先加载的核心 Java 类
├── core/               <-- 客户端框架层（运行在各 App 进程），包含四大组件 API、View 视图树、资源管理
├── services/           <-- 服务端常驻层（system_server 进程），包含 AMS、WMS、PMS、电源管理等
├── native/             <-- 窗口、输入分发、传感器等核心服务在 C++ 层的实现及 Binder 跨进程架构
├── libs/               <-- 框架层依赖的 C++ 共享库与 JNI 桥接层（如 android_runtime、hwui 图形库）
└── cmds/               <-- 命令行工具（如 am、wm、pm 的代理入口）与应用进程初始化（app_process）
=================================================================================================
【 3. 核心子系统与硬件特性封装（横向分类） 】 ---> 向上层提供标准的 Java 硬件抽象 API
├─ [ 图形与多媒体 ] ──────────────────────────────────────────────────────────────────────────────
│  ├── graphics/         随系统发布、与框架层高度耦合的独立生态组件
├── packages/           <-- 内置关键组件：SystemUI（状态栏/锁屏）、SettingsProvider（全局配置数据库）等
├── apex/               <-- 定义哪些系统组件会被打包成 APEX（Mainline 模块化更新组件）
├── data/               <-- 存放系统级静态资源（默认字体 Fonts、全局系统铃声与音效）
├── mime/               <-- 维护全局文件扩展名与互联网媒体类型（MIME Type）的映射表
├── proto/              <-- 存放 Protobuf 定义文件，用于电池、窗口状态在跨进程/存储时的轻量化序列化
├── sax/                <-- 高效事件驱动 XML 解析器的扩展封装（系统内部配置解析）
└── vendor/             <-- 预留给 OEM 手机厂商进行接口自定义扩展与 HAL 对接的存根
=================================================================================================
【 5. 质量保障、测试与工程效能矩阵 】 ---> 提供自动化编译期检查与双轨制测试环境
│
├─ [ 宿主机超高速测试 (Android 16 核心演进) ] ────────────────────────────────────────────────────
│  ├── ravenwood/       <-- 允许框架层单元测试脱离真机/模拟器，直接在 PC 端 JVM 上秒级运行
│  └── errorprone/      <-- 集成 Google 静态分析规则，在编译期拦截内存泄漏、并发等代码缺陷
│
├─ [ 设备端自动化测试 (Target-driven) ] ─────────────────────────────────────────────────────────
│  ├── apct-tests/      <-- 自动化性能平台（APCT）的常驻基准测试用例（压力与耗时测试）
│  ├── startop/         <-- 测量和优化 App/系统服务启动耗时、ART 预热、类加载加速的工具集
│  ├── tests/           <-- 全面覆盖 Graphics、UI、核心服务的功能测试用例集
│  └── [ 测试基础设施桩 ]: test-base / test-runner / test-mock / test-junit / test-legacy
│
└─ [ 开发者辅助资源 ] ────────────────────────────────────────────────────────────────────────────
├── tools/           <-- 编译、资源打包、代码生成时使用的各种系统内部脚本与辅助工具
├── docs/            <-- 用于生成官方标准 Javadoc SDK 说明文档的模板与配置
└── samples/         <-- 演示如何使用系统新引入 API 的官方标准示例工程
==================================================================================================
```

**核心架构：**

- `core`/`services`​：这是 Android 最经典的 **客户端/服务端** 架构。`core` 中的类（如 `Activity.java`）运行在各个 App 进程中，当应用执行复杂操作时，`core` 会通过 `native` 层的 Binder 驱动跨进程呼叫 `services`（如 `ActivityTaskManagerService`）执行具体逻辑。
- **`boot`​**：负责配置系统的 ​**Bootclasspath​**。在 zygote 进程启动时，这里的类会被最先加载和初始化。

**横向硬件子系统：**

- **`graphics` / `opengl`​**：所有 UI 界面绘制的基石。`graphics` 暴露了 Canvas 和 RenderNode 等高级 Java 接口，底层通过 `libs/hwui` 连接硬件渲染引擎。
- `wifi` / `telephony` / `telecomm`​：三大网络与通话支柱。`telephony` 处理底层基带和数据链路，`telecomm` 负责管理通话路由与音频焦点。

**安全与身份（现代 Android 核心）**

- **`identity` / `keystore`**：近几个版本（包括 Android 16）极力强化的安全目录。**`identity` 用于支持银行级硬件加密的数字身份证件，`keystore` 用于保护应用秘钥不被读取。

## 2.1 核心架构与策略管理（Flag & API）
- **`android-sdk-flags`**: **【Android 16 新特性重点】** 存放基于 `Aconfig` 机制的 SDK 级别特性开关（Flags）。控制哪些新 API 或功能在当前编译版本中对 App 可见或生效。
- **`api`**: 存放 Android 官方的 **API 签名签名档（Current.txt 等）**。用于 `make update-api` 检查和确保系统修改没有破坏向前兼容性。
- **`config`**: 系统的编译期基础配置。包括权限白名单、默认类加载器配置以及系统属性（properties）定义。

## 2.2 核心运行库与核心服务（Core & Services）

- **`core`**: **核心中的核心**。包含 Android Java 层的核心基础类（如 `android.app`, `android.content`, `android.view`, `android.os`），以及系统的资源管理（res）、窗口、输入法等基础交互组件。
- **`services`**: **系统服务控制中心**。包含 `ActivityManagerService (AMS)`、`WindowManagerService (WMS)`、`PackageManagerService (PMS)` 等运行在 `system_server` 进程中的所有核心系统服务的具体实现 Java 源码。
- **`boot`**: 包含系统启动阶段（Boot）必需的公共库声明及初始化配置，与 Zygote 启动和基础类预加载（Preloaded Classes）紧密相关。
- **`native`**: 存放框架层依赖的 C/C++ 本地代码。例如核心 Graphics（图形）渲染底层接口、Binder 的本地 C++ 实现以及 JNI 绑定的中间层。
- **`libs`**: 框架层内部或提供给系统应用的各类 Java/C++ **静态公共库**与工具库（例如 `hwui` 渲染库的部分底层逻辑）。
- **`proto`**: 存放 Protocol Buffers (`.proto`) 协议文件。用于系统服务的序列化数据传输、日志收集（如 IncidentReport、Statsd 统计数据结构）。

## 2.3 图形与硬件抽象（Graphics & Media）

- **`graphics`**: **图形图像处理框架**。包含 Canvas（画布）、Paint（画笔）、Bitmap（位图）、Font（字体）等 Java 层的图形 API，以及底层的硬件加速调用接口。
- **`opengl`**: Android 系统的 **OpenGL ES 渲染框架**。管理 3D 图形渲染管线、EGL 上下文环境以及与 SurfaceFlinger 的对接。
- **`media`**: **多媒体框架**。包含 MediaPlayer、MediaCodec、AudioTrack 等音视频播放/录制/编解码的控制流框架（部分上层逻辑已随 Module 移出，此处保留核心骨架）。
- **`drm`**: 统一的 **数字版权管理（DRM）框架**。为受版权保护的媒体内容提供解密和播放的安全隔离通道。
 
## 2.4 通信与连接性（Connectivity）

- **`telephony`**: **蜂窝移动通信框架**。包含 SIM 卡管理、短信（SMS/MMS）收发逻辑、数据上网、电话呼叫底层数据链路（TelephonyManager）的核心实现。
- **`telecomm`**: **高级通话管理框架（Telecom Service）**。负责处理系统层面的拨号路由、VoIP 通话集成、蓝牙耳机通话挂断以及多路通话控制。
- **`wifi`**: **Wi-Fi 核心服务框架**。管理无线网络连接、热点状态（SoftAP）、Wi-Fi 扫描及与底层 `wpa_supplicant` 通信的控制逻辑。
- **`location`**: **位置服务框架（Location Manager）**。聚合了 GPS、基站（Cell）、Wi-Fi 等多种定位提供者（Providers）的调度逻辑与地理围栏管理。
- **`omapi`**: **安全元件访问接口（Open Mobile API）**。允许系统或受信任应用直接访问安全芯片（SE, Secure Element）或 SIM 卡中的安全应用（如刷公交卡、模拟加密门禁）。
- **`nfc-non-updatable` / `nfc-extras`**: NFC（近场通信）非更新核心组件与扩展框架。提供 NFC 读写卡、点对点传输及卡模拟的基础 Framework 支撑。

## 2.5 安全、身份与存储（Security & Identity）

- **`keystore`**: **密钥保险箱客户端框架**。提供硬件级加密、密钥生成（KeyStore/Keymaster/KeyMint）的 Java 层 API 和系统安全守护进程交互逻辑。
- **`identity`**: **电子身份凭证框架**。用于支持数字驾照（ISO 18013-5 移动驱动执照 mDL）及电子身份证等符合国际安全标准的身份凭证存储与安全传输。
- **`data`**: 包含系统内置的静态数据资源。例如：默认的系统音效、动态壁纸默认配置、键位映射表（Keymaps）等。

## 2.6 传统协议、特殊数据与遗留组件

- **`mms`**: 传统的彩信（Multimedia Messaging Service）处理公共库，提供彩信 PDU 编解码及特定网络协议支持。
- **`mime`**: 系统的 MimeType（媒体文件类型解析）映射表与维护框架，负责将文件后缀名（如 `.mp3`）正确识别为对应的媒体类型。
- **`obex`**: 蓝牙对象交换协议（Object Exchange）的底层实现，常用于蓝牙传文件、同步联系人（PBAP 协议）等场景。
- **`sax`**: 针对 XML 文件的 SAX（Simple API for XML）解析器扩展框架，属于较早期的 XML 高效流式解析支持。
- **`rs` (RenderScript)**: 曾用于高性能 3D 渲染和计算的框架源码（注：由于已被 Vulkan 替代，此处主要为向后兼容保留的遗留/废弃代码）。

## 2.7 现代模块化与打包（APEX & Apps）

- **`apex`**: **模块化组件配置**。存放可滚动更新的系统模块（APEX 模块）的打包定义、权限清单和框架层拦截逻辑。
- **`packages`**: 存放**系统内置的核心应用与界面组件**。例如 `SystemUI`（状态栏/导航栏）、`SettingsProvider`（设置数据库）、屏幕录制、输入法以及各类默认的 System Apps 源码。
- **`metadata`**: 存放整个 Framework 仓的元数据配置，包括版权许可信息（METADATA 文件）及模块所有者（OWNERS）定义。
- **`vendor`**: 厂商定制扩展预留目录。允许 SOC 厂商（高通、联发科等）或设备厂商在不破坏 AOSP 原生结构的前提下，在此添加专有的框架扩展。

## 2.8 自动化工具、效率与诊断

- **`cmds`**: **系统核心命令行工具**。包含了通过 adb 常用的 shell 工具底层源码，例如 `am` (Activity Manager)、`pm` (Package Manager)、`wm` (Window Manager)、`bmgr` (备份管理器) 等。
- **`tools`**: 编译、解析和优化 Framework 的各种主机端脚本工具。包含 AOSP 编译链中用于代码静态生成的辅助工具。
- **`errorprone`**: 静态代码分析配置目录。引入了 Google 的 Error Prone 插件，在编译 Framework 源码时进行严格的 Java 语法与潜在 Bug 静态安全检查。
- **`startop`**: 专门用于**系统和应用启动速度优化**（Start Optimization）的实验性工具和跟踪辅助代码。
- **`docs`**: 存放用于生成底层技术文档和官方 API 开发者文档的数据及模板配置。

## 2.9 测试与环境模拟（Tests & Ravenwood）

- **`ravenwood`**: **【Android 16 架构演进焦点】** 专门用于在**宿主机（PC/服务器 Linux JVM）**上无缝运行 Android 平台代码的高性能、轻量级单元测试环境。无需烧录固件或启动模拟器，即可直接在本地对 Framework 的 Service 进行极速测试。
- **`apct-tests`**: 自动化性能测试控制（Automated Platform Performance Testing）相关的测试用例和性能基准测试代码（PerfTests）。
- **`tests`**: 庞大的 **Framework 本地集成测试与单元测试集**。包含了对核心系统服务（如 AMS/WMS）功能的覆盖性测试代码。
- **【测试基类家族】 (`test-*`)**:
    - **`test-base`**: 包含 Android 原生测试环境最基础的类（如 `AndroidTestCase`）。
    - **`test-junit`**: 系统框架层与 JUnit 测试框架集成的核心桥梁代码。
    - **`test-runner`**: 负责在设备端解析、调度并执行测试用例的运行器（Test Runner）。
    - **`test-mock`**: 提供系统核心类（如 Context, PackageManager）的官方 **Mock（模拟）对象**，专门用于在测试时隔离真实系统环境。
    - **`test-legacy`**: 为旧版（早于 Android 29）测试用例提供向前兼容的结构和遗留测试类支持。




