# Functional implementation
+ 1. In the `TaskControlBlock` struct within `task.rs`, add a `sys_call_times` array to track the count of each system call made by the current `task`.
+ 2. On each system call execution, increment the respective system call count in the `sys_call_times` array within the `TaskControlBlock` struct of the `current_task` in the global `TASK_MANAGER`.
+ 3. Implement the `get_sys_call_times` method for `TaskManager` to retrieve a copy of the `sys_call_times` array from the `TaskControlBlock` struct of the `current_task`.
+ 4. Complete the `sys_task_info` function in `process.rs` by calling `get_sys_call_times` and `get_time_ms` to populate the `syscall_times` and `time` fields in the `TaskInfo` struct, setting the `status` field to `Running`.

# Short answer question answers
## Part1
正确进入 U 态后，程序的特征还应有：使用 S 态特权指令，访问 S 态寄存器后会报错。 请同学们可以自行测试这些内容（运行 三个 bad 测例 (ch2b_bad_*.rs) ）， 描述程序出错行为，同时注意注明你使用的 sbi 及其版本。

Rustsbi 版本: 0.2.0-alpha.2
报错list: 
```bash
[kernel] PageFault in application, bad addr = 0x0, bad instruction = 0x804003c4, kernel killed it.
[kernel] IllegalInstruction in application, kernel killed it.
[kernel] IllegalInstruction in application, kernel killed it.
```

`ch2b_bad_address.rs` 由于除0错误触发异常退出
`ch2b_bad_instructions.rs` 在用户态非法使用指令`sret`
`ch2b_bad_register.rs` 在用户态非法使用指令`csrr`


## Part2
深入理解 trap.S 中两个函数 __alltraps 和 __restore 的作用，并回答如下问题:

### L40：刚进入 __restore 时，a0 代表了什么值。请指出 __restore 的两种使用情景。
> 1. 刚进入 __restore 时，a0 代表了系统调用的第一个参数
> 2. __restore 的作用包括:
>   - 从系统调用和异常返回时, 恢复返回的用户态的上下文
>   - 任务切换时, 恢复要切换的任务的上下文信息

### L43-L48：这几行汇编代码特殊处理了哪些寄存器？这些寄存器的的值对于进入用户态有何意义？请分别解释。
```bash
ld t0, 32*8(sp) # 内核栈 32*8(sp) 处存储了原 sstatus 寄存器的值, 将其读取到 t0
ld t1, 33*8(sp) # 内核栈 32*8(sp) 处存储了原 sepc 寄存器的值, 将其读取到 t1
ld t2, 2*8(sp) # 内核栈 32*8(sp) 处存储了原 sscratch 寄存器的值, 将其读取到 t2
csrw sstatus, t0 # 将 t0中原 sstatus 寄存器的值读取到 sstatus
csrw sepc, t1 # 将 t0中原 sepc 寄存器的值读取到 sepc
csrw sscratch, t2 # 将 t0中原 sscratch 寄存器的值读取到 sscratch
```

### L50-L56：为何跳过了 x2 和 x4？
1. 跳过`x2`:因为`x2`对应的用户栈指针保存到了sscratch寄存器, 不需要从内核栈中进行恢复
2. 跳过`x4`:因为并没有使用它, 无需恢复

### L60：该指令之后，sp 和 sscratch 中的值分别有什么意义？
`sp`指向用户栈, `sscratch`指向内核栈

### __restore：中发生状态切换在哪一条指令？为何该指令执行之后会进入用户态？
`sret`后发生了状态切换, 执行该指令后, PC设置为 `sepc` 寄存器的值。`sepc` 存储着产生中断或异常前的指令地址，实现了到原始代码的返回。

### L13：该指令之后，sp 和 sscratch 中的值分别有什么意义？
```bash
csrrw sp, sscratch, sp
```
`sp`, `sscratch`寄存器的内容被交换, `sp`保存了原`sscratch`中的内核栈指针, `sscratch`保存了原`sp`中的用户栈栈指针

### 从 U 态进入 S 态是哪一条指令发生的？
command: `ecall`

# 荣誉准则

+ 1.在完成本次实验的过程（含此前学习的过程）中，我曾分别与团队成员周顺进行以下方面做过交流，还在代码中对应的位置以注释形式记录了具体的交流对象及内容：

关于实现系统调用的步骤，关于需要考虑的全局变量控制

+ 2.此外，我也参考了 以下资料 ，还在代码中对应的位置以注释形式记录了具体的参考来源及内容：

rust圣经，rBook指南

3. 我独立完成了本次实验除以上方面之外的所有工作，包括代码与文档。 我清楚地知道，从以上方面获得的信息在一定程度上降低了实验难度，可能会影响起评分。

4. 我从未使用过他人的代码，不管是原封不动地复制，还是经过了某些等价转换。 我未曾也不会向他人（含此后各届同学）复制或公开我的实验代码，我有义务妥善保管好它们。 我提交至本实验的评测系统的代码，均无意于破坏或妨碍任何计算机系统的正常运转。 我清楚地知道，以上情况均为本课程纪律所禁止，若违反，对应的实验成绩将按“-100”分计。

# 实验设计与难度分析
## 难度与工作量

实验难度属于中等偏上，主要体现在以下几方面：
1. **Rust 的系统级开发复杂度**：Rust 的所有权机制和复杂的代码结构会给部分学生带来一定挑战，特别是在操作系统内核环境下。
2. **任务模块间的交互理解**：对 `TaskControlBlock` 和 `TaskManager` 结构的理解，以及系统调用的处理机制需要一定的背景知识。
3. **实验工作量适中**：任务分配较合理，但在结构扩展与信息获取的整合方面需要精细的代码调整，对代码规范和理解要求较高。

### 改进建议

1. **增加前置知识讲解**：在实验说明中加入对系统调用机制、任务管理结构等的概述，帮助学生更快理解任务之间的关系。
2. **提供样例代码片段**：特别是对于生命周期管理和所有权控制，提供简化的样例代码片段能有效帮助学生理解。
3. **引入更多调试信息**：增加对系统调用计数的实时调试输出，可以帮助学生确认实现的正确性，同时增强实验的反馈体验。


