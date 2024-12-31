## ADDG
## 内存标签技术
FEAT_MTE2  
FEAT_MTE2 是 ARMv8.5 及更高版本中引入的内存标记扩展（Memory Tagging Extension, MTE）的一个特性。MTE 提供了一种机制，通过在内存中附加标签来增强内存安全性，检测和防止内存错误，如缓冲区溢出和使用未初始化的内存。

FEAT_MTE2 是 MTE 的第二阶段扩展，进一步增强了内存标记功能。以下是 FEAT_MTE2 的一些关键特性：

标签存储：每个内存块（通常是 16 字节）都有一个关联的标签，标签存储在内存的高位地址中。
标签检查：在内存访问时，硬件会自动检查标签是否匹配。如果标签不匹配，会触发标签检查故障。
标签管理指令：提供了一组新的指令，用于设置、清除和检查内存标签。
增强的错误检测：通过标签机制，可以更有效地检测和防止内存错误，如缓冲区溢出、使用未初始化的内存等。
FEAT_MTE2 的主要目标是提高软件的内存安全性，减少内存相关的漏洞和错误。它在操作系统和应用程序中都可以使用，特别适用于需要高安全性和可靠性的场景。


在 ARMv8 的内存标记扩展（Memory Tagging Extension, MTE）中，标签存储区域的管理和位置是由硬件和操作系统共同决定的。标签存储区域既不是简单地位于栈空间或堆空间，也不是直接暴露给用户的逻辑地址或硬件实际地址。以下是一些关键点：

标签存储区域
独立的标签存储区域：

标签存储区域是独立于普通数据存储区域的。它们通常由硬件和操作系统管理，确保标签和数据的关联性。
逻辑地址与物理地址：

标签存储区域的地址映射通常是由硬件和操作系统管理的。对于应用程序来说，标签存储区域的地址是透明的，应用程序通过特定的指令和机制来访问和操作标签，而不需要直接处理标签存储区域的地址。
标签存储区域的管理可能涉及到虚拟地址（逻辑地址）和物理地址的转换，但这些细节通常由操作系统和硬件抽象出来。

在 ARMv8 的内存标记扩展（Memory Tagging Extension, MTE）中，内存标签的初始化和权限设置通常由操作系统和运行时环境管理。以下是一些常见的初始化和权限设置策略：

内存标签的初始化
默认标签值：

内存标签在分配时通常会被初始化为一个默认值。这个默认值可以是零，也可以是其他特定的值，具体取决于操作系统和运行时环境的实现。
例如，在某些实现中，内存标签可能被初始化为零，以表示未使用或未标记的内存区域。
标签分配策略：

操作系统和运行时环境可能会使用特定的策略来分配和管理标签。例如，可以使用循环分配、随机分配或基于上下文的分配策略来初始化标签。
读权限和写权限的初始化
默认权限设置：

内存区域的读写权限通常由操作系统的内存管理单元（MMU）和页表设置。在启用 MTE 的系统中，标签检查是额外的安全机制，读写权限的初始化和管理仍然遵循传统的内存管理机制。
默认情况下，内存区域的读写权限可能会被初始化为允许读写，具体取决于内存的用途和分配策略。
标签检查机制：

MTE 提供了额外的标签检查机制，用于在内存访问时验证标签是否匹配。如果标签不匹配，可以触发标签检查故障，从而增强内存安全性。
标签检查机制可以配置为在读访问、写访问或两者都进行检查。具体的配置取决于操作系统和应用程序的需求。

```C++
#include <iostream>
#include <cstdint>

// 假设我们有一个函数来初始化内存标签
void initialize_memory_tag(uintptr_t data_address, uint8_t tag) {
    // 使用 MTE 指令来设置标签
    // 具体实现依赖于硬件和操作系统支持
    asm volatile("STG %0, [%1]" : : "r"(tag), "r"(data_address));
}

// 假设我们有一个函数来设置内存区域的读写权限
void set_memory_permissions(uintptr_t data_address, size_t size, bool read, bool write) {
    // 使用操作系统提供的系统调用或库函数来设置权限
    // 具体实现依赖于操作系统支持
    // 例如，在 Linux 上可以使用 mprotect 函数
    int prot = 0;
    if (read) prot |= PROT_READ;
    if (write) prot |= PROT_WRITE;
    mprotect(reinterpret_cast<void*>(data_address), size, prot);
}

int main() {
    uintptr_t data_address = 0x1000;
    size_t size = 64; // 假设我们有 64 字节的内存区域

    // 初始化内存标签
    initialize_memory_tag(data_address, 0xA);

    // 设置内存区域的读写权限
    set_memory_permissions(data_address, size, true, true);

    // 其他代码逻辑...

    return 0;
}
```

## 指令意义
其主要目的就是设置内存的tag

伪代码：  

```
if !HaveMTEExt() then UNDEFINED;
integer d = UInt(Xd);
integer n = UInt(Xn);
bits(4) tag_offset = uimm4;
bits(64) offset = LSL(ZeroExtend(uimm6, 64), LOG2_TAG_GRANULE);
boolean ADD = TRUE;
bits(64) operand1 = if n == 31 then SP[] else X[n, 64];
bits(4) start_tag = AArch64.AllocationTagFromAddress(operand1);
bits(16) exclude = GCR_EL1.Exclude;
bits(64) result;
bits(4) rtag;

if AArch64.AllocationTagAccessIsEnabled(AccType_NORMAL) then
    rtag = AArch64.ChooseNonExcludedTag(start_tag, tag_offset, exclude);
else
    rtag = '0000';

if ADD then
    (result, -) = AddWithCarry(operand1, offset, '0');
else
    (result, -) = AddWithCarry(operand1, NOT(offset), '1');

result = AArch64.AddressWithAllocationTag(result, AccType_NORMAL, rtag);

if d == 31 then
    SP[] = result;
else
    X[d, 64] = result;
```