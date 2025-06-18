---
layout: post
title: GDB 调试指南
date: 2025-06-18 08:32:00
tags: 
    - RE
    - GDB
categories: 
    - GDB
license: cc_by_nc_sa
---

# GDB 调试指南


## 基础命令

| 命令               | 缩写 | 功能                       |
| ------------------ | ---- | -------------------------- |
| `run [参数]`       | `r`  | 启动程序                   |
| `quit`             | `q`  | 退出 GDB                   |
| `list [行号]`      | `l`  | 查看源代码                 |
| `help [命令]`      | `h`  | 查看帮助                   |
| `backtrace [full]` | `bt` | 查看调用栈（包含局部变量） |
| `info [类型]`      | `i`  | 显示程序状态信息           |
| `print [表达式]`   | `p`  | 计算并显示表达式值         |
| `ptype [变量]`     |      | 查看变量类型               |

**启动调试：**
```bash
gdb [可执行文件]
(gdb) run [程序参数]
```

**信息查看命令：**
```bash
(gdb) info functions    # 查看所有函数
(gdb) info variables    # 查看所有全局变量
(gdb) info locals       # 查看当前函数局部变量
(gdb) info args         # 查看当前函数参数
```

---

## 断点操作

### 断点设置

| 命令                | 示例          | 说明                 |
| ------------------- | ------------- | -------------------- |
| `break [函数]`      | `b main`      | 函数入口断点         |
| `break [行号]`      | `b 15`        | 当前文件行断点       |
| `break [文件]:[行]` | `b test.c:20` | 指定文件行断点       |
| `break *[地址]`     | `b *0x4005a6` | 内存地址断点         |
| `tbreak`            | `tbreak 30`   | 临时断点（触发一次） |

### 条件断点
```bash
(gdb) break 25 if x==5         # 变量条件
(gdb) break calculate if n>100 # 函数参数条件
```

### 观察点
```bash
(gdb) watch total_count        # 变量写入中断
(gdb) rwatch *0x7fffffffd0    # 内存读取中断
(gdb) awatch ptr              # 变量读写中断
```

### 断点管理
```bash
(gdb) info breakpoints         # 查看所有断点
(gdb) disable 2                # 禁用2号断点
(gdb) enable 2                 # 启用2号断点
(gdb) delete 2                 # 删除2号断点
(gdb) commands 2               # 为断点设置命令序列
> print var
> continue
> end
```

---

## 数据查看与修改

### 基本查看
```bash
(gdb) print my_var             # 查看变量值
(gdb) print /x *ptr            # 十六进制格式
(gdb) print array[5]@10        # 查看数组元素
(gdb) print $rax               # 查看寄存器值
```

### 结构体/对象查看
```bash
(gdb) print *obj               # 查看对象内容
(gdb) print student.name       # 查看结构体成员
```

### 数据修改
```bash
(gdb) set var count=0          # 修改变量值
(gdb) set $eax=0               # 修改寄存器值
(gdb) set {int}0x7fffffffdc=42 # 修改内存值
```

---

## 程序执行控制

| 命令           | 缩写   | 说明                       |
| -------------- | ------ | -------------------------- |
| `continue`     | `c`    | 继续执行                   |
| `step`         | `s`    | 单步进入（进入函数）       |
| `next`         | `n`    | 单步越过（不进入函数）     |
| `finish`       | `fin`  | 执行到函数返回             |
| `until [行号]` | `u 30` | 运行到指定行               |
| `stepi`        | `si`   | 单条汇编指令（进入函数）   |
| `nexti`        | `ni`   | 单条汇编指令（不进入函数） |

**信号控制：**
```bash
(gdb) signal SIGINT            # 发送中断信号
(gdb) handle SIGSEGV stop      # 设置信号处理
```

---

## 显示布局

| 命令   | 值    | 说明                     |
| ------ | ----- | ------------------------ |
| layout | asm   | 显示汇编窗口             |
| layout | regs  | 显示寄存器窗口           |
| layout | split | 同时显示源代码和汇编窗口 |

```bash
(gdb) layout asm             # 显示汇编窗口
(gdb) layout regs            # 显示寄存器窗口
(gdb) layout split           # 同时显示源代码和汇编窗口
```

## 汇编级调试

| 命令        | 缩写   | 说明                           |
| :---------- | :----- | :----------------------------- |
| `stepi`     | `si`   | 执行一条汇编指令（可进入函数） |
| `nexti`     | `ni`   | 执行一条汇编指令（不进入函数） |
| `stepi [n]` | `si 5` | 执行n条汇编指令                |

### 反汇编命令
```bash
(gdb) disassemble              # 当前函数反汇编
(gdb) disassemble main         # 指定函数反汇编
(gdb) disassemble 0x400536,0x40054a # 地址范围反汇编
(gdb) disassemble /m calculate # 源码+汇编混合显示
```

### 寄存器操作
```bash
(gdb) info registers           # 查看所有寄存器
(gdb) info all-registers       # 包含浮点/向量寄存器
(gdb) print $rip               # 查看特定寄存器
(gdb) set $rax=0x10            # 修改寄存器值
```

### 执行控制
```bash
(gdb) si                       # 单步执行汇编指令
(gdb) ni 3                     # 执行3条汇编指令
```

### 调用栈分析
```bash
(gdb) backtrace full           # 完整调用栈（含参数）
(gdb) frame 2                  # 切换到2号栈帧
(gdb) info frame               # 当前栈帧信息
```

---

## 内存操作完全指南

### 内存查看命令
```bash
x/[数量][格式][单位] [地址]
```

**格式说明：**
| 格式 | 说明         | 示例                      |
| ---- | ------------ | ------------------------- |
| `x`  | 十六进制     | `x/xw` → 0xdeadbeef       |
| `d`  | 有符号十进制 | `x/dw` → -559038737       |
| `u`  | 无符号十进制 | `x/uw` → 3735928559       |
| `o`  | 八进制       | `x/ow` → 033653337357     |
| `t`  | 二进制       | `x/tw` → 11011110...      |
| `f`  | 浮点数       | `x/f` → 1.2345            |
| `a`  | 地址         | `x/a` → 0x400000 <_start> |
| `i`  | 指令         | `x/i` → mov eax,0x1       |
| `c`  | 字符         | `x/c` → 'A'               |
| `s`  | 字符串       | `x/s` → "Hello"           |

**单位说明：**
| 单位 | 大小  | 说明       |
| ---- | ----- | ---------- |
| `b`  | 1字节 | Byte       |
| `h`  | 2字节 | Halfword   |
| `w`  | 4字节 | Word       |
| `g`  | 8字节 | Giant word |

### 内存操作示例
```bash
(gdb) x/16xb buffer            # 16字节十六进制查看
(gdb) x/4gf &dbl_array         # 4个双精度浮点数
(gdb) x/10i $pc                # 10条指令反汇编
(gdb) x/s 0x4006b4             # 查看字符串
(gdb) x/20cb str_ptr           # 20个字符查看
```

### 高级内存操作
```bash
# 内存搜索
(gdb) find 0x400000, 0x401000, "secret"

# 内存修改
(gdb) set {int}0x7fffffffdc=42

# 内存区域比较
(gdb) compare-memory addr1 addr2 length

# 内存转储
(gdb) dump binary memory dump.bin 0x400000 0x401000
```

### 内存断点
```bash
(gdb) watch *0x7fffffffdc      # 硬件断点
(gdb) awatch *ptr              # 读写监控
(gdb) rwatch -l *ptr           # 指针指向内存监控
```

---

## 实例演示

### 示例程序
```c
// demo.c
#include <stdio.h>
#include <string.h>

struct Data {
    int id;
    char name[20];
    float score;
};

int calculate(int n) {
    int sum = 0;
    for(int i=0; i<=n; i++) {
        sum += i;
    }
    return sum;
}

int main() {
    // 基本类型
    int count = 10;
    char *str = "Debug Example";
    float values[3] = {1.1, 2.2, 3.3};
    
    // 结构体
    struct Data student = {101, "Alice", 92.5};
    
    // 指针
    int *ptr = &count;
    
    // 计算
    int result = calculate(count);
    printf("Result: %d\n", result);
    
    return 0;
}
```

### 调试流程

1. **编译程序**
```bash
gcc -g demo.c -o demo
```

2. **启动调试**
```bash
gdb ./demo
```

3. **设置断点**
```bash
(gdb) break main
(gdb) break calculate
(gdb) break 8 if i==5
```

4. **运行程序**
```bash
(gdb) run
```

5. **查看数据**
```bash
(gdb) print count              # 基本变量
(gdb) p values                 # 数组
(gdb) p/x &student             # 结构体地址
(gdb) x/20xb &student          # 结构体原始内存
```

6. **汇编级调试**
```bash
(gdb) disassemble calculate    # 反汇编函数
(gdb) ni 5                     # 执行5条指令
(gdb) info registers           # 查看寄存器状态
```

7. **内存操作**
```bash
(gdb) x/3fw values             # 查看浮点数组
(gdb) x/s str                  # 查看字符串
(gdb) watch sum                # 监控变量变化
```

8. **修改与继续**
```bash
(gdb) set var count=20         # 修改输入值
(gdb) set $eax=100             # 修改返回值
(gdb) continue                 # 继续执行
```

9. **高级调试**
```bash
(gdb) disassemble /m main      # 混合源码和汇编
(gdb) tbreak printf            # 临时断点
(gdb) commands                 # 自动命令序列
> backtrace
> info args
> continue
> end
```

### 综合调试技巧

1. **条件断点与命令**
```bash
(gdb) break 10 if sum > 100
(gdb) commands
  printf "i=%d, sum=%d\n", i, sum
  continue
end
```

2. **内存分析**
```bash
(gdb) p/x &count
$1 = 0x7fffffffdc
(gdb) x/wx 0x7fffffffdc        # 十六进制查看
(gdb) x/dw 0x7fffffffdc        # 十进制查看
(gdb) set {int}0x7fffffffdc=42 # 修改内存
```

3. **调用栈分析**
```bash
(gdb) bt full                  # 完整调用栈
(gdb) frame 1                  # 切换到调用帧
(gdb) info locals              # 查看局部变量
```

4. **指令级调试**
```bash
(gdb) layout asm               # 汇编窗口
(gdb) layout regs              # 寄存器窗口
(gdb) si                       # 单指令执行
(gdb) x/10i $pc                # 查看后续指令
```

5. **程序状态保存**
```bash
(gdb) checkpoint               # 创建检查点
(gdb) restart ckpt_id          # 恢复检查点
(gdb) generate-core-file       # 生成核心转储
```

---

## 总结图表

### GDB 调试命令速查表

| 类别     | 常用命令                           | 功能说明           |
| -------- | ---------------------------------- | ------------------ |
| **基础** | `run`, `quit`, `list`              | 启动/退出/查看源码 |
| **断点** | `break`, `watch`, `tbreak`         | 断点设置与管理     |
| **数据** | `print`, `x`, `ptype`              | 变量查看与修改     |
| **执行** | `step`, `next`, `continue`         | 程序流程控制       |
| **汇编** | `disassemble`, `si`, `ni`          | 指令级调试         |
| **内存** | `x/格式`, `set {type}addr`         | 内存操作           |
| **栈**   | `backtrace`, `frame`, `info frame` | 调用栈分析         |
| **信息** | `info`, `help`                     | 状态查看与帮助     |

### 内存查看格式速查

| 命令   | 示例            | 输出说明                |
| ------ | --------------- | ----------------------- |
| `x/xw` | `x/xw 0x400000` | 十六进制字 (0xdeadbeef) |
| `x/dw` | `x/dw &count`   | 十进制有符号整数        |
| `x/tw` | `x/tw &flags`   | 二进制表示              |
| `x/f`  | `x/f &pi`       | 浮点数值                |
| `x/i`  | `x/i $pc`       | 汇编指令                |
| `x/s`  | `x/s 0x4006b4`  | ASCII字符串             |
| `x/c`  | `x/10cb buffer` | 字符序列                |
| `x/a`  | `x/a func_ptr`  | 地址和符号              |



在实际调试中结合使用`help [command]`查看详细帮助信息。