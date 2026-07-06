# ysyx_ai_core — Copilot/Agent 指令

本仓库当前聚焦 RV32E_Zicsr，不再沿用旧 RV64 Linux/Ubuntu bring-up 目标。
所有工程判断优先服从 `memory/RISCV-32E Core AI生成规格文档.md`。

## 当前交付物

- C 语言 NEMU 风格 CPU 模拟器。
- Verilog 五级流水线 CPU。
- Verilator 仿真 RT-Thread，打印终端信息并通过
  `/home/ywj/ysyx_ai_core/monitor.py`。
  禁止修改这个monitor.py测试

## 不在范围内

不要主动实现或引入：

- RV64/RV64GC、Sv39、S/U mode。
- OpenSBI、Linux kernel、Ubuntu、systemd、rootfs/initramfs。
- virtio、SSH、APT、framebuffer、QEMU/Linux 设备 gate。
- 旧 agent e2e、旧 `.github/memory`、旧 task-run 证据库工作流。

## 开工上下文

先读：

- `memory/RISCV-32E Core AI生成规格文档.md`
- `memory/tools.md`
- `memory/whataido/requirement.md`
- 当前源码目录中的 README、Makefile、测试脚本

写 Verilog 前还要读：

- `.github/instructions/rtl-generation-workflow.instructions.md`

## 实现约束

- ISA 只按 RV32E_Zicsr 实现；寄存器数量按 RV32E 处理。
- 只支持 M 模式；`mstatus.MPP` 可硬编码为 M，其余位按规格硬编码为 0。
- 未实现 CSR 访问必须触发非法指令异常。
- 访存只支持地址对齐的小端访问。
- 不支持虚拟内存、PMP、PMA、dcache 和异步中断。
- `fence` 可作为 `nop`，`fence.i` 要清空 icache。
- AXI4 master 为 32 位；保留 slave 端口输出硬编码为 0。

## 验证习惯

- C 模拟器优先用小程序、指令单测和异常/CSR 边界测试验证。
- Verilog RTL 优先用 Verilator lint/build/sim、波形和与 C 模拟器对比验证。
- RT-Thread 目标以 `monitor.py` 和 `hit good trap` 作为关键观察点。
- 不能因为旧 RV64/Linux 证据曾通过，就声明当前 RV32E 目标已完成。
- AI 做过的文件修改要写入 `memory/whataido/` 的 Markdown。
- 用户和 AI 的对话要写入 `memory/whataitalkwithman/` 的 Markdown。

## 风格

- C 代码遵循本仓库已有风格。
- 可综合 RTL 默认写 Verilog-2001 风格，模块和控制边界清晰。
- 注释只写有助于理解硬件/模拟器意图的内容。
- 文档和注释默认中文。
