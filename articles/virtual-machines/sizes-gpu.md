---
title: Azure VM sizes - GPU | Microsoft Docs
description: Lists the different GPU optimized sizes available for virtual machines in Azure. Lists information about the number of vCPUs, data disks and NICs as well as storage throughput and network bandwidth for sizes in this series.
author: vikancha-MSFT
ms.service: virtual-machines
ms.subservice: sizes
ms.topic: conceptual
ms.date: 10/27/2022
ms.author: vikancha
---

# GPU optimized virtual machine sizes

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs :heavy_check_mark: Flexible scale sets :heavy_check_mark: Uniform scale sets

> [!TIP]
> Try the **[Virtual machines selector tool](https://aka.ms/vm-selector)** to find other sizes that best fit your workload.

GPU optimized VM sizes are specialized virtual machines available with single, multiple, or fractional GPUs. These sizes are designed for compute-intensive, graphics-intensive, and visualization workloads. This article provides information about the number and type of GPUs, vCPUs, data disks and NICs as well as storage throughput and network bandwidth for sizes in this series.

- The [NCv3-series](ncv3-series.md) and [NC T4_v3-series](nct4-v3-series.md) sizes are optimized for compute-intensive GPU-accelerated applications. Some examples are CUDA and OpenCL-based applications and simulations, AI, and Deep Learning. The NC T4 v3-series is focused on inference workloads featuring NVIDIA's Tesla T4 GPU and AMD EPYC2 Rome processor. The NCv3-series is focused on high-performance computing and AI workloads featuring NVIDIA’s Tesla V100 GPU.

- The [NC 100 v4-series](nc-a100-v4-series.md) sizes are focused on midrange AI training and batch inference workload. The NC A100 v4-series offers flexibility to select one, two, or four NVIDIA A100 80GB PCIe Tensor Core GPUs per VM to use the right-size GPU acceleration for your workload.

- The [ND A100 v4-series](nda100-v4-series.md) sizes are focused on scale-up and scale-out deep learning training and accelerated HPC applications. The ND A100 v4-series uses 8 NVIDIA A100 TensorCore GPUs, each available with a 200 Gigabit Mellanox InfiniBand HDR connection and 40 GB of GPU memory.

- [NGads V620-series](ngads-v-620-series.md) VM sizes are optimized for high performance, interactive gaming experiences hosted in Azure.  They're powered by AMD Radeon PRO V620 GPUs and AMD EPYC 7763 (Milan) CPUs.

- [NVv3-series](nvv3-series.md) and [NVads A10 v5-series](nva10v5-series.md) sizes are optimized and designed for remote visualization, streaming, gaming, encoding, and VDI scenarios using frameworks such as OpenGL and DirectX. These VMs are backed by NVIDIA Tesla M60 GPU (NVv3) and A10 GPUs(NVads A10 v5).

- [NVv4-series](nvv4-series.md) VM sizes optimized and designed for VDI and remote visualization. With partitioned GPUs, NVv4 offers the right size for workloads requiring smaller GPU resources. These VMs are backed by AMD Radeon Instinct MI25 GPU. NVv4 VMs currently support only Windows guest operating system.

- [NDm A100 v4-series](ndm-a100-v4-series.md) virtual machines are designed for high-end Deep Learning training and tightly coupled scale-up and scale-out HPC workloads. The NDm A100 v4 series starts with a single virtual machine (VM) and eight NVIDIA Ampere A100 80GB Tensor Core GPUs.


## Supported operating systems and drivers

To take advantage of the GPU capabilities of Azure N-series VMs, NVIDIA or AMD GPU drivers must be installed.

- For VMs backed by NVIDIA GPUs, the [NVIDIA GPU Driver Extension](./extensions/hpccompute-gpu-windows.md) installs appropriate NVIDIA CUDA or GRID drivers. Install or manage the extension using the Azure portal or tools such as Azure PowerShell or Azure Resource Manager templates. See the [NVIDIA GPU Driver Extension documentation](./extensions/hpccompute-gpu-windows.md) for supported operating systems and deployment steps. For general information about VM extensions, see [Azure virtual machine extensions and features](./extensions/overview.md).

   Alternatively, you may install NVIDIA GPU drivers manually. See [Install NVIDIA GPU drivers on N-series VMs running Windows](./windows/n-series-driver-setup.md) or [Install NVIDIA GPU drivers on N-series VMs running Linux](./linux/n-series-driver-setup.md) for supported operating systems, drivers, installation, and verification steps.

- For VMs backed by AMD GPUs, the [AMD GPU driver extension](./extensions/hpccompute-amd-gpu-windows.md) installs appropriate AMD drivers. Install or manage the extension using the Azure portal or tools such as Azure PowerShell or Azure Resource Manager templates. For general information about VM extensions, see [Azure virtual machine extensions and features](./extensions/overview.md).

   Alternatively, you may install AMD GPU drivers manually. See [Install AMD GPU drivers on N-series VMs running Windows](./windows/n-series-amd-driver-setup.md) for supported operating systems, drivers, installation, and verification steps.

## Deployment considerations

- For availability of N-series VMs, see [Products available by region](https://azure.microsoft.com/regions/services/).

- N-series VMs can only be deployed in the Resource Manager deployment model.

- N-series VMs differ in the type of Azure Storage they support for their disks. NC and NV VMs only support VM disks that are backed by Standard Disk Storage (HDD). All other GPU VMs support VM disks that are backed by Standard Disk Storage and Premium Disk Storage (SSD).

- If you want to deploy more than a few N-series VMs, consider a pay-as-you-go subscription or other purchase options. If you're using an [Azure free account](https://azure.microsoft.com/free/), you can use only a limited number of Azure compute cores.

- You might need to increase the cores quota (per region) in your Azure subscription, and increase the quota separately for each GPU VM family. To request a quota increase, [open an online customer support request](../azure-portal/supportability/how-to-create-azure-support-request.md) at no charge. Default limits may vary depending on your subscription category.

## Other sizes

- [General purpose](sizes-general.md)
- [Compute optimized](sizes-compute.md)
- [High performance compute](sizes-hpc.md)
- [Memory optimized](sizes-memory.md)
- [Storage optimized](sizes-storage.md)
- [Previous generations](sizes-previous-gen.md)

## Next steps

Learn more about how [Azure compute units (ACU)](acu.md) can help you compare compute performance across Azure SKUs.
