---
title: 缩放 HPC 应用程序-Azure 虚拟机 |Microsoft Docs
description: 了解如何缩放 Azure Vm 上的 HPC 应用程序。
author: vermagit
ms.service: virtual-machines
ms.topic: article
ms.date: 05/15/2019
ms.author: amverma
ms.reviewer: cynthn
ms.openlocfilehash: 2c20fcf2624c1f620b68b46e56e0602f15041348
ms.sourcegitcommit: 83610f637914f09d2a87b98ae7a6ae92122a02f1
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/13/2020
ms.locfileid: "91993822"
---
# <a name="scaling-hpc-applications"></a>缩放 HPC 应用程序

Azure 上的 HPC 应用程序的最佳向上扩展和向外扩展性能需要对特定工作负荷进行性能优化和优化试验。 本部分和 VM 系列特定页面提供了有关扩展应用程序的一般指南。

## <a name="compiling-applications"></a>编译应用程序

尽管不是必需的，但编译具有相应优化标志的应用程序可以在 HB-ACCT-WC 和 HC Vm 上提供最佳的向上缩放性能。

### <a name="amd-optimizing-cc-compiler"></a>AMD 优化 C/c + + 编译器

AMD 优化 C/c + + 编译器 (AOCC) 编译器系统提供高级别的高级优化、多线程和处理器支持，其中包括全局优化、矢量化、过程间分析、循环转换和代码生成。 AOCC 编译器二进制文件适用于) 版本2.17 及更高版本的、具有 GNU C 库 (glibc 的 Linux 系统。 编译器套件由 C/c + + 编译器 (clang) ，Fortran 编译器 (FLANG) ，并使用 Fortran 前端到 Clang (龙蛋。

### <a name="clang"></a>Clang

Clang 是 C、c + + 和目标 C 编译器，处理预处理、分析、优化、代码生成、程序集和链接。 Clang 支持  `-march=znver1` 标志，为基于 AMD Zen 的 x86 体系结构启用最佳代码生成和优化。

### <a name="flang"></a>FLANG

FLANG 编译器是 AOCC suite 的最新补充 (增加了) 2018 年4月的版本，并且当前处于预发布版本，供开发人员下载和测试。 基于 Fortran 2008，AMD 扩展了 GitHub 版本的 FLANG (https://github.com/flang-compiler/flang) 。 FLANG 编译器支持所有 Clang 编译器选项和多个特定于 FLANG 的编译器选项。

### <a name="dragonegg"></a>DragonEgg

DragonEgg 是一个 gcc 插件，它将 GCC 的优化器和代码生成器替换为 LLVM 项目中的项目。 AOCC 随附的 DragonEgg 适用于 gcc 1.x. x，已经针对 x86-32/x86-64 目标进行了测试，并已成功在各种 Linux 平台上使用。

GFortran 是 Fortran 程序的实际前端，负责预处理、分析和语义分析生成 GCC GIMPLE 中间表示形式 (IR) 。 DragonEgg 是一个 GNU 插件，可插入 GFortran 编译流中。 它实现 GNU 插件 API。 借助插件体系结构，DragonEgg 成为了编译器驱动程序，用于驱动编译的不同阶段。  完成下载和安装说明后，可以使用以下命令调用龙蛋状物： 

```bash
$ gfortran [gFortran flags] 
   -fplugin=/path/AOCC-1.2-Compiler/AOCC-1.2-     
   FortranPlugin/dragonegg.so [plugin optimization flags]     
   -c xyz.f90 $ clang -O3 -lgfortran -o xyz xyz.o $./xyz
```
   
### <a name="pgi-compiler"></a>PGI 编译器
PGI 社区版。 17已确认使用 AMD EPYC。 流的 PGI 编译版本提供平台的全部内存带宽。 较新的社区版 18.10 (11 月 2018) 同样适用。 下面是通过 Intel 编译器优化编译器的示例 CLI：

```bash
pgcc $(OPTIMIZATIONS_PGI) $(STACK) -DSTREAM_ARRAY_SIZE=800000000 stream.c -o stream.pgi
```

### <a name="intel-compiler"></a>Intel 编译器
Intel 编译器版本。 18已确认使用 AMD EPYC。 下面是通过 Intel 编译器优化编译器的示例 CLI。

```bash
icc -o stream.intel stream.c -DSTATIC -DSTREAM_ARRAY_SIZE=800000000 -mcmodel=large -shared-intel -Ofast –qopenmp
```

### <a name="gcc-compiler"></a>GCC 编译器 
对于 HPC，AMD 建议采用 GCC 编译器7.3 或更高版本。 不建议使用早期版本，如 RHEL/CentOS 7.4 随附的4.8.5。 对于 HPL.DAT、HPCG 和 DGEMM 测试，GCC 7.3 和更新版本将提供明显更高的性能。

```bash
gcc $(OPTIMIZATIONS) $(OMP) $(STACK) $(STREAM_PARAMETERS) stream.c -o stream.gcc
```

## <a name="scaling-applications"></a>缩放应用程序 

以下建议适用于最佳应用程序缩放效率、性能和一致性：

* 使用顺序固定方法将进程固定到核心0-59， (与自动平衡方法) 不同。 
* 由 Numa/Core/HwThread 绑定比默认绑定更好。
* 对于 (OpenMP + MPI) 的混合并行应用程序，请使用4个线程，每个 CCX 1 个 MPI 排名。
* 对于纯 MPI 应用程序，请针对每个 CCX 试验 1-4 MPI 排名，以获得最佳性能。
* 对于内存带宽有极大敏感性的某些应用程序，可以从每个 CCX 使用减少的内核数中获益。 对于这些应用程序，每个 CCX 使用3或2个核心可能会降低内存带宽争用，并产生更高的实际性能或更高的可伸缩性。 特别是，MPI Allreduce 可以从此获得。
* 对于规模更大的运行，建议使用 UD 或混合 RC + UD 传输。 许多 MPI 库/运行时库在内部执行此操作 (例如 UCX 或 MVAPICH2) 。 检查传输配置的大规模运行。

## <a name="next-steps"></a>后续步骤

了解有关 Azure 上的 [HPC](/azure/architecture/topics/high-performance-computing/) 的详细信息。
