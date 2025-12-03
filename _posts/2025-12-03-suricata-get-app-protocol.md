---
title: Suricata detect app protocol(ai)
date: 2025-12-02 00:11:00
pin: true
categories: [suricata]
tag: [suricata]
---

# Suricata 中基于端口的协议检测：修正后的详细流程

根据我们对 Suricata 源代码的分析，特别是澄清了基于端口的检测在模式匹配（PM）和探测解析器（PP）中的具体作用后，以下是 Suricata 中基于端口的协议检测从注册到使用的详细流程：

---

### Suricata 中基于端口的协议检测：修正后的详细流程

1.  **配置阶段 (Configuration)**
    *   **用户操作**：用户在 Suricata 的配置文件 `suricata.yaml` 中，通过 `app-layer.protocols.<protocol_name>.detection-ports` 项，指定哪些应用层协议预计在哪些端口上被检测。
    *   **目的**：为 Suricata 提供端口与预期协议之间的显式映射关系。

2.  **初始化/注册阶段 (Initialization/Registration - Suricata 启动时)**
    *   **配置解析**：Suricata 启动时，`SCAppLayerProtoDetectPPParseConfPorts` 函数负责解析 `suricata.yaml` 中的端口配置。
    *   **探测解析器注册**：解析完成后，`SCAppLayerProtoDetectPPRegister` 函数会被调用。它接收解析出的 IP 协议（如 TCP）、具体的端口（或端口范围）以及对应的应用层协议 (`AppProto alproto`)，并将其与**相应的探测解析器函数**关联。
    *   **内部数据结构构建**：`SCAppLayerProtoDetectPPRegister` 通过调用 `AppLayerProtoDetectInsertNewProbingParser`，将这些关联信息（IP 协议、端口、应用协议、探测解析器函数）插入到 Suricata 内部的一个高效查找表 (`alpd_ctx.ctx_pp`) 中。这个查找表的作用是：**给定一个 IP 协议和端口，能够快速找到为该组合注册的所有探测解析器**。
    *   **目的**：在 Suricata 启动时，建立一个优化过的查找机制，用于在后续处理中，根据流的端口快速定位到可能适用的**探测解析器集合**。

3.  **流创建阶段 (Flow Creation - 数据包处理时)**
    *   **事件**：当 Suricata 接收到一个网络数据包，并识别出一个新的网络流（例如，一个新的 TCP 连接开始）时。
    *   **信息提取**：Suricata 从数据包头部提取关键信息，包括源/目的 IP 地址、源/目的端口号和传输层协议（TCP/UDP），并创建或更新 `Flow` 对象。
    *   **目的**：建立一个网络流的上下文 (`Flow` 对象)，其中包含了进行协议检测所需的基本信息，包括端口。

4.  **应用层协议检测阶段 (`AppLayerProtoDetectGetProto` - 顶层协调者)**
    *   `AppLayerProtoDetectGetProto` 是顶层的协议检测函数，它协调多个检测尝试，直到识别出协议或所有方法都失败。

    *   **第一尝试：模式匹配 (Pattern Matching - PM)**
        *   **函数调用**：`AppLayerProtoDetectPMGetProto` 会被调用。
        *   **内部机制**：这个函数主要依赖于**多模式匹配器 (MPM)**。它会利用流的 IP 协议（通过 `f->protomap`）和方向来选择一组预编译的模式进行匹配。例如，TCP 流量只会用 TCP 相关的模式去匹配。
        *   **端口作用**：**端口信息在 `AppLayerProtoDetectPMGetProto` 内部不会直接用于过滤或加速模式匹配。**它的加速来自于模式集的预专门化（按 IP 协议和方向）。
        *   **目的**：快速识别具有明显特征的协议。如果匹配成功，则立即确定协议并返回。

    *   **第二尝试：探测解析器 (Probing Parsers - PP)**
        *   **条件**：如果模式匹配（PM）未能成功识别协议。
        *   **函数调用**：`AppLayerProtoDetectPPGetProto` 会被调用。
        *   **端口作用 (核心)**：**这里就是基于端口的检测发挥关键作用的地方。**
            *   `AppLayerProtoDetectPPGetProto` 会获取当前流的目的端口 (`dp`) 和源端口 (`sp`)。
            *   它会查询在 Suricata 启动时构建的内部查找表 (`alpd_ctx.ctx_pp`)，**根据 `ipproto` 和 `dp/sp` 查找并检索所有已注册的探测解析器。**
            *   **高效筛选**：通过这种方式，`AppLayerProtoDetectPPGetProto` 避免了对所有可能的探测解析器进行盲目尝试，而是只执行那些通过端口信息被认为是**相关且有可能成功**的探测解析器。这极大地提高了效率。
        *   **目的**：如果模式匹配失败，则使用更深入、但通过端口信息预筛选过的探测解析器，尝试解析数据以确定协议。如果匹配成功，则立即确定协议并返回。

    *   **第三尝试：预期列表 (Expectation List - PE)**
        *   **条件**：如果模式匹配（PM）和探测解析器（PP）都未能成功识别协议。
        *   **函数调用**：`AppLayerProtoDetectPEGetProto` 会被咨询。
        *   **目的**：利用流中积累的上下文信息或此前检测到的协议所产生的预期来尝试确定协议。

---

**总结**：

基于端口的检测在 Suricata 的协议识别中扮演了一个**至关重要的“智能筛选器”角色**。它主要通过**预配置和在探测解析器（PP）阶段进行目标性查找**来提高效率，而不是直接加速模式匹配（PM）。这种分层且智能的策略确保了 Suricata 能够高效而准确地识别网络流量中的应用层协议。
