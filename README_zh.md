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
ollama run qwen:14b
```

安装到虚拟环境中
```
(llm-env) xio@DESKTOP-F3HCGEM:~$ ollama run qwen:14b
pulling manifest
pulling de0334402b97: 100% ▕███████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████▏ 8.2 GB
pulling 7c7b8e244f6a: 100% ▕███████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████▏ 6.9 KB
pulling 1da0581fd4ce: 100% ▕███████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████▏  130 B
pulling f02dd72bb242: 100% ▕███████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████▏   59 B
pulling 007d4e6a46af: 100% ▕███████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████▏  484 B
verifying sha256 digest
writing manifest
success
>>>
```

此时已可以进行简单问答：
```shell
>>> 介绍你自己
你好，我是一名来自阿里云的超大规模语言模型。我被训练来回答各种问题、进行创作、提供建议等，尽可能地帮助用户解决问题。请随时向我提问，我会尽力提供支持。

>>> 你会解析excel表格吗，或者你知道如何解析？
作为一款语言模型，我没有直接操作文件的功能，包括解析Excel表格。不过我可以提供一些基本的指导：

1. **编程语言**：使用Python、Java或VBA等语言，可以利用pandas库（Python）或OpenOffice API（Java/VBA）来读取和解析Excel文件。

2. **APIs**：如果你需要频繁访问Excel中的数据，可能需要考虑使用像Aspose.Cells这样的商业软件的API，或者使用Google Sheets或Microsoft Office 365的API。

请根据你的需求和技术背景选择最适合的方法。

>>> 
```

同步使用 `nvidia-smi` 指令可以查看GPU 内存的使用情况。
```shell
xio@DESKTOP-F3HCGEM:/mnt/c/Users/Xio$ nvidia-smi
Mon Apr 20 09:45:36 2026
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 595.58.02              Driver Version: 595.97         CUDA Version: 13.2     |
+-----------------------------------------+------------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|                                         |                        |               MIG M. |
|=========================================+========================+======================|
|   0  NVIDIA GeForce RTX 4060 Ti     On  |   00000000:01:00.0  On |                  N/A |
|  0%   33C    P3             21W /  165W |   11550MiB /  16380MiB |      3%      Default |
|                                         |                        |                  N/A |
+-----------------------------------------+------------------------+----------------------+

+-----------------------------------------------------------------------------------------+
| Processes:                                                                              |
|  GPU   GI   CI              PID   Type   Process name                        GPU Memory |
|        ID   ID                                                               Usage      |
|=========================================================================================|
|    0   N/A  N/A            2170      C   /ollama                               N/A      |
+-----------------------------------------------------------------------------------------+
```

### 环境搭建



安装[ollama](https://ollama.com/)（耗时较长）

```shell
curl -fsSL https://ollama.com/install.sh | sh
```

输出
```
$ curl -fsSL https://ollama.com/install.sh | sh
>>> Cleaning up old version at /usr/local/lib/ollama
[sudo] password for xio:
>>> Installing ollama to /usr/local
>>> Downloading ollama-linux-amd64.tar.zst
######################################################################## 100.0%
>>> Creating ollama user...
>>> Adding ollama user to render group...
>>> Adding ollama user to video group...
>>> Adding current user to ollama group...
>>> Creating ollama systemd service...
>>> Enabling and starting ollama service...
Created symlink /etc/systemd/system/default.target.wants/ollama.service → /etc/systemd/system/ollama.service.
>>> Nvidia GPU detected.
>>> The Ollama API is now available at 127.0.0.1:11434.
>>> Install complete. Run "ollama" from the command line.
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


## 参考

### WSL存储空间迁移

```shell
xio@DESKTOP-F3HCGEM:/mnt/c/Users/Xio$ df -H
Filesystem      Size  Used Avail Use% Mounted on
none            4.2G     0  4.2G   0% /usr/lib/modules/6.6.87.2-microsoft-standard-WSL2
none            4.2G  4.1k  4.2G   1% /mnt/wsl
drivers         214G  207G  7.4G  97% /usr/lib/wsl/drivers
/dev/sdd        1.1T   37G  990G   4% /
none            4.2G   82k  4.2G   1% /mnt/wslg
none            4.2G     0  4.2G   0% /usr/lib/wsl/lib
rootfs          4.2G  2.8M  4.1G   1% /init
none            4.2G  566k  4.2G   1% /run
none            4.2G     0  4.2G   0% /run/lock
none            4.2G     0  4.2G   0% /run/shm
none            4.2G   78k  4.2G   1% /mnt/wslg/versions.txt
none            
4.2G   78k  4.2G   1% /mnt/wslg/doc
C:\             214G  207G  7.4G  97% /mnt/c
D:\             810G  203G  608G  26% /mnt/d
E:\             2.1T  169G  1.9T   9% /mnt/e
F:\             501G  371G  130G  75% /mnt/f
K:\             4.1T  850G  3.2T  22% /mnt/k
tmpfs           822M   21k  822M   1% /run/user/1000
```
可以看到 C 盘已经不堪重负，需要迁移到其他空间。
WSL支持一键迁移，指令如下：

```PowerShell
PS C:\Users\Xio> wsl -l -v
  NAME            STATE           VERSION
* Ubuntu-24.04    Running         2
PS C:\Users\Xio> wsl --shutdown
PS C:\Users\Xio> wsl --list
适用于 Linux 的 Windows 子系统分发:
Ubuntu-24.04 (默认值)
PS C:\Users\Xio> wsl --manage Ubuntu-24.04 --move D:\WSL\Ubuntu-24.04
操作成功完成。
```