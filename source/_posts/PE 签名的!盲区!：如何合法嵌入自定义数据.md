---
updated: '2025-11-08 00:00:00'
categories: code
excerpt: 在 Windows 平台上，数字签名是验证软件可信度的重要机制。然而，在某些场景下，开发者需要向已签名的 PE 文件中嵌入额外信息（如配置数据、版本标识等），同时又不能破坏原有的数字签名。本文将介绍如何利用 PE 文件签名机制的特性，实现这一目标。
date: '2025-11-08 00:00:00'
tags:
  - C++
  - Python
urlname: write-info-to-pe-without-breaking-signature
title: PE 签名的盲区：如何合法嵌入自定义数据
---

# 概述


在 Windows 平台上，数字签名是验证软件可信度的重要机制。然而，在某些场景下，开发者需要向已签名的 PE 文件中嵌入额外信息（如配置数据、版本标识等），同时又不能破坏原有的数字签名。本文将介绍如何利用 PE 文件签名机制的特性，实现这一目标。


# 技术背景


## 数字签名的作用


数字签名为软件安全提供了三重保障：

1. **身份认证**：通过证书验证软件发布者的真实身份
2. **完整性验证**：确保文件在签名后未被篡改
3. **信任建立**：提升用户对软件的信任度，避免安全警告

## PE 文件签名机制


Windows PE 文件采用 Authenticode 签名机制，其核心流程包括：

1. **哈希计算**：对 PE 文件的特定部分计算哈希值（排除某些字段以允许签名嵌入）
2. **签名生成**：使用私钥对哈希值进行签名，生成 `WIN_CERTIFICATE` 结构
3. **签名嵌入**：将签名数据嵌入到 PE 文件的安全目录（Security Directory）中
4. **签名验证**：系统使用公钥验证签名，确保文件完整性

### 哈希计算的排除区域


关键在于：Authenticode 在计算文件哈希时，会**排除**以下三个部分：[[1]](https://www.symbolcrash.com/wp-content/uploads/2019/02/Authenticode_PE-1.pdf)[[2]](https://www.freebuf.com/articles/game/427201.html)

1. **PE Header 中的 CheckSum 字段**：该字段会因文件内容变化而改变
2. **Certificate Table Entry**：位于数据目录表的第 4 项（`IMAGE_DIRECTORY_ENTRY_SECURITY`），记录签名数据的位置和大小
3. **签名数据本身**：即 `WIN_CERTIFICATE` 结构及其内容

这种设计使得签名可以嵌入文件本身，而不会导致"签名包含自身"的循环依赖问题。


# 实现原理


## 方法一：在 WIN_CERTIFICATE 之后追加数据


由于签名验证时只会读取 `WIN_CERTIFICATE` 结构中 `dwLength` 字段指定的长度，因此在该结构之后追加的数据**不会被包含在哈希计算中**，从而不影响签名有效性。[[3]](https://juejin.cn/post/7491269083180531727)[[4]](https://www.reversinglabs.com/blog/tampering-with-signed-objects-without-breaking-the-integrity-seal)


### 实现要点

1. **修改 Security Directory Entry 的 Size 字段**
	- 将 `IMAGE_DIRECTORY_ENTRY_SECURITY.Size` 增加为：原签名长度 + 附加数据长度
	- 注意：附加数据大小需要填充到 8 字节对齐
2. **保持 WIN_CERTIFICATE 结构不变**
	- `dwLength` 字段保持原值，仍指向原始签名数据的长度
	- 这样签名验证时不会读取附加的数据
3. **安全性考量**
	- Windows 默认**不会**将这种附加数据视为签名失效
	- 微软曾提供注册表选项（`EnableCertPaddingCheck`）来检测这种行为，但该选项默认禁用以保持兼容性[[2]](https://www.freebuf.com/articles/game/427201.html)

### 示意图说明


```javascript
PE File Structure:
├─ DOS Header
├─ PE Header
├─ Section Headers
├─ Sections (.text, .data, etc.)
└─ Security Directory
   ├─ WIN_CERTIFICATE (签名数据)
   │  ├─ dwLength: 原始签名长度
   │  ├─ wRevision: WIN_CERT_REVISION_2_0
   │  ├─ wCertificateType: WIN_CERT_TYPE_PKCS_SIGNED_DATA
   │  └─ bCertificate: 签名内容
   └─ [附加数据区域] ← 可在此处添加自定义数据
```


## 方法二：利用 PKCS#7 的 unauthenticated 字段


PKCS#7 签名结构中包含一个 `unauthenticated` 属性字段，该字段**不参与签名计算**。虽然此字段原本用于存储时间戳，但实际上可以填写任意数据。[[5]](https://vcsjones.dev/authenticode-stuffing-tricks/)


知名软件如 Dropbox 的安装程序就采用了这种技术来嵌入配置信息。


# 代码示例


## 工具使用


使用 Didier Stevens 开发的 [`disitool.py`](http://disitool.py/) 工具可以方便地向 PE 文件注入数据：[[6]](https://blog.didierstevens.com/programs/disitool/)


```bash
python [disitool.py](http://disitool.py/) inject --paddata source.exe data.txt output.exe
```


## 读取附加数据的 C++ 实现


以下代码演示如何在运行时读取附加在 `WIN_CERTIFICATE` 之后的数据：


```c++
#include <windows.h>
#include <wintrust.h>
#include <stdio.h>

// 获取安全目录项（兼容 x86 和 x64）
BOOL GetSecurityDirectoryEntry(
    LPBYTE map,
    PIMAGE_DATA_DIRECTORY* securityDir,
    PDWORD securitySize)
{
    *securityDir = nullptr;
    *securitySize = 0;

    auto dosHeader = (PIMAGE_DOS_HEADER)map;
    if (dosHeader->e_magic != IMAGE_DOS_SIGNATURE)
        return FALSE;

    auto headers = (PIMAGE_NT_HEADERS)(map + dosHeader->e_lfanew);
    if (headers->Signature != IMAGE_NT_SIGNATURE)
        return FALSE;

    // 判断是 x86 还是 x64
    WORD magic = headers->OptionalHeader.Magic;

    if (magic == IMAGE_NT_OPTIONAL_HDR32_MAGIC)
    {
        // x86 架构
        auto headers32 = (PIMAGE_NT_HEADERS32)(map + dosHeader->e_lfanew);

        if (headers32->OptionalHeader.NumberOfRvaAndSizes > IMAGE_DIRECTORY_ENTRY_SECURITY)
        {
            *securityDir = &headers32->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_SECURITY];
            *securitySize = headers32->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_SECURITY].Size;
        }
    }
    else if (magic == IMAGE_NT_OPTIONAL_HDR64_MAGIC)
    {
        // x64 架构
        auto headers64 = (PIMAGE_NT_HEADERS64)(map + dosHeader->e_lfanew);

        if (headers64->OptionalHeader.NumberOfRvaAndSizes > IMAGE_DIRECTORY_ENTRY_SECURITY)
        {
            *securityDir = &headers64->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_SECURITY];
            *securitySize = headers64->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_SECURITY].Size;
        }
    }
    else
    {
        return FALSE;
    }

    // 验证安全目录是否有效
    if (*securityDir && (*securityDir)->VirtualAddress != 0 && (*securityDir)->Size != 0)
    {
        return TRUE;
    }

    return FALSE;
}

// 读取附加在签名后的数据
void ReadAuthenticodeTail(PCWSTR path, LPBYTE* tailData, LPDWORD tailSize)
{
    *tailData = nullptr;
    *tailSize = 0;
    PIMAGE_DATA_DIRECTORY securityDir = nullptr;
    DWORD securitySize = 0;

    // 打开文件并映射到内存
    auto file = CreateFile(path, GENERIC_READ, FILE_SHARE_READ, nullptr, OPEN_EXISTING, 0, nullptr);
    if (file == INVALID_HANDLE_VALUE) return;

    auto mapping = CreateFileMapping(file, nullptr, PAGE_READONLY, 0, 0, nullptr);
    if (!mapping) { 
        CloseHandle(file); 
        return; 
    }

    auto map = (LPBYTE)MapViewOfFile(mapping, FILE_MAP_READ, 0, 0, 0);
    if (!map) { 
        CloseHandle(mapping); 
        CloseHandle(file); 
        return; 
    }

    // 获取安全目录项
    if (GetSecurityDirectoryEntry(map, &securityDir, &securitySize))
    {
        // 注意：Certificate Table Entry 的 VirtualAddress 实际是文件偏移，不是 RVA
        auto va = securityDir->VirtualAddress;

        // 验证偏移是否在文件范围内
        if (va + securitySize > GetFileSize(file, nullptr))
        {
            wprintf(L"Security Directory extends beyond file size.\n");
        }
        else
        {
            auto cert = (LPWIN_CERTIFICATE)(map + va);

            wprintf(L"Revision: %u\n", cert->wRevision);
            wprintf(L"Certificate Type: %u\n", cert->wCertificateType);
            wprintf(L"Certificate Length: %u\n", cert->dwLength);
            wprintf(L"Size of Directory Entry: %u\n", securitySize);

            // 检查是否为 Authenticode 签名，且有附加数据
            if (cert->wRevision == WIN_CERT_REVISION_2_0 &&
                cert->wCertificateType == WIN_CERT_TYPE_PKCS_SIGNED_DATA &&
                cert->dwLength < securitySize)
            {
                // 读取签名后的附加数据
                *tailSize = securitySize - cert->dwLength;

                *tailData = new BYTE[*tailSize];

                if (*tailData)
                {
                    CopyMemory(*tailData, map + va + cert->dwLength, *tailSize);
                }
                else
                {
                    *tailSize = 0;
                    wprintf(L"Error: Memory allocation failed.\n");
                }
            }
        }
    }

    UnmapViewOfFile(map);
    CloseHandle(mapping);
    CloseHandle(file);
}

int main()
{
    LPBYTE tail;
    DWORD size;

    wchar_t path[MAX_PATH];
    GetModuleFileName(NULL, path, MAX_PATH);

    ReadAuthenticodeTail(path, &tail, &size);

    if (tail)
    {
        wprintf(L"Successfully read %u bytes of tail data.\n", size);
        
        // 在此处理读取的数据
        // ...
        
        delete[] tail;
    }
    else
    {
        wprintf(L"Did not find valid Authenticode tail data.\n");
    }

    return 0;
}
```


### 代码要点说明

1. **架构兼容性**：代码通过检查 `Magic` 字段来区分 x86 和 x64 PE 文件
2. **文件偏移 vs RVA**：`Certificate Table Entry` 的 `VirtualAddress` 是文件偏移，而非相对虚拟地址（RVA）
3. **数据提取**：通过比较 `securitySize` 和 `cert->dwLength` 来判断是否存在附加数据

# 安全性讨论


## CVE-2013-3900 漏洞


历史上，这种技术曾被恶意软件利用（CVE-2013-3900），攻击者可以向合法签名的文件中注入恶意代码而不破坏签名。[[2]](https://www.freebuf.com/articles/game/427201.html)


微软提供了注册表选项来检测这种行为：


```javascript
[HKEY_LOCAL_MACHINE\Software\Microsoft\Cryptography\Wintrust\Config]
"EnableCertPaddingCheck"=dword:1
```


但为了保持向后兼容性，该选项**默认禁用**。


## 合法使用场景


尽管存在安全风险，但这种技术也有合法用途：

- 嵌入每用户配置信息
- 添加部署标识或追踪信息
- 存储非关键的元数据

## 最佳实践建议

1. **仅用于非安全关键数据**：不要在附加数据中存储敏感信息
2. **额外完整性校验**：可以对附加数据单独进行签名或哈希验证
3. **文档化使用**：明确说明软件使用了这种技术及其用途
4. **考虑替代方案**：评估是否可以使用资源段或独立配置文件

# 参考资料


[PE 格式 - Win32 apps | Microsoft Learn](https://learn.microsoft.com/zh-cn/windows/win32/debug/pe-format)


[RFC 2315 - PKCS #7: Cryptographic Message Syntax Version 1.5](https://datatracker.ietf.org/doc/html/rfc2315)


[Authenticode 代码签名的注意事项 | Microsoft Learn](https://learn.microsoft.com/en-us/archive/blogs/ieinternals/caveats-for-authenticode-code-signing)


[在 PE 文件中嵌入数据同时保持数字签名完整及对应的检测方法 | ](https://crackme.net/articles/wincert/)[CrackMe.net](http://crackme.net/)


[Disitool | Didier Stevens](https://blog.didierstevens.com/programs/disitool/)

