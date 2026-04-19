# 基于WSL环境的神经网络加速运算环境

## 概述

本项目旨在包含一块nVidia 4060Ti(16G)显卡的Windows主机上，通过WSL来进行大语言模型的推理和训练。


## 环境初始化

> 输入：硬件安装和环境初始化
>
> 输出：完成环境初始化，能够运行 *Hello, Xio~*
>
> 要完成的事儿：必要驱动安装


#### checkpoint: 显卡驱动

- 确认WSL版本
```PowerShell
PS C:\Users\Xio> wsl --status
默认分发: Ubuntu-24.04
默认版本: 2
```

- 确认 GPU 型号、显存、驱动版本、CUDA 版本
```shell
$ nvidia-smi
Sun Apr 19 19:47:13 2026
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 595.58.02              Driver Version: 595.97         CUDA Version: 13.2     |
+-----------------------------------------+------------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|                                         |                        |               MIG M. |
|=========================================+========================+======================|
|   0  NVIDIA GeForce RTX 4060 Ti     On  |   00000000:01:00.0  On |                  N/A |
|  0%   31C    P8             12W /  165W |     648MiB /  16380MiB |     15%      Default |
|                                         |                        |                  N/A |
+-----------------------------------------+------------------------+----------------------+

+-----------------------------------------------------------------------------------------+
| Processes:                                                                              |
|  GPU   GI   CI              PID   Type   Process name                        GPU Memory |
|        ID   ID                                                               Usage      |
|=========================================================================================|
|  No running processes found                                                             |
+-----------------------------------------------------------------------------------------+
```

检查 GPU 设备节点
`ls /dev/nvidia*`
实际输出异常 TODO

- 确认 CUDA 编译器

```shell
$ nvcc --version
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2023 NVIDIA Corporation
Built on Fri_Jan__6_16:45:21_PST_2023
Cuda compilation tools, release 12.0, V12.0.140
Build cuda_12.0.r12.0/compiler.32267302_0
```
安装 CUDA
`$ sudo apt install nvidia-cuda-toolkit`

- 将PyTorch安装到Python虚拟环境

  创建虚拟环境bash运行 `python3 -m venv llm-env`
  激活环境 `source llm-env/bin/activate`
  安装PyTorch(耗时) `pip3 install torch torchvision`

## LLM选型

16GB 显存（VRAM）是本地跑 LLM 的黄金入门配置，4-bit 量化后可流畅跑 7B–20B 级模型，部分 MoE / 稀疏模型甚至能跑到 35B。

### 优先推荐：Qwen3-14B
Qwen3-14B / Qwen3.5-14B（中文首选）
量化：4-bit（GGUF Q4_K_M / AWQ）
显存：约 10–12GB
速度：50–80 tokens/s（RTX 4080/4090）
优势：中文极强、长上下文、代码 / 数学 / 对话全能
工具：Ollama、LM Studio、llama.cpp、vLLM

### 部署工具：Ollama
- Ollama：最简单，ollama run qwen3:14b
- LM Studio：可视化，一键下载 / 加载 / 对话
- llama.cpp：极致性能，支持 GGUF
- vLLM：高吞吐，适合批量 / API 服务



Ollama 一键运行命令清单

```shell
ollama run qwen3.5:14b
```


### 环境搭建



安装[ollama](https://ollama.com/)（耗时较长）

```shell
curl -fsSL https://ollama.com/install.sh | sh
```

一键安装对应的LLM，

#### checkpoint: PyTorch安装验证
```shell
python3 -c "
import torch
print('CUDA 可用:', torch.cuda.is_available())
print('GPU 型号:', torch.cuda.get_device_name(0))
print('CUDA 版本:', torch.version.cuda)
# 测试显存分配
x = torch.randn(1000, 1000).cuda()
print('显存分配成功')
"
```
