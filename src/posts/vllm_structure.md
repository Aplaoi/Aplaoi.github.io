---
category: 博客
tags: 
    - vllm
    - AI
---

# vLLM 代码结构解析（更新中）

这几天在研究 [vLLM](https://github.com/vllm-project/vllm) 代码，因为我的研究方向是大模型推理优化.vLLM 是一个高效的大模型推理架构,它的代码结构比较复杂，所以我花了一些时间来梳理它的代码结构。本文将分享我对 vLLM 代码结构的理解。（同时吐槽一下官方文档写的太杂了，而且没有代码结构的介绍，导致二次开发难度较大qwq）

----

## 代码结构概览

首先这是 vLLM 的代码结构树（只看 vllm 模块）：

```bash
vllm/               
├── assets/             # 处理视频、音频等多模态数据的模块
├── attention/          # 注意力机制相关的实现
├── benchmarks/         # 性能基准测试相关代码
├── compilation/        # 推理模块编译相关代码
├── config/             # 配置文件和参数管理  
├── device_allocator/   # GPU 和 CPU 资源分配
├── distributed/        # 多设备下分布式通信
├── engine/             # 推理引擎核心代码（重要）
├── entrypoints/        # 入口点，包括离线和OpenAI API接口（重要）
├── executor/           # 利用Ray进行分布式推理
├── inputs/             # 输入数据处理，包括Prompt等
├── logging_utils/      # 日志记录和调试工具
├── lora/               # LoRA 相关代码
├── model_executor/     # 模型加载（重要）
├── multimodal/         # 多模态模型相关代码
├── platforms/          # 不同平台的适配代码
├── plugins/            # 包括 LoRA Resolver, io processor 等插件
├── profiler/           # 性能分析工具
├── ray/                # Ray 相关集成
├── reasoning/          # 各种模型的推理parser
├── third_party/        # nvml三方库
├── transformers_utils/ # Huggingface transformers 相关工具
├── triton_utils/       # Triton 导入相关工具
├── usage/              # 使用统计和分析
├── utils/              # 工具库，如DeepGEMM, FlashInfer等
├── v1/                 # v1 版本代码
└── vllm_flash_attn/    

```

## 技术栈

### MLA

vLLM 使用 MLA (Multi-head Latent Attention) 来优化注意力机制的计算效率。MLA 通过将注意力计算分解为多个头部，并在每个头部上进行并行计算，从而提高了整体的计算速度和效率。