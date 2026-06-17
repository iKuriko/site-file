---
title: "Deepseek model deploy"
date: 2026-04-10T16:57:02+08:00
draft: true
tags:
  - Tools
description: 在 GB200 服务器上拉起 deepseek 大模型
---

安装虚拟环境以下载大模型镜像

```bash
apt update
```

```bash
apt install -y software-properties-common
```

```bash
add-apt-repository ppa:deadsnakes/ppa
```

```bash
apt install -y python3.10 python3.10-venv python3.10-dev python3.10-distutils
```

下载 [Deepseek大模型](<https://huggingface.co/deepseek-ai/DeepSeek-R1-0528/tree/main>) 到本地

```bash
python3.10 -m venv deepseek-env-py310
source deepseek-env-py310/bin/activate
```

```bash
pip install huggingface_hub
```

```bash
hf download deepseek-ai/DeepSeek-R1-0528 --local-dir ./deepseek-r1-0528
```

 

安装依赖以支持拉起GPU容器 

```bash
apt-get install -y nvidia-container-toolkit
```

```bash
nvidia-ctk runtime configure --runtime=docker
```

```bash
systemctl restart docker
```

 

拉起大模型

```bash
docker run -itd \
  --name vllm-gb200 \
  --gpus all \
  --ipc=host \
  --network host \
  -v /root/deepseek-r1-0528:/model/deepseek-r1-0528  \
  vllm/vllm-openai:latest \
  --model /models/deepseek-r1-0528 \
  --trust-remote-code \
  --dtype bfloat16 \
  --kv-cache-dtype fp8 \
  --tensor-parallel-size 4 \
  --gpu-memory-utilization 0.95 \
  --max-model-len 32768 \
  --max-num-seqs 128 \
  --enforce-eager --enable-auto-tool-choice  --tool-call-parser hermes
```



查看拉起进度

```bash
docker logs -f vllm-gb200
```



调用测试

```bash
curl http://localhost:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "/models/DeepSeek-R1-Distill-Llama-70B",
    "messages": [
      {"role": "user", "content": "介绍一下量子计算的基本原理。"}
    ],
    "max_tokens": 100,
    "temperature": 0.7
  }'
```