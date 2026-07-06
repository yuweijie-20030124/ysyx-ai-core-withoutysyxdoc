# AGENTS.md — ysyx_ai_core 当前工作规范

本文件是当前仓库内 AI agent 的轻量入口。项目事实以根目录 `memory/`
为准，尤其是：

- `memory/RISCV-32E Core AI生成规格文档.md`
- `memory/tools.md`
- `memory/spec/riscv-unprivileged.pdf`

## 当前目标

当前目标不是 RV64 Linux/Ubuntu bring-up。当前只面向：

- 用 C 语言实现一个 NEMU 风格的 RV32E_Zicsr CPU 模拟器。
- 用 Verilog 实现一个五级流水线 RV32E_Zicsr CPU。
- 通过 Verilator 跑 RT-Thread，终端能打印信息，并通过
  `/home/ywj/ysyx_ai_core/monitor.py` CI 检查。
- 跑 RT-Thread 时重点观察是否出现 `hit good trap`。

## 旧目标边界

以下旧 RV64/Linux 方向的信息不应作为当前需求：

- RV64/RV64GC、Sv39、S/U mode、OpenSBI、Linux kernel、Ubuntu 22.04。
- initramfs/rootfs、systemd、virtio、PLIC/CLINT Linux 设备 bring-up。
- QEMU reference、Ubuntu shell、APT、SSH、framebuffer/display 相关 gate。
- 旧 `.github/task-runs`、`.github/db-backup`、agent e2e 数据库和大段历史 evidence。

如果遇到旧文档或代码提到这些内容，只能当历史参考，不能把它们加入当前实现范围。

## 必读顺序

非平凡任务开工前先读：

1. `memory/RISCV-32E Core AI生成规格文档.md`
2. `memory/tools.md`
3. `memory/whataido/requirement.md`
4. 与当前改动相关的源码、Makefile、测试脚本
5. 若要写或改 Verilog RTL，再读
   `.github/instructions/rtl-generation-workflow.instructions.md`

## 规格摘要

- ISA：RV32E_Zicsr。
- 特权级：仅 M 模式，单核。
- 复位地址：参数可配置，默认 `0x20000000`。
- 特权指令：`ecall`、`ebreak`、`mret`、`wfi`，其中 `wfi` 可实现为 `nop`。
- CSR：只实现 `mvendorid`、`marchid`、`mepc`、`mtvec`、`mcause`、`mstatus`。
  其他 CSR 访问触发非法指令异常。
- 访存：小端、只支持对齐访问；未对齐访问触发对应 misaligned fault。
- 地址翻译/PMP/PMA：不支持，整个地址空间开放访问。
- 中断：不支持异步中断，只支持规格列出的同步异常。
- 总线：外部采用 32 位 AXI4 master；保留的 `io_interrupt` 和 AXI slave 输入不得使用，
  保留输出硬编码为 0。
- icache：8 条指令容量，16B cacheline，直接映射，支持突发；`fence.i` 清空 icache。
- dcache：不支持。
- 性能计数：需要能计算 icache AMAT。

## 工具

- 看 VCD 波形：`gtkwave`。
- 硬件仿真：`verilator`。
- RISC-V 工具链前缀：`riscv64-linux-gnu-`。
- 综合/STA：`/home/ywj/software/yosys-sta`。

使用到的新工具、用途和命令应补充到 `memory/tools.md`。

## 工作方式

- 优先按当前规格做最小闭环，不引入未要求的 Linux/Ubuntu 复杂度。
- NEMU C 模拟器和 Verilog RTL 要保持语义一致；有条件时用同一程序或 trace 对比。
- 任何源码修改后都要运行可行的最小验证；如果验证缺失，最终说明原因。
- AI 做的任何文件修改，都必须记录到 `memory/whataido/` 下的 Markdown。
- 用户和 AI 的对话内容，必须记录到 `memory/whataitalkwithman/` 下的 Markdown。
- 不删除或回退用户已有改动，除非用户明确要求。
- 文档和注释默认使用中文。
