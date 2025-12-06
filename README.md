# 🌊 ReifyFlow

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)
[![Status: Alpha](https://img.shields.io/badge/Status-Alpha-orange.svg)]()
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)]()

> **From Eidos to Matter.**
>
> **从理念到物质。下一代 AI 原生嵌入式开发基础设施。**

ReifyFlow 是一个开源的基础设施项目，旨在打破 **“抽象逻辑 (Idea/Logic)”** 与 **“物理实现 (Reality/Hardware)”** 之间的鸿沟。我们构建了一套 AI 原生 (AI-Native) 的工作流，致力于将自然语言需求自动化转化为高质量的固件代码，实现真正的 **“意图驱动编程” (Intent-Driven Programming)**。

---

## 🚀 Vision (愿景)

在嵌入式开发领域，工程师长期被**碎片化的上下文**所困扰：代码在 IDE 里，硬件配置在 CubeMX 里，寄存器定义在几千页的 PDF 里，而调试日志在串口助手里。

ReifyFlow 通过引入 **AI Agent**、**可视化验证** 和 **双回环测试 (Dual-Loop Testing)**，将这些孤岛连接起来，构建了一个**自我感知、自我纠错、软硬解耦**的自动化开发流水线。

---

## 🏗️ Architecture Overview (架构概览)

ReifyFlow 采用**三层漏斗模型**，确保从需求到落地的每一步都可控、可验证。

### 1. 系统宏观架构 (System Architecture)

```mermaid
graph TD
    %% 样式定义
    classDef user fill:#2c3e50,stroke:#fff,color:#fff;
    classDef brain fill:#e74c3c,stroke:#fff,color:#fff;
    classDef soft fill:#2980b9,stroke:#fff,color:#fff;
    classDef hard fill:#e67e22,stroke:#fff,color:#fff;
    classDef test fill:#27ae60,stroke:#fff,color:#fff;
    classDef data fill:#7f8c8d,stroke:#333,stroke-dasharray: 5 5;

    User((用户)):::user <-->|自然语言交互| Analyst[AI 需求分析师]:::brain

    subgraph "阶段一：需求标准化"
        Analyst -->|生成| Task_Spec(标准任务规格书 JSON):::data
    end

    subgraph "阶段二：软件架构域 (Software Domain)"
        direction TB
        Task_Spec --> Arch_Agent[架构师 Agent]:::brain
        
        Arch_Agent -->|1. 定义接口| Interfaces[抽象接口 .h]:::soft
        Arch_Agent -->|2. 设计调度| Scheduler[调度器/状态机]:::soft
        Arch_Agent -->|3. 实现逻辑| Biz_Logic[纯业务代码 .c]:::soft
        
        %% 软件回环
        Interfaces & Biz_Logic --> Mock_Gen[Mock 虚拟对象生成器]
        Mock_Gen --> SIL_Runner[PC 端仿真运行器]:::test
        SIL_Runner -->|逻辑验证结果| Arch_Agent
    end

    subgraph "阶段三：硬件实现域 (Hardware Domain)"
        direction TB
        Task_Spec & Interfaces --> HW_Agent[硬件工程师 Agent]:::brain
        
        HW_Agent -->|4. 查阅手册| Knowledge_Base[(SVD / 数据手册 / RAG库)]
        HW_Agent -->|5. 硬件映射| Pin_Map[引脚/外设映射表]:::data
        
        Pin_Map --> CubeMX_Driver[CubeMX 配置器]:::hard
        CubeMX_Driver --> HAL_Code[HAL 底层驱动]:::hard
        
        HW_Agent --> Adapter_Gen[适配层生成器]:::hard
        Adapter_Gen -->|连接 HAL 与 接口| Driver_Adapter[驱动适配层 .c]:::hard
        
        %% 硬件回环
        HAL_Code & Driver_Adapter --> Builder[交叉编译 & 烧录]:::test
        Builder --> Board[物理硬件板卡]
        Board -->|串口/RTT 日志| Log_Analyzer[回环诊断器]:::test
        Log_Analyzer -->|硬件修正| HW_Agent
    end

    %% 跨域连接
    Biz_Logic -.->|编译链接| Builder
```

---

### 2. 全链路执行流程 (Execution Workflow)

ReifyFlow 强调 **人机交互 (Human-in-the-Loop)**。在关键节点，特别是硬件映射阶段，系统提供可视化界面供用户确认，避免 AI "幻觉" 导致的硬件风险。

```mermaid
sequenceDiagram
    autonumber
    actor User as 用户
    participant AI_Analyst as 需求分析 AI
    participant AI_Arch as 软件架构 AI
    participant SIL as 软件回环测试 (SIL)
    participant AI_HW as 硬件实现 AI
    participant HIL as 硬件回环测试 (HIL)

    Note over User, AI_Analyst: 步骤 1: 需求定义
    User->>AI_Analyst: "做一个温控风扇，超过30度转动"
    AI_Analyst->>AI_Analyst: 拆解任务指标 (采样率, 阈值, 响应时间)
    AI_Analyst-->>User: 确认任务规格书 (Task Spec)

    Note over AI_Arch, SIL: 步骤 2: 软件架构 & 调度
    AI_Arch->>AI_Arch: 生成接口 (I_Temp.h, I_Motor.h)
    AI_Arch->>AI_Arch: 生成调度策略 (状态机 / RTOS任务)
    AI_Arch->>AI_Arch: 生成业务逻辑 (app_ctrl.c)
    AI_Arch->>SIL: 运行 PC 端 Mock 测试 (虚拟温度变化)
    SIL-->>AI_Arch: 逻辑验证通过 (无死锁，逻辑正确)

    Note over AI_HW, HIL: 步骤 3: 硬件映射 & 实现
    AI_HW->>AI_HW: 读取 Task Spec & 接口定义
    AI_HW->>User: **[可视化交互]** 展示硬件连线图 (ADC1->PA0, PWM->PA8)
    User->>AI_HW: 确认或修改引脚
    AI_HW->>AI_HW: 调用 CubeMX 生成 HAL 驱动
    AI_HW->>AI_HW: 生成适配层 (Driver_Adapter.c)
    AI_HW->>HIL: 编译并下载到 STM32

    Note over HIL, User: 步骤 4: 最终验证
    HIL->>HIL: 硬件运行 & 回传日志
    HIL-->>User: 仪表盘显示：温度 31度，风扇转速 50%
```

---

## 📂 Project Structure (标准化工程结构)

ReifyFlow 生成的项目遵循严格的 **分层解耦** 原则，使得业务逻辑可以脱离硬件独立测试和移植。

```text
Project_Root/
├── .oef/                       # [OEF配置区] - 存放核心元数据，AI 的记忆库
│   ├── task_spec.json          # 1. 任务规格书 (AI生成，唯一事实来源)
│   ├── arch_graph.json         # 2. 软件架构图数据
│   └── hw_map.json             # 3. 硬件映射数据 (引脚分配表)
│
├── core/                       # [软件架构层 - 纯逻辑] - 可以在 PC 上直接跑单元测试
│   ├── inc/
│   │   ├── interface_temp.h    # 抽象接口：只定义 Get_Temp()，绝不包含 HAL 库头文件
│   │   └── interface_motor.h
│   ├── src/
│   │   ├── app_main.c          # 调度器/主循环 (FSM, Scheduler)
│   │   └── business_logic.c    # 核心算法 (PID, 滤波, 状态机)
│   └── tests/                  # SIL 测试代码 (PC端运行，Mock 硬件)
│
├── bsp/                        # [硬件实现层 - 适配器] - 连接软硬的“胶水”
│   ├── stm32f103/              # 具体芯片实现目录
│   │   ├── adapter_temp.c      # 实现 interface_temp.h，内部调用 HAL_ADC
│   │   └── adapter_motor.c     # 实现 interface_motor.h，内部调用 HAL_TIM
│
├── drivers/                    # [底层驱动层 - 自动生成] - CubeMX 的领地，AI 只读不改
│   ├── STM32CubeMX/            # CubeMX 工程目录
│   │   ├── Core/Src/main.c     # 硬件初始化代码
│   │   └── Drivers/            # 厂商提供的 HAL/LL 库文件
│   └── linker/                 # 链接脚本
│
├── tools/                      # [工具链] - 本地自动化脚本
│   ├── build.py                # 自动构建脚本 (调用 GCC/Keil)
│   └── monitor.py              # 日志监控与回环诊断脚本
│
└── Makefile                    # 顶层构建规则
```

---

## 🗺️ Roadmap (路线图)

我们正在分阶段实现这个宏大的愿景：

- [ ] **Phase 1: 核心协议与工具链 (Atomic Tools)**
    - 定义 `reify-protocol` (JSON Schemas)。
    - 实现 `reify-svd-parser` (寄存器映射)。
    - 实现 `reify-rag-engine` (PDF 智能检索)。
- [ ] **Phase 2: 胶水层与生成器 (The Glue)**
    - 实现 `reify-cube-driver` (.ioc 文件操作)。
    - 实现 `reify-code-gen` (基于抽象接口的代码生成)。
- [ ] **Phase 3: 交互与可视化 (The Visualizer)**
    - 开发 Web 可视化工作台，实现代码 <-> 手册的实时跳转。
    - 实现“硬件拓扑图”的拖拽式修改。
- [ ] **Phase 4: 闭环诊断 (Self-Healing)**
    - 集成串口/RTT 日志分析。
    - 实现基于错误日志的自动代码修复。

---

## 🤝 Contributing (贡献)

ReifyFlow 目前处于 **Stealth Mode (隐身开发模式)**，我们正在构建核心 MVP。

如果你对 **嵌入式开发自动化**、**AI Agent** 或 **编译器设计** 感兴趣，欢迎关注我们的进展。

---

*Let's build the compiler for reality.*
