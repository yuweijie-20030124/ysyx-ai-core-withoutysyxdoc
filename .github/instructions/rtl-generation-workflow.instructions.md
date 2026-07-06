---
description: "当前 RV32E CPU 的 RTL 生成流程。写或修改 Verilog RTL 前，先给出需求、协议、状态机、不变量、数据通路和拓扑，再落代码。"
applyTo: "**/*.{v,sv,vh,svh}"
---

# RTL 生成流程

写或修改 Verilog/SystemVerilog RTL 前，先完成以下推导。只解释代码或只读分析时可简化；
一旦要落盘改 RTL，就不能跳过。

## 1. 需求

说明模块要做什么：

- 功能目标。
- 输入/输出端口、位宽、时钟和复位。
- 单周期、多周期或流水线时序要求。
- 上下游边界和 backpressure。
- 明确不在本次范围内的事项。

## 2. 协议规则

说明接口如何工作：

- valid/ready、req/ack、固定时序或组合返回。
- payload 在哪一拍稳定。
- stall、flush、kill、异常和重试规则。
- 与 RV32E_Zicsr、AXI4、icache 或内部流水线协议的对应关系。

## 3. 状态机

对有状态控制逻辑列出：

- 状态枚举、复位状态。
- 转移条件。
- 每个状态的输出行为。
- 非法状态的处理。

## 4. 不变量

列出设计必须长期保持的性质，例如：

- RV32E 可见通用寄存器为 x0-x15；写 x0 结果保持为 0。
- 同周期最多一个写回源写寄存器堆。
- valid 未被接收前 payload 不变化。
- flush 优先于普通流水线推进。
- 异常进入 trap 时 `mepc`、`mcause`、PC 跳转关系一致。

每条不变量要说明触发条件和违反后果。

## 5. 数据通路约束

固定硬件骨架：

- 关键寄存器和 pipeline register。
- mux、旁路、hazard 检测、flush/stall 优先级。
- ALU、CSR、load/store、branch/jump、icache/AXI 访问路径。
- 位宽、符号扩展、对齐和字节写使能。
- 可能的关键路径。

## 6. RTL 拓扑

写代码前先列拓扑：

- module 边界。
- 状态寄存器列表。
- 主要组合逻辑块。
- FSM 和流水级 valid 流向。
- reset/flush/stall/kill 优先级。
- 共享资源的 mux、enable 和控制来源。

## 7. 落 RTL

代码应能映射回上面的协议、状态机和不变量：

- 可综合硬件优先写 `.v`。
- 时序逻辑用 `always @(posedge clock)`。
- 组合逻辑用 `always @(*)`，块首给默认值，避免锁存器。
- 小型纯组合 helper 可用 `function`；FSM、仲裁、握手、流水线寄存器不能藏进 function。
- 新增模块尽量一个 module 一个文件，并更新对应 filelist/Makefile。

## 8. 验证

至少做一种与改动匹配的验证：

- Verilator lint/build/sim。
- 小型 testbench。
- 与 C 模拟器 trace 对比。
- RT-Thread/CI 路径验证。

无法验证时要说明缺口。
