---
layout: post
title:  "BIOS Exception 分析流程"
date:   2022-05-20 09:10:00 +0800
categories: BIOS
tags: bios
---

**注：因 BIN 文件未保证编译一致性，此步骤定位的指令不一定准确。需要用当前编译环境再编一版测试版辅助定位问题后继续分析。**

### 一、收集错误日志
```bash
e:\BIOS\MdeModulePkg\Core\Dxe\Hand\DriverSupport.c(182): CR has Bad Signature
!!!! X64 Exception Type - 0D(#GP - General Protection)  CPU Apic ID - 00000000 !!!!
ExceptionData - 0000000000000000
RIP  - 000000006C686FFC, CS  - 0000000000000038, RFLAGS - 0000000000010216
RAX  - 000901000004092A, RCX - 0000000000000000, RDX - 00000000000003F8
RBX  - 0000000000000000, RSP - 000000006C680AC0, RBP - 0000000062048AD8
RSI  - 0009010000040932, RDI - 0000000062048AA0
R8   - 0000000000000000, R9  - 000000006C680A2F, R10 - 000000000000000F
R11  - 000000000000007F, R12 - 0000000080000000, R13 - 000000006C69DBD0
R14  - 0000000000000000, R15 - 0000000066E65E70
DS   - 0000000000000030, ES  - 0000000000000030, FS  - 0000000000000030
GS   - 0000000000000030, SS  - 0000000000000030
CR0  - 0000000080010013, CR2 - 0000000000000000, CR3 - 000000006C201000
CR4  - 0000000000000668, CR8 - 0000000000000000
DR0  - 0000000000000000, DR1 - 0000000000000000, DR2 - 0000000000000000
DR3  - 0000000000000000, DR6 - 00000000FFFF0FF0, DR7 - 0000000000000400
GDTR - 000000006BC89A98 0000000000000047, LDTR - 0000000000000000
IDTR - 000000006495B018 0000000000000FFF,   TR - 0000000000000000
FXSAVE_STATE - 000000006C680720
!!!! Find image based on IP(0x6C686FFC) e:\BIOS\Build\WilsonCity\DEBUG_VS2015\X64\MdeModulePkg\Core\Dxe\DxeMain\DEBUG\DxeCore.pdb (ImageBase=000000006C681000, EntryPoint=000000006C681C60) !!!!
```

### 二、找到导致 Exception 的源码

#### 1. 定位模块
根据 log 内容，可推断 Exception 发生在 DxeCore 模块中

#### 2. 定位指令码在模块内的 Offset
导致发生 Exception 的指令在模块内的偏移 = RIP - ImageBase = 0x5FFC

#### 3. 生成汇编及Map文件
- 在 DxeCore 模块的 inf 文件中增加 BuildOptions 以保留汇编&map文件
` *_*_*_CC_FLAGS = /FACS /Zd /mapinf:lines`
- 编译源码

#### 4. 定位模块中的 Object
查看 "Build\WilsonCity\DEBUG_VS2015\X64\MdeModulePkg\Core\Dxe\DxeMain\DEBUG\DxeCore.map" 如下：
```
Address             Publics by Value               Rva+Base             Lib:Object
.                   .                              .                    .
.                   .                              .                    .
.                   .                              .                    .
0001:00005b0c       CoreConnectController          0000000000005dcc f   DxeCore:DriverSupport.obj
0001:00005eb4       AddSortedDriverBindingProtocol 0000000000006174 f   DxeCore:DriverSupport.obj
.                   .                              .                    .
.                   .                              .                    .
.                   .                              .                    .
```
根据步骤 2 中计算得出的错误指令偏移为 0x5FFC，结合 DxeCore.map Rva+Base 字段可以推断出，导致 Exception 的指令在 DriverSupport.obj

#### 5. 定位指令码
用二进制方式打开 "Build\WilsonCity\DEBUG_VS2015\X64\MdeModulePkg\Core\Dxe\DxeMain\DEBUG\DxeCore.efi"，根据 2 中计算得出的错误指令偏移为 0x5FFC，可以找到发生问题的指令(不定长指令)：
```
Offset      Val

00005FE0    00 48 8B 4C 24 30 48 8B  C7 48 83 C0 40 48 8B 30
00005FF0    48 3B F0 74 51 48 8B E8  48 8D 46 F8 48 81 38 70
```

#### 6. 定位汇编及 C 源码
因步骤 5 中，已经推断出导致 Exception 的指令在 DriverSupport.obj 中，打开 **Build\WilsonCity\DEBUG_VS2015\X64\MdeModulePkg\Core\Dxe\DxeMain\DriverSupport.asm 文件**，在文件中直接搜索步骤 5 中的指令码，搜索结果如下：
```
	; 182  :         OpenData = CR (ProtLink, OPEN_PROTOCOL_DATA, Link, OPEN_PROTOCOL_DATA_SIGNATURE);
  0022c	48 8d 46 f8	 lea	 rax, QWORD PTR [rsi-8]
  00230	48 81 38 70 6f
	64 6c		 cmp	 QWORD PTR [rax], 1818521456 ; 6c646f70H
  00237	74 20		 je	 SHORT $LN48@CoreConnec
  00239	4c 8d 05 00 00
	00 00		 lea	 r8, OFFSET FLAT:??_C@_0BF@NDBIKIKC@CR?5has?5Bad?5Signature?$AA@
  00240	ba b6 00 00 00	 mov	 edx, 182		; 000000b6H
  00245	48 8d 0d 00 00
	00 00		 lea	 rcx, OFFSET FLAT:??_C@_0EB@GFHBICFB@e?3?2m6?2ibios?9m6?2isbios?2MdeModuleP@
  0024c	e8 00 00 00 00	 call	 DebugAssert
  00251	48 8b 4c 24 30	 mov	 rcx, QWORD PTR ChildHandleCount$1$[rsp]
  00256	48 8b c6	 mov	 rax, rsi
```
定位至指令`00230	48 81 38 70 6f 64 6c		 cmp	 QWORD PTR [rax], 1818521456 ; 6c646f70H`
此指令从 RAX 所存的地址开始 8 字节存储单元中取出值与 6c646f70H 做 cmp，此指令导致 Excepiton，大概率为 RAX 中存的地址异常。再查看 Exception 信息：
```
!!!! X64 Exception Type - 0D(#GP - General Protection)  CPU Apic ID - 00000000 !!!!
ExceptionData - 0000000000000000
RIP  - 000000006C686FFC, CS  - 0000000000000038, RFLAGS - 0000000000010216
RAX  - 000901000004092A, RCX - 0000000000000000, RDX - 00000000000003F8
```
RAX 的值为 000901000004092A，此地址明显异常，机器没有这么大的内存地址
由此可推断，此 Excepiton 为异常访存导致。
根据上下文，找到 DriverSupport.c 中对应的源码段为：
```
//
// Count ControllerHandle's children
//
 for (Link = Handle->Protocols.ForwardLink, ChildHandleCount = 0; Link != &Handle->Protocols; Link = Link->ForwardLink) {
   Prot = CR(Link, PROTOCOL_INTERFACE, Link, PROTOCOL_INTERFACE_SIGNATURE);
   for (ProtLink = Prot->OpenList.ForwardLink;
       ProtLink != &Prot->OpenList;
       ProtLink = ProtLink->ForwardLink) {
     OpenData = CR (ProtLink, OPEN_PROTOCOL_DATA, Link, OPEN_PROTOCOL_DATA_SIGNATURE);
     if ((OpenData->Attributes & EFI_OPEN_PROTOCOL_BY_CHILD_CONTROLLER) != 0) {
       ChildHandleCount++;
     }
   }
 }
```
**注：因 BIN 文件未保证编译一致性，此步骤定位的指令不一定准确，但根据上下文判断，导致 Exception 的指令应该就在附近，很有可能就是链表中的 Node 被异常修改后未从链表中删除导致。需要用当前编译环境再编一版测试版用于辅助定位问题后继续分析。**