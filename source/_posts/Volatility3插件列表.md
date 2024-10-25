---
layout: post
title: Volatility 3 插件列表
date: 2024-10-25 9:10:00
# 标签
tags: 
    - CTF
    - 取证
# 分类
categories: 
    # 主类
    - CTF
    # 子类
    - 取证
# 文章页头图
cover: "images/Volatility/gw.png"
#banner: "图片链接"
# 文章摘要
excerpt: "基本的Volatility 3 插件列表翻译"
license: all_rights_reserved
---

# Volatility 3 插件列表

{% notel blue 提示 %}
Volatility 3 工具中可用插件的列表
{% endnotel %}


{% tabs First unique name %}
<!-- tab 通用插件 -->
**通用插件**

- **banners.Banners**: 尝试识别镜像中的潜在 Linux 横幅信息。
- **configwriter.ConfigWriter**: 运行自动魔法并打印和输出配置到输出目录。
- **frameworkinfo.FrameworkInfo**: 列出 Volatility 框架的各种模块化组件。
- **isfinfo.IsfInfo**: 确定当前可用 ISF 文件或特定文件的信息。
- **layerwriter.LayerWriter**: 运行自动魔法并写出由堆叠器生成的主要层。
- **timeliner.Timeliner**: 运行所有相关插件提供时间相关信息并按时间顺序排序结果。
- **vmscan.Vmscan**: 扫描 Intel VT-d 结构并生成虚拟机的配置。
<!-- endtab -->

<!-- tab Linux 插件 -->
**Linux 插件**

- **linux.bash.Bash**: 恢复内存中的 bash 命令历史。
- **linux.capabilities.Capabilities**: 列出进程的能力。
- **linux.check_afinfo.Check_afinfo**: 验证网络协议的操作函数指针。
- **linux.check_creds.Check_creds**: 检查是否有进程共享凭据结构。
- **linux.check_idt.Check_idt**: 检查中断描述符表（IDT）是否被修改。
- **linux.check_modules.Check_modules**: 将模块列表与 sysfs 信息进行比较（如果可用）。
- **linux.check_syscall.Check_syscall**: 检查系统调用表是否有钩子。
- **linux.elfs.Elfs**: 列出所有进程的内存映射 ELF 文件。
- **linux.envars.Envars**: 列出带有环境变量的进程。
- **linux.iomem.IOMem**: 生成类似于运行系统上的 /proc/iomem 的输出。
- **linux.keyboard_notifiers.Keyboard_notifiers**: 解析键盘通知程序调用链。
- **linux.kmsg.Kmsg**: 内核日志缓冲区读取器。
- **linux.library_list.LibraryList**: 枚举加载到进程中的库。
- **linux.lsmod.Lsmod**: 列出加载的内核模块。
- **linux.lsof.Lsof**: 列出所有进程的内存映射。
- **linux.malfind.Malfind**: 列出可能包含注入代码的进程内存范围。
- **linux.mountinfo.MountInfo**: 列出进程挂载命名空间中的挂载点。
- **linux.netfilter.Netfilter**: 列出 Netfilter 钩子。
- **linux.proc.Maps**: 列出所有进程的内存映射。
- **linux.psaux.PsAux**: 列出带有命令行参数的进程。
- **linux.pslist.PsList**: 列出特定 Linux 内存镜像中的进程。
- **linux.psscan.PsScan**: 扫描特定 Linux 镜像中的进程。
- **linux.pstree.PsTree**: 基于父进程 ID 以树状列出进程。
- **linux.sockstat.Sockstat**: 列出所有进程的网络连接。
- **linux.tty_check.tty_check**: 检查 tty 设备是否有钩子。
- **linux.vmayarascan.VmaYaraScan**: 使用 yara 扫描所有任务的虚拟内存区域。
<!-- endtab -->

<!-- tab Mac 插件 -->
**Mac 插件**

- **mac.bash.Bash**: 恢复内存中的 bash 命令历史。
- **mac.check_syscall.Check_syscall**: 检查系统调用表是否有钩子。
- **mac.check_sysctl.Check_sysctl**: 检查 sysctl 处理程序是否有钩子。
- **mac.check_trap_table.Check_trap_table**: 检查 mach 陷阱表是否有钩子。
- **mac.dmesg.Dmesg**: 打印内核日志缓冲区。
- **mac.ifconfig.Ifconfig**: 列出所有设备的网络接口信息。
- **mac.kauth_listeners.Kauth_listeners**: 列出 kauth 监听器及其状态。
- **mac.kauth_scopes.Kauth_scopes**: 列出 kauth 范围及其状态。
- **mac.kevents.Kevents**: 列出进程注册的事件处理程序。
- **mac.list_files.List_Files**: 列出所有进程的打开文件描述符。
- **mac.lsmod.Lsmod**: 列出加载的内核模块。
- **mac.lsof.Lsof**: 列出所有进程的打开文件描述符。
- **mac.malfind.Malfind**: 列出可能包含注入代码的进程内存范围。
- **mac.mount.Mount**: 生成类似于 Mac 系统上 `mount` 命令的输出。
- **mac.netstat.Netstat**: 列出所有进程的网络连接。
- **mac.proc_maps.Maps**: 列出可能包含注入代码的进程内存范围。
- **mac.psaux.Psaux**: 恢复程序的命令行参数。
- **mac.pslist.PsList**: 列出特定 Mac 内存镜像中的进程。
- **mac.pstree.PsTree**: 基于父进程 ID 以树状列出进程。
- **mac.socket_filters.Socket_filters**: 枚举内核套接字过滤器。
- **mac.timers.Timers**: 检查恶意的内核计时器。
- **mac.trustedbsd.Trustedbsd**: 检查恶意的 TrustedBSD 模块。
- **mac.vfsevents.VFSevents**: 列出过滤文件系统事件的进程。
<!-- endtab -->

<!-- tab Windows 插件 -->
**Windows 插件**

- **windows.bigpools.BigPools**: 列出大页池。
- **windows.cachedump.Cachedump**: 从内存中转储 LSA 秘密。
- **windows.callbacks.Callbacks**: 列出内核回调和通知例程。
- **windows.cmdline.CmdLine**: 列出进程命令行参数。
- **windows.crashinfo.Crashinfo**: 列出 Windows 崩溃转储的信息。
- **windows.devicetree.DeviceTree**: 列出基于驱动程序和附加设备的树结构。
- **windows.dlllist.DllList**: 列出特定 Windows 内存镜像中的加载模块。
- **windows.driverirp.DriverIrp**: 列出特定 Windows 内存镜像中的驱动程序 IRP。
- **windows.drivermodule.DriverModule**: 确定是否有加载的驱动程序被 rootkit 隐藏。
- **windows.driverscan.DriverScan**: 扫描特定 Windows 镜像中的驱动程序。
- **windows.dumpfiles.DumpFiles**: 从 Windows 内存样本中转储缓存的文件内容。
- **windows.envars.Envars**: 显示进程环境变量。
- **windows.filescan.FileScan**: 扫描特定 Windows 内存镜像中的文件对象。
- **windows.getservicesids.GetServiceSIDs**: 列出进程令牌的 SIDs。
- **windows.getsids.GetSIDs**: 打印每个进程的 SID。
- **windows.handles.Handles**: 列出进程打开的句柄。
- **windows.hashdump.Hashdump**: 从内存中转储用户哈希。
- **windows.hollowprocesses.HollowProcesses**: 列出空洞化进程。
- **windows.iat.IAT**: 提取导入地址表以列出程序使用的 API。
- **windows.info.Info**: 显示被分析的内存样本的操作系统和内核详细信息。
- **windows.joblinks.JobLinks**: 打印进程作业链接信息。
- **windows.kpcrs.KPCRs**: 打印每个处理器的 KPCR 结构。
- **windows.ldrmodules.LdrModules**: 列出特定 Windows 内存镜像中的加载模块。
- **windows.lsadump.Lsadump**: 从内存中转储 LSA 秘密。
- **windows.malfind.Malfind**: 列出可能包含注入代码的进程内存范围。
- **windows.mbrscan.MBRScan**: 扫描并解析潜在的主引导记录（MBR）。
- **windows.memmap.Memmap**: 打印内存映射。
- **windows.mftscan.ADS**: 扫描备用数据流（ADS）。
- **windows.mftscan.MFTScan**: 扫描特定 Windows 内存镜像中的 MFT 文件对象。
- **windows.modscan.ModScan**: 扫描特定 Windows 镜像中的模块。
- **windows.modules.Modules**: 列出加载的内核模块。
- **windows.mutantscan.MutantScan**: 扫描特定 Windows 内存镜像中的互斥体。
- **windows.netscan.NetScan**: 扫描特定 Windows 内存镜像中的网络对象。
- **windows.netstat.NetStat**: 遍历特定 Windows 内存镜像中的网络跟踪结构。
- **windows.pedump.PEDump**: 从特定地址空间中的特定地址提取 PE 文件。
- **windows.poolscanner.PoolScanner**: 通用池扫描插件。
- **windows.privileges.Privs**: 列出进程令牌权限。
- **windows.processghosting.ProcessGhosting**: 列出设置了 DeletePending 位或 FILE_OBJECT 设置为 0 的进程。
- **windows.pslist.PsList**: 列出特定 Windows 内存镜像中的进程。
- **windows.psscan.PsScan**: 扫描特定 Windows 镜像中的进程。
- **windows.pstree.PsTree**: 基于父进程 ID 以树状列出进程。
- **windows.psxview.PsXView**: 使用四种方法列出所有进程，可能有助于识别隐藏的进程。
- **windows.registry.certificates.Certificates**: 列出注册表的证书存储中的证书。
- **windows.registry.getcellroutine.GetCellRoutine**: 报告具有被钩住的 GetCellRoutine 处理程序的注册表配置单元。
- **windows.registry.hivelist.HiveList**: 列出特定内存镜像中的注册表配置单元。
- **windows.registry.hivescan.HiveScan**: 扫描特定 Windows 镜像中的注册表配置单元。
- **windows.registry.printkey.PrintKey**: 列出特定配置单元或特定键值下的注册表键。
- **windows.registry.userassist.UserAssist**: 打印 UserAssist 注册表键及信息。
- **windows.sessions.Sessions**: 列出从环境变量中提取的会话信息。
- **windows.shimcachemem.ShimcacheMem**: 从 ahcache.sys AVL 树中读取 Shimcache 条目。
- **windows.skeleton_key_check.Skeleton_Key_Check**: 查找 Skeleton Key 恶意软件的迹象。
- **windows.ssdt.SSDT**: 列出系统调用表。
- **windows.statistics.Statistics**: 列出内存空间的统计信息。
- **windows.strings.Strings**: 读取 strings 命令的输出并指示每个字符串属于哪个进程。
- **windows.suspicious_threads.SupsiciousThreads**: 列出可疑的用户态进程线程。
- **windows.svcdiff.SvcDiff**: 比较通过列表遍历和扫描找到的服务以发现 rootkit。
- **windows.svclist.SvcList**: 列出 services.exe 的双向链表中的服务。
- **windows.svcscan.SvcScan**: 扫描 Windows 服务。
- **windows.symlinkscan.SymlinkScan**: 扫描特定 Windows
<!-- endtab -->

{% endtabs %}
