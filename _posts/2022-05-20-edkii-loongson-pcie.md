---
layout: post
title:  "EDKII PCIE (Loongson)"
date:   2022-05-20 09:15:00 +0800
categories: BIOS
tags: bios
---

## 一、 主要函数概述
### 1. 创建 Root Bridges (资源描述 & Access 接口实现)

```C
/**
  Return all the root bridge instances in an array.

  @param Count  Return the count of root bridge instances.

  @return All the root bridge instances in an array.
          The array should be passed into PciHostBridgeFreeRootBridges()
          when it's not used.
**/
PCI_ROOT_BRIDGE *
EFIAPI
PciHostBridgeGetRootBridges (
  UINTN *Count
  )
```

**Instance:** PciHostBridgeLib/PciHostBridgeLib.c

```C
/**
  Construct the Pci Root Bridge instance.
      
  @param Bridge            The root bridge instance.
      
  @return The pointer to PCI_ROOT_BRIDGE_INSTANCE just created
          or NULL if creation fails.
**/ 
PCI_ROOT_BRIDGE_INSTANCE *
CreateRootBridge (
  IN PCI_ROOT_BRIDGE       *Bridge
  )
```

**Instance:** PciHostBridgeDxe/PciRootBridgeIo.c  
**Access Configuration Space:**  
ootBridgeIoPciRead(Write) <- RootBridgeIoPciAccess <- PciSegmentRead(Write)Buffer <- PciSegmentRead(Write)8(16/32) <- PciRead(Write)8(16/32) <- 底层实现  
**Access IO/Mmio:**  
RootBridgeIoIo(Mem)Read(Write) <- mCpuIo->Io(Mem).Read(Write) <- CpuIo(Memory)ServiceRead(Write) <- Io(Mmio)Read(Write)8(16/32) <- 底层实现
 
### 2. 向 GCD 中注册 IO/MMIO 信息
通过 gDS 可调用相关接口. 
**Instance:** MdeModulePkg/Core/Dxe/Gcd/Gcd.c
 
### 3. 设备枚举,分配 BUS 号
```C
/**
  Enumerate PCI root bridge.

  @param PciResAlloc   Pointer to protocol instance of EFI_PCI_HOST_BRIDGE_RESOURCE_ALLOCATION_PROTOCOL.
  @param RootBridgeDev Instance of root bridge device.

  @retval EFI_SUCCESS  Successfully enumerated root bridge.
  @retval other        Failed to enumerate root bridge.
**/
EFI_STATUS
PciRootBridgeEnumerator (
  IN EFI_PCI_HOST_BRIDGE_RESOURCE_ALLOCATION_PROTOCOL  *PciResAlloc,
  IN PCI_IO_DEVICE                                     *RootBridgeDev
  )
```
**Instance:** PciBusDxe/PciEnumerator.c

```C
/**
  Scan pci bus and assign bus number to the given PCI bus system.

  @param  Bridge           Bridge device instance.
  @param  StartBusNumber   start point.
  @param  SubBusNumber     Point to sub bus number.
  @param  PaddedBusRange   Customized bus number.

  @retval EFI_SUCCESS      Successfully scanned and assigned bus number.
  @retval other            Some error occurred when scanning pci bus.

  @note   Feature flag PcdPciBusHotplugDeviceSupport determine whether need support hotplug.
**/
EFI_STATUS
PciScanBus (
  IN PCI_IO_DEVICE                      *Bridge,
  IN UINT8                              StartBusNumber,
  OUT UINT8                             *SubBusNumber,
  OUT UINT8                             *PaddedBusRange
  )
```

**Instance:** PciBusDxe/PciLib.c
 
### 4. 分配 IO/Mem 资源

```C
/**
  This routine is used to enumerate entire pci bus system
  in a given platform.

  @param Controller          Parent controller handle.
  @param HostBridgeHandle    Host bridge handle.

  @retval EFI_SUCCESS    PCI enumeration finished successfully.
  @retval other          Some error occurred when enumerating the pci bus system.
**/
EFI_STATUS
PciEnumerator (
  IN EFI_HANDLE                    Controller,
  IN EFI_HANDLE                    HostBridgeHandle
  )
```

**Instance:** PciBusDxe/PciEnumerator.c

```C
/**
  Submits the I/O and memory resource requirements for the specified PCI Host Bridge.
  
  @param PciResAlloc  Point to protocol instance of EFI_PCI_HOST_BRIDGE_RESOURCE_ALLOCATION_PROTOCOL.
  
  @retval EFI_SUCCESS           Successfully finished resource allocation.
  @retval EFI_NOT_FOUND         Cannot get root bridge instance.
  @retval EFI_OUT_OF_RESOURCES  Platform failed to program the resources if no hot plug supported.
  @retval other                 Some error occurred when allocating resources for the PCI Host Bridge.

  @note   Feature flag PcdPciBusHotplugDeviceSupport determine whether need support hotplug.
  
**/
EFI_STATUS 
PciHostBridgeResourceAllocator (
  IN EFI_PCI_HOST_BRIDGE_RESOURCE_ALLOCATION_PROTOCOL *PciResAlloc
  ) 
```

**Instance:** PciBusDxe/PciLib.c
```C
/** 
  This function is used to program the resource allocated
  for each resource node under specified bridge.
    
  @param Base     Base address of resource to be progammed.
  @param Bridge   PCI resource node for the bridge device.
  
  @retval EFI_SUCCESS            Successfully to program all resouces
                                 on given PCI bridge device.
  @retval EFI_OUT_OF_RESOURCES   Base is all one.

**/
EFI_STATUS
ProgramResource (
  IN UINT64            Base,
  IN PCI_RESOURCE_NODE *Bridge
  ) 
```

**Instance:** PciBusDxe/PciResourceSupport.c

## 二、 Host Bridges 初始化
### 1. 初始化 Root Bridges 实例基本信息

`RootBridges = PciHostBridgeGetRootBridges (&RootBridgeCount);`
获取 Root Bridge 数量，实例化 PCI_ROOT_BRIDGE 中的成员用以描述 Root Bridge 的资源, 以数组的形式返回

```C
typedef struct {
  UINT32                    Segment;
  UINT64                    Supports;
  UINT64                    Attributes;
  BOOLEAN                   DmaAbove4G;
  BOOLEAN                   NoExtendedConfigSpace;
  BOOLEAN                   ResourceAssigned;
  UINT64                    AllocationAttributes;
  PCI_ROOT_BRIDGE_APERTURE  Bus;
  PCI_ROOT_BRIDGE_APERTURE  Io;
  PCI_ROOT_BRIDGE_APERTURE  Mem;
  PCI_ROOT_BRIDGE_APERTURE  MemAbove4G;
  PCI_ROOT_BRIDGE_APERTURE  PMem;
  PCI_ROOT_BRIDGE_APERTURE  PMemAbove4G;
  EFI_DEVICE_PATH_PROTOCOL  *DevicePath;
} PCI_ROOT_BRIDGE;
```

**RootBridge.Segment (UINT32) :**
此成员用于描述本 RootBridge 所属的Segment。当 Platform 包含多个 RootBridge 时， 有两种方式来区分不同 Root Bridge 的 PCI 空间，第一种是多个 Root Bridge 共享 一组 Bus/Dev/Func 号；第二种是通过 Segment 来区分不同的Root Bridge， 每个 Root Bridge 都有自己独立一组 Bus/Dev/Func 号。当用第一种方式时，RootBridge.Segment 成员初始化为 0 即可。

![2-1-0](/assets/images/BIOS/PCIE-LOONGSON/pci-host-bus00.png)
![2-1-1](/assets/images/BIOS/PCIE-LOONGSON/pci-host-bus01.png)

**RootBridge.Support & RootBridge.Attributes (UINT64) :**
UEFI Spec 中定义的 Attribute 种类如下，Spec 中对每个 Attribute 都做了相应的描述。做龙芯项目的过程中未接触到相关内容，暂不清楚实际应用中的作用。

```C
#define EFI_PCI_ATTRIBUTE_ISA_MOTHERBOARD_IO          0x0001
#define EFI_PCI_ATTRIBUTE_ISA_IO                      0x0002
#define EFI_PCI_ATTRIBUTE_VGA_PALETTE_IO              0x0004
#define EFI_PCI_ATTRIBUTE_VGA_MEMORY                  0x0008
#define EFI_PCI_ATTRIBUTE_VGA_IO                      0x0010
#define EFI_PCI_ATTRIBUTE_IDE_PRIMARY_IO              0x0020
#define EFI_PCI_ATTRIBUTE_IDE_SECONDARY_IO            0x0040
#define EFI_PCI_ATTRIBUTE_MEMORY_WRITE_COMBINE        0x0080
#define EFI_PCI_ATTRIBUTE_MEMORY_CACHED               0x0800
#define EFI_PCI_ATTRIBUTE_MEMORY_DISABLE              0x1000
#define EFI_PCI_ATTRIBUTE_DUAL_ADDRESS_CYCLE          0x8000
#define EFI_PCI_ATTRIBUTE_ISA_IO_16                   0x10000
#define EFI_PCI_ATTRIBUTE_VGA_PALETTE_IO_16           0x20000
#define EFI_PCI_ATTRIBUTE_VGA_IO_16                   0x40000
```

**RootBridge.DmaAbove4G (BOOLEAN) :**
当设置为 TRUE  时表示 Root Bridge 支持 4GB 以上 DMA，否则表示不支持

**RootBridge.NoExtendedConfigSpace (BOOLEAN) :**
是否支持扩展类型的配置空间，配置为 FALSE 时表示支持扩展配置空间 (4096-byte)，否则表示只支持 256-byte 配置空间

**RootBridge.ResourceAssigned (BOOLEAN) :**
Root Bridge 的资源分配状态位，如果设置为 TRUE，则表示 Root Bridge 的 Bus/IO/MMIO 资源已分配

**RootBridge.AllocationAttributes (UINT64) :**
资源分配属性，描述如下

```C
/// If this bit is set, then the PCI Root Bridge does not
/// support separate windows for Non-prefetchable and Prefetchable
/// memory. A PCI bus driver needs to include requests for Prefetchable
/// memory in the Non-prefetchable memory pool.   
///
#define EFI_PCI_HOST_BRIDGE_COMBINE_MEM_PMEM  1   
  
///
/// If this bit is set, then the PCI Root Bridge supports
/// 64 bit memory windows.  If this bit is not set,
/// the PCI bus driver needs to include requests for 64 bit
/// memory address in the corresponding 32 bit memory pool.
///
#define EFI_PCI_HOST_BRIDGE_MEM64_DECODE   2
```

**RootBridge.Bus/Io/Mem/MemAbove4G/PMem/PMemAbove4G (PCI_ROOT_BRIDGE) :**

```C
//
// (Base > Limit) indicates an aperture is not available.
//
typedef struct {
  //  
  // Base and Limit are the device address instead of host address when
  // Translation is not zero
  //  
  UINT64 Base;
  UINT64 Limit;
  //  
  // According to UEFI 2.7, Device Address = Host Address + Translation,
  // so Translation = Device Address - Host Address.
  // On platforms where Translation is not zero, the subtraction is probably to
  // be performed with UINT64 wrap-around semantics, for we may translate an
  // above-4G host address into a below-4G device address for legacy PCIe device
  // compatibility.
  //  
  // NOTE: The alignment of Translation is required to be larger than any BAR
  // alignment in the same root bridge, so that the same alignment can be
  // applied to both device address and host address, which simplifies the                                                                                                                                         
  // situation and makes the current resource allocation code in generic PCI
  // host bridge driver still work.
  //  
  UINT64 Translation;
} PCI_ROOT_BRIDGE_APERTURE;
```

用以描述该 Root Bridge 可以使用的各种资源窗口, 如:

```C
RootBridges->Bus.Base  = 0;          //可用的最小 Bus 号
RootBridges->Bus.Limit = 127;        //可用的最大 Bus 号
RootBridges->Mem.Base  = 0x40000000; //可用的 Memory 空间基址
RootBridges->Mem.Limit = 0x80000000; //可用的 Memory 空间最高地址
```
**RootBridge.Devicepath (EFI_DEVICE_PATH_PROTOCOL *) :** 
DevicePath, UEFI 中用于描述设备路径的接口

### 2. 创建 Root Bridge Handle
#### 2.1 创建 Root Bridges 实例

`RootBridge = CreateRootBridge (&RootBridges[Index]);`
继承了 PCI_ROOT_BRIDGE 中基本信息的基础上, 增加了一些成员

```C
typedef struct {
  UINT32                            Signature;             //new
  LIST_ENTRY                        Link;                  //new
  EFI_HANDLE                        Handle;                //new
  UINT64                            AllocationAttributes;
  UINT64                            Attributes;
  UINT64                            Supports;
  PCI_RES_NODE                      ResAllocNode[TypeMax]; //new
  PCI_ROOT_BRIDGE_APERTURE          Bus;
  PCI_ROOT_BRIDGE_APERTURE          Io;
  PCI_ROOT_BRIDGE_APERTURE          Mem;
  PCI_ROOT_BRIDGE_APERTURE          PMem;
  PCI_ROOT_BRIDGE_APERTURE          MemAbove4G;
  PCI_ROOT_BRIDGE_APERTURE          PMemAbove4G;
  BOOLEAN                           DmaAbove4G;
  BOOLEAN                           NoExtendedConfigSpace;
  VOID                              *ConfigBuffer;         //new
  EFI_DEVICE_PATH_PROTOCOL          *DevicePath;
  CHAR16                            *DevicePathStr;        //new
  EFI_PCI_ROOT_BRIDGE_IO_PROTOCOL   RootBridgeIo;          //new

  BOOLEAN                           ResourceSubmitted;     //new
  LIST_ENTRY                        Maps;                  //new
} PCI_ROOT_BRIDGE_INSTANCE;
```
**RootBridge->Signature(UINT32) :**
实例的数字签名
**RootBridge->DevicePathStr(CHAR16 *) :**
通过 `ConvertDevicePathToText (Bridge->DevicePath, FALSE, FALSE))); `将 DevicePath 转换成的描述设备路径的字符串, 形式如下:
>PciRoot(0x0)
**RootBridge->ConfigBuffer(VOID *) :**
由多个 QWORD 地址空间资源描述符及一个结束标签构成, 用以表示 Root Bridge 地址空间资源配置

```C
RootBridge->ConfigBuffer = AllocatePool (
    TypeMax * sizeof (EFI_ACPI_ADDRESS_SPACE_DESCRIPTOR) + sizeof (EFI_ACPI_END_TAG_DESCRIPTOR)
    );
```

```C
typedef struct {
  UINT8   Desc;
  UINT16  Len;
  UINT8   ResType;
  UINT8   GenFlag;
  UINT8   SpecificFlag; 
  UINT64  AddrSpaceGranularity;
  UINT64  AddrRangeMin; 
  UINT64  AddrRangeMax;
  UINT64  AddrTranslationOffset;
  UINT64  AddrLen;
} EFI_ACPI_ADDRESS_SPACE_DESCRIPTOR; //地址空间描述符
```

```C
typedef struct {
  UINT8 Desc;
  UINT8 Checksum;
} EFI_ACPI_END_TAG_DESCRIPTOR;       //结束标签
```
**RootBridge->Maps(LIST_ENTRY) :**  
待补充...  
**RootBridge->ResAllocNode(PCI_RES_NODE *) :**  
待补充...  
**RootBridge->RootBridgeIo(EFI_PCI_ROOT_BRIDGE_IO_PROTOCOL) :**  
用来访问配置 PCI 控制器的接口实现

```C
///
/// Provides the basic Memory, I/O, PCI configuration, and DMA interfaces that are
/// used to abstract accesses to PCI controllers behind a PCI Root Bridge Controller.
///
struct _EFI_PCI_ROOT_BRIDGE_IO_PROTOCOL {
  /// 
  /// The EFI_HANDLE of the PCI Host Bridge of which this PCI Root Bridge is a member.
  /// 
  EFI_HANDLE                                      ParentHandle;
  EFI_PCI_ROOT_BRIDGE_IO_PROTOCOL_POLL_IO_MEM     PollMem;
  EFI_PCI_ROOT_BRIDGE_IO_PROTOCOL_POLL_IO_MEM     PollIo;
  EFI_PCI_ROOT_BRIDGE_IO_PROTOCOL_ACCESS          Mem;
  EFI_PCI_ROOT_BRIDGE_IO_PROTOCOL_ACCESS          Io;
  EFI_PCI_ROOT_BRIDGE_IO_PROTOCOL_ACCESS          Pci;
  EFI_PCI_ROOT_BRIDGE_IO_PROTOCOL_COPY_MEM        CopyMem;
  EFI_PCI_ROOT_BRIDGE_IO_PROTOCOL_MAP             Map;
  EFI_PCI_ROOT_BRIDGE_IO_PROTOCOL_UNMAP           Unmap;
  EFI_PCI_ROOT_BRIDGE_IO_PROTOCOL_ALLOCATE_BUFFER AllocateBuffer;
  EFI_PCI_ROOT_BRIDGE_IO_PROTOCOL_FREE_BUFFER     FreeBuffer;
  EFI_PCI_ROOT_BRIDGE_IO_PROTOCOL_FLUSH           Flush;
  EFI_PCI_ROOT_BRIDGE_IO_PROTOCOL_GET_ATTRIBUTES  GetAttributes;
  EFI_PCI_ROOT_BRIDGE_IO_PROTOCOL_SET_ATTRIBUTES  SetAttributes;
  EFI_PCI_ROOT_BRIDGE_IO_PROTOCOL_CONFIGURATION   Configuration;

  /// 
  /// The segment number that this PCI root bridge resides.
  /// 
  UINT32                                          SegmentNumber;
};
```