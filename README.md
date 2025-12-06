# 🌊 ReifyFlow

<div align="center">
  <img src="https://avatars.githubusercontent.com/u/248118397?s=400&u=0d029fc7994a08e90c07ae40a4fdb80fc5963073&v=4" width="120" alt="ReifyFlow Logo" />
  
  <h3>From Eidos to Matter.</h3>
  <p><b>从理念到物质。下一代 AI 原生嵌入式开发基础设施。</b></p>

  [![License](https://img.shields.io/badge/License-AGPL_v3-blue.svg)](LICENSE)
  [![Status](https://img.shields.io/badge/Status-MVP_Dev-orange.svg)]()
  [![Platform](https://img.shields.io/badge/Platform-STM32-blue.svg)]()
</div>

---

## 📖 项目简介 (Introduction)

**ReifyFlow** 是一个开源的工程化生态系统，致力于打破 **抽象软件逻辑 (Idea)** 与 **物理硬件实现 (Reality)** 之间的鸿沟。

传统的嵌入式开发中，代码编辑器、硬件配置工具（CubeMX）、数据手册（PDF）和调试工具是割裂的孤岛。ReifyFlow 通过 **AI Agent**、**标准化协议** 和 **双回环验证**，构建了一套 **AI Native** 的自动化流水线：

1.  **意图驱动**：用自然语言描述需求，AI 自动拆解任务。
2.  **软硬解耦**：业务逻辑与底层驱动物理隔离，支持在 PC 端进行纯软件仿真。
3.  **交互验证**：在生成代码前，通过可视化拓扑图确认硬件连接，拒绝黑盒操作。
4.  **自愈闭环**：通过硬件回环日志，自动诊断故障并修正底层配置。

---

## 🏗️ 系统架构 (System Architecture)

ReifyFlow 采用 **三段式漏斗模型**，将开发流程划分为三个独立的阶段。下图展示了数据如何在软件域与硬件域之间流转。

```mermaid
graph TD
    %% ================= 样式定义 =================
    classDef user fill:#2c3e50,stroke:#fff,stroke-width:2px,color:#fff;
    classDef brain fill:#e74c3c,stroke:#fff,stroke-width:2px,color:#fff;
    classDef soft fill:#2980b9,stroke:#fff,stroke-width:2px,color:#fff;
    classDef hard fill:#e67e22,stroke:#fff,stroke-width:2px,color:#fff;
    classDef test fill:#27ae60,stroke:#fff,stroke-width:2px,color:#fff;
    classDef data fill:#7f8c8d,stroke:#333,stroke-width:1px,stroke-dasharray: 5 5,color:#fff;

    %% ================= 1. 需求标准化阶段 =================
    User((用户)):::user <-->|自然语言交互| Analyst[AI 需求分析师]:::brain
    Analyst -->|生成| Task_Spec(标准任务规格书 JSON):::data

    %% ================= 2. 软件架构域 (Software Domain) =================
    %% 这一层只关注逻辑，不关注具体芯片
    subgraph Phase_Software [阶段二：软件架构定义]
        direction TB
        Task_Spec --> Arch_Agent[架构师 Agent]:::brain
        
        Arch_Agent -->|1.定义接口| Interfaces[抽象接口 .h]:::soft
        Arch_Agent -->|2.设计调度| Scheduler[调度器/状态机]:::soft
        Arch_Agent -->|3.实现逻辑| Biz_Logic[纯业务代码 .c]:::soft
        
        %% SIL 软件回环
        Interfaces & Biz_Logic --> Mock_Gen[Mock 虚拟对象生成]
        Mock_Gen --> SIL_Runner[PC端 仿真运行器]:::test
        SIL_Runner -->|逻辑验证结果| Arch_Agent
    end

    %% ================= 3. 硬件实现域 (Hardware Domain) =================
    %% 这一层负责将逻辑落地到具体芯片
    subgraph Phase_Hardware [阶段三：硬件映射与实现]
        direction TB
        HW_Agent[硬件工程师 Agent]:::brain
        Task_Spec & Interfaces --> HW_Agent
        
        HW_Agent -->|4.查阅手册| KB[(SVD / 手册 / RAG库)]
        HW_Agent -->|5.生成映射| Pin_Map[引脚/外设映射表]:::data
        
        Pin_Map --> CubeMX[CubeMX 配置器]:::hard
        CubeMX --> HAL[HAL 底层驱动]:::hard
        
        HW_Agent --> Adapter_Gen[适配层生成器]:::hard
        Adapter_Gen -->|连接 HAL 与 接口| Adapter[驱动适配层 .c]:::hard
        
        %% HIL 硬件回环
        HAL & Adapter --> Builder[交叉编译 & 烧录]:::test
        Builder --> Board[物理硬件板卡]
        Board -->|串口/RTT 日志| Log_Analyzer[回环诊断器]:::test
        Log_Analyzer -->|硬件修正建议| HW_Agent
    end

    %% 跨域连接：业务逻辑与构建工具的连接
    Biz_Logic -.->|编译链接| Builder
```

---

## 🔄 全链路执行流程 (Execution Workflow)

这是一个 **人机协作 (Human-in-the-Loop)** 的过程。AI 负责繁琐的生成工作，人类负责关键节点的决策与确认。

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
    User->>AI_HW: 确认或修改引脚 (人工介入)
    AI_HW->>AI_HW: 调用 CubeMX 生成 HAL 驱动
    AI_HW->>AI_HW: 生成适配层 (Driver_Adapter.c)
    AI_HW->>HIL: 编译并下载到 STM32

    Note over HIL, User: 步骤 4: 最终验证
    HIL->>HIL: 硬件运行 & 回传日志
    HIL-->>User: 仪表盘显示：温度 31度，风扇转速 50%
```

---

## 🧩 仓库矩阵 (Repository Matrix)

ReifyFlow 采用微服务化的 **Multi-Repo** 架构，各模块职责分明：

| 仓库名称 | 核心职责 | 技术栈 |
| :--- | :--- | :--- |
| **[`reify-protocol`](https://github.com/ReifyFlow/reify-protocol)** | 定义所有组件交互的数据标准（软件定义、硬件映射、日志格式）。**所有开发由此开始。** | JSON Schema |
| **[`reify-core`](https://github.com/ReifyFlow/reify-core)** | 核心编排引擎。集成 LLM、SVD 解析器、PDF 检索引擎、日志分析器。 | Python (FastAPI) |
| **[`reify-driver`](https://github.com/ReifyFlow/reify-driver)** | 通用硬件适配器。屏蔽厂商差异，操作 CubeMX/CMake，调用编译器与下载器。 | Python CLI |
| **[`reify-studio`](https://github.com/ReifyFlow/reify-studio)** | 可视化交互工作台 (VS Code 插件)。提供硬件拓扑图绘制、手册联动阅读。 | TS / React |
| **[`reify-chips`](https://github.com/ReifyFlow/reify-chips)** | 芯片知识库。存放 SVD 寄存器定义、Datasheet 索引映射、代码模板。 | Data |

---

## 📂 标准化工程结构 (Project Structure)

ReifyFlow 生成的工程遵循严格的 **分层解耦** 原则：

```text
Project_Root/
├── .oef/                       # [OEF配置区] - 存放核心元数据
│   ├── task_spec.json          # 1. 任务规格书 (AI生成)
│   ├── arch_graph.json         # 2. 软件架构图数据
│   └── hw_map.json             # 3. 硬件映射数据
│
├── core/                       # [软件架构层 - 纯逻辑] - 可以在电脑上跑
│   ├── inc/
│   │   ├── interface_temp.h    # 抽象接口：只定义 Get_Temp()，不含 HAL 库
│   │   └── interface_motor.h
│   ├── src/
│   │   ├── app_main.c          # 调度器/主循环
│   │   └── business_logic.c    # 核心算法 (PID, 状态机)
│   └── tests/                  # SIL 测试代码 (PC端运行)
│
├── bsp/                        # [硬件实现层 - 适配器] - 连接软硬的胶水
│   ├── stm32f103/              # 具体芯片实现
│   │   ├── adapter_temp.c      # 实现 interface_temp.h，调用 HAL_ADC
│   │   └── adapter_motor.c     # 实现 interface_motor.h，调用 HAL_TIM
│
├── drivers/                    # [底层驱动层 - 自动生成] - CubeMX 的地盘
│   ├── STM32CubeMX/            # CubeMX 工程目录
│   │   ├── Core/Src/main.c     # 硬件初始化
│   │   └── Drivers/            # HAL 库文件
│   └── linker/                 # 链接脚本
│
├── tools/                      # [工具链]
│   ├── build.py                # 自动构建脚本
│   └── monitor.py              # 日志监控脚本
│
└── Makefile                    # 顶层构建规则
```

## 🔄 The Workflow (标准工作流)

我们定义了 **T-V-E-L** 标准流程，确保每一步都可控：

1. **Translate (翻译)** : AI 将自然语言需求转化为 **Task Spec** (JSON)。
2. **Verify (验证)** : `reify-studio` 渲染出硬件连线图，用户进行**可视化确认**。
3. **Execute (执行)** : `reify-driver` 修改底层配置，生成代码并烧录。
4. **Loopback (回环)** : 硬件日志回传，AI 进行故障诊断。

---

## 🗺️ MVP Roadmap (当前计划)

目前项目处于 **MVP 开发阶段**，主要聚焦于 STM32F103 的点灯与串口通信闭环。

- [ ] **Step 1**: 完成 `reify-protocol` 协议定义。
- [ ] **Step 2**: 实现 `reify-driver` 对 STM32CubeMX `.ioc` 文件的读写。
- [ ] **Step 3**: 跑通 "自然语言 -> 代码生成 -> 硬件运行" 的单向链路。
- [ ] **Step 4**: 开发 VS Code 插件可视化界面。

---

## 🤝 Join Us

ReifyFlow 是一个开放的实验。如果你对 **AI Agent**、**嵌入式开发自动化** 或 **编译器设计** 感兴趣，欢迎 Star 或提交 PR。

*License: AGPL-3.0 (Core/Driver) & MIT (Protocol/UI)*
