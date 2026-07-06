# RISCV\-32E Core AI生成规格文档

# 核心规格

## 基本配置

支持的指令集：RV32E\_Zicsr。

支持的特权级：仅M模式。

核心数量：单核。且微架构设计仅需考虑单核的情况。

复位地址：参数可配置，默认为`0x20000000`。

## 实现的特权指令

实现ecall、ebreak、mret和wfi指令，其中wfi可以实现为nop。

## 实现的CSR

mvendorid：硬编码为0。

marchid：硬编码为0。

mepc：提供一个足以容纳所有对齐到4字节的32位地址的实现。

mtvec：提供一个足以容纳所有对齐到4字节的32位地址的实现。

mcause：提供一个足以表示该核心可能遇到的所有中断与异常的实现。

mstatus：MPP硬编码为M模式、其余位硬编码为0。

其他所有CSR一律为未实现。尝试访问这些CSR应该触发非法指令异常。

## 访存行为

访存为顺序访存，fence指令被实现为nop。

虚拟内存与地址翻译：不支持。

总线：仅支持地址对齐的小端序访存，地址未对齐的访存会产生load/store/instruction misaligned fault。对外采用32位位宽的AXI 4协议。AXI 返回SLVERR/DECERR时产生load/store/instruction access fault。

PMP与PMA：不支持，整个地址空间开放访问。

## 中断与异常行为

支持以下类型的异常（sync exception）：

- Instruction address misaligned

- Instruction access fault

- Illegal instruction

- Breakpoint

- Load address misaligned

- Load access fault

- Store/AMO address misaligned

- Store/AMO access fault

- Environment call from M\-mode

不支持中断（async interrupt）。

# 缓存配置

使用触发器实现容量为8条指令（32字节），cacheline大小为16字节，associativity为1的指令缓存。指令缓存假设所有可取指令的地址都是可缓存的。

指令缓存需支持突发传输。

fence\.i指令被实现为清空指令缓存。

不支持数据缓存。

内部要有性能计数器来计算icache的AMAT

# 外部接口（需要进一步说明）

|时钟|input|`clock`||||
|---|---|---|---|---|---|
|复位\(高电平有效\)|input|`reset`||||
|外部中断|input|`io_interrupt`||||
|AXI4 Master总线|||AXI4 Slave总线|||
|`input`||`io_master_awready`|`output`||`io_slave_awready`|
|`output`||`io_master_awvalid`|`input`||`io_slave_awvalid`|
|`output`|`[31:0]`|`io_master_awaddr`|`input`|`[31:0]`|`io_slave_awaddr`|
|`output`|`[3:0]`|`io_master_awid`|`input`|`[3:0]`|`io_slave_awid`|
|`output`|`[7:0]`|`io_master_awlen`|`input`|`[7:0]`|`io_slave_awlen`|
|`output`|`[2:0]`|`io_master_awsize`|`input`|`[2:0]`|`io_slave_awsize`|
|`output`|`[1:0]`|`io_master_awburst`|`input`|`[1:0]`|`io_slave_awburst`|
|`input`||`io_master_wready`|`output`||`io_slave_wready`|
|`output`||`io_master_wvalid`|`input`||`io_slave_wvalid`|
|`output`|`[31:0]`|`io_master_wdata`|`input`|`[31:0]`|`io_slave_wdata`|
|`output`|`[3:0]`|`io_master_wstrb`|`input`|`[3:0]`|`io_slave_wstrb`|
|`output`||`io_master_wlast`|`input`||`io_slave_wlast`|
|`output`||`io_master_bready`|`input`||`io_slave_bready`|
|`input`||`io_master_bvalid`|`output`||`io_slave_bvalid`|
|`input`|`[1:0]`|`io_master_bresp`|`output`|`[1:0]`|`io_slave_bresp`|
|`input`|`[3:0]`|`io_master_bid`|`output`|`[3:0]`|`io_slave_bid`|
|`input`||`io_master_arready`|`output`||`io_slave_arready`|
|`output`||`io_master_arvalid`|`input`||`io_slave_arvalid`|
|`output`|`[31:0]`|`io_master_araddr`|`input`|`[31:0]`|`io_slave_araddr`|
|`output`|`[3:0]`|`io_master_arid`|`input`|`[3:0]`|`io_slave_arid`|
|`output`|`[7:0]`|`io_master_arlen`|`input`|`[7:0]`|`io_slave_arlen`|
|`output`|`[2:0]`|`io_master_arsize`|`input`|`[2:0]`|`io_slave_arsize`|
|`output`|`[1:0]`|`io_master_arburst`|`input`|`[1:0]`|`io_slave_arburst`|
|`output`||`io_master_rready`|`input`||`io_slave_rready`|
|`input`||`io_master_rvalid`|`output`||`io_slave_rvalid`|
|`input`|`[1:0]`|`io_master_rresp`|`output`|`[1:0]`|`io_slave_rresp`|
|`input`|`[31:0]`|`io_master_rdata`|`output`|`[31:0]`|`io_slave_rdata`|
|`input`||`io_master_rlast`|`output`||`io_slave_rlast`|
|`input`|`[3:0]`|`io_master_rid`|`output`|`[3:0]`|`io_slave_rid`|

目前，`io_interruput`和所有AXI slave接口是保留的。对于保留的输入接口，其输入不应该被使用。对于保留的输出接口，其输出应该被硬编码为0。

# 任务

用verilator跑出RT\-Thread程序，终端打印出信息，通过CI测试。

CI测试：https://github\.com/sashimi\-yzh/ysyx\-submit\-test/blob/master/monitor\.py

跑rtt要看有无hit good trap

CI测试在这里，/home/ywj/ysyx_ai_core/monitor.py

# 提供信息（需要确认）

我认为需要提供给ai提供ysyx\-rtt源码ysyxSoc。

需要提供ysyx\-workbench的nemu abstract\-machine吗，ysyx提供给学生的一切环境提供给ai是否可行？

人为每一次提示词都应该记录下来。

ai做的每一个改动需要记录下来吗？



