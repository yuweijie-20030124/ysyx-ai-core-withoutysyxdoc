你需要完成的是RISC-V五级流水线，接入AXI总线
ai不允许查看一生一芯官网的讲义。
ai做的每一次改动都应该写到markdown文件中，方便ai自己后续迭代。放在/home/ywj/ysyx_ai_core/memory/whataido
ai与人类的每一次对话都应该放在另一个markdown文件中。放在/home/ywj/ysyx_ai_core/memory/whataitalkwithman
全程放在我的bash里面跑，不要污染我的fish环境。
我允许你进行git 提交，本地仓库也好，远程仓库也好，但是要标注好是你提交的。类似“ai commit”
步骤：
1.完成nemu，跑完riscv架构的am-kernels中的cpu-test功能。
2.完成nemu，跑完riscv架构的RT-thread，这个RT-thread放在~/Templates那里。
3.实现nemu的difftest框架，搭建npc的仿真环境，尽量保持与nemu类似，除了处理器执行的代码其他可以复用，这里的复用指的是复制粘贴到npc中，然后按照符合verilator仿真的方式修改，其中NPC与仿真环境的交互可以用Verilator中的DPIC机制，具体可以查官网https://verilator.org/guide/latest/
4.首先确保NPC可以跑完RT-Thread之后，将NPC接入ysyxSoc总线中，并且继续在verilator环境中跑仿真Soc环境，直到跑完soc环境下的RT-Thread。
5.跑完CI测试，也就是/home/ywj/ysyx_ai_core/monitor.py。
