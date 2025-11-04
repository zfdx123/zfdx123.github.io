---
layout: post
title: 找回Vmware的TMP密码
date: 2025-11-4 14:37:00
# 标签
tags: 
    - 虚拟机
    - Vmware
# 分类
categories: 
    # 主类
    - 虚拟机
# 文章页头图
cover: "images/vmware/vmware.png"
#banner: "图片链接"
# 文章摘要
excerpt: "打开powershell输入...."
license: all_rights_reserved
---

# 🔐 找回Vmware的TPM密码完整指南

## 📋 准备工作

在开始之前，请确保：
- 您有管理员权限的Windows账户
- 知道虚拟机文件(.vmx)的位置
- PowerShell可以正常运行

## 🚀 操作步骤

### 步骤1：加载Credential读取模块

打开**PowerShell**，复制并执行以下代码：

```powershell
Add-Type @"
using System;
using System.Runtime.InteropServices;

public class Win32Cred
{
    [StructLayout(LayoutKind.Sequential, CharSet = CharSet.Unicode)]
    public struct CREDENTIAL
    {
        public int Flags;
        public int Type;
        public IntPtr TargetName;
        public IntPtr Comment;
        public System.Runtime.InteropServices.ComTypes.FILETIME LastWritten;
        public int CredentialBlobSize;
        public IntPtr CredentialBlob;
        public int Persist;
        public int AttributeCount;
        public IntPtr Attributes;
        public IntPtr TargetAlias;
        public IntPtr UserName;
    }

    [DllImport("advapi32.dll", CharSet = CharSet.Unicode, SetLastError = true)]
    public static extern bool CredReadW(string target, int type, int reservedFlag, out IntPtr credentialPtr);

    [DllImport("advapi32.dll", SetLastError = true)]
    public static extern void CredFree(IntPtr buffer);

    public static string CredRead(string targetName, int type = 1)
    {
        IntPtr credPtr;
        if (CredReadW(targetName, type, 0, out credPtr))
        {
            CREDENTIAL cred = (CREDENTIAL)Marshal.PtrToStructure(credPtr, typeof(CREDENTIAL));
            string pass = Marshal.PtrToStringAnsi(cred.CredentialBlob, cred.CredentialBlobSize);
            CredFree(credPtr);
            return pass;
        }
        throw new System.ComponentModel.Win32Exception(Marshal.GetLastWin32Error());
    }
}
"@
```

**执行成功后，您将看到命令行提示符返回，没有错误信息。**

---

### 步骤2：查找虚拟机GUID

1. **找到虚拟机配置文件**
   - 导航到您的虚拟机存储目录
   - 找到扩展名为 `.vmx` 的配置文件

2. **编辑虚拟机配置文件**
   - 右键点击 `.vmx` 文件
   - 选择"用记事本打开"或您喜欢的文本编辑器

3. **搜索GUID字符串**
   - 在文件中搜索 `encryptedVM.guid`
   - 您将看到类似这样的行：
     ```
     encryptedVM.guid = "{833AB4F5-587E-4B11-9260-4DB13742FA7F}"
     ```

![查找GUID示意图](images/vmware/vmx1.png)
![GUID位置示意图](images/vmware/vmx2.png)

---

### 步骤3：获取TPM密码

在**同一个PowerShell窗口**中，执行以下命令：

```powershell
# 将大括号内的GUID替换为您在步骤2中找到的实际GUID
[Win32Cred]::CredRead("{833AB4F5-587E-4B11-9260-4DB13742FA7F}")
```

{% note warning %}
**重要提示：**
- 确保替换 `{833AB4F5-587E-4B11-9260-4DB13742FA7F}` 为您自己的GUID
- 包括大括号 `{}`
- 在同一个PowerShell会话中执行此命令
{% endnote %}

![密码获取结果](images/vmware/pass.png)
