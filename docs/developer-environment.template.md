# 开发者硬件环境

> **用途**：planner 在起草项目 PLAN.md 第 0 节（环境契约）时必须先读此文件，
> 确保 task 对 GPU 显存、内存、CPU 的需求不超出可用硬件。
> builder / reviewer 也应了解此信息，以便判断"能不能在本机跑"。
>
> **使用说明**：将本文件复制为 `docs/developer-environment.md` 并填入你的硬件信息。
> `developer-environment.md` 在 `.gitignore` 中，不会被提交——你的硬件信息不会泄露到公开仓库。

---

## 本地开发机

| 项目 | 规格 |
|---|---|
| OS | **<YOUR-OS>**（如 WSL2 Ubuntu 24.04 / macOS 15 / Windows 11） |
| CPU | **<YOUR-CPU>**（如 AMD Ryzen 9 5950X 16 核 32 线程） |
| RAM | **<YOUR-RAM>**（如 DDR4 3200MHz 16GB × 4 = 64GB） |
| GPU | **<YOUR-GPU>**（如 NVIDIA GeForce RTX 4070 Ti 12GB GDDR6X） |
| CUDA | <YOUR-CUDA-STATUS>（如"通过 WSL2 直通，支持 CUDA / cuDNN / PyTorch GPU"） |

### GPU 能力评估

根据你的 GPU 显存大小，明确标注典型适用场景。以下以 12GB 显存为例：

- ✅ 7B 参数模型（fp16 ≈ 14GB，需 QLoRA 4-bit ≈ 6-8GB）
- ✅ LoRA / QLoRA 微调（7B 模型 4-bit ≈ 6-8GB 显存占用）
- ✅ Stable Diffusion 1.5 / SDXL 推理与 DreamBooth 微调
- ✅ vLLM 部署 7B-INT4 量化模型
- ⚠️ 7B 模型 fp16 全量训练 — 显存紧张，需 gradient checkpointing + micro-batch
- ❌ 13B+ 模型 fp16 推理（需 26GB+ 显存）
- ❌ 全量 SFT 微调 7B 模型（需 ~60GB+ 显存）

> **硬件约束是硬性的**：PLAN.md 中标注的 GPU 需求若超出你的本地显存，必须在 PLAN 中说明
> "不可在本地完成，需租用云 GPU"并提供替代方案（如 QLoRA 降级、Colab T4 等）。

---

## 云算力（可选）

当项目对 GPU 有更高要求（如全量微调、多 GPU 分布式训练、70B 模型推理）时，
可以租用云 GPU 服务器（Linux 环境）。

| 场景 | 典型方案 | 预估成本 |
|---|---|---|
| 小规模训练 / 微调 | AutoDL / Colab Pro（A100 40GB） | ¥3-10/小时 |
| 中等规模训练 | AutoDL / Lambda Labs（A100 80GB / H100） | ¥10-30/小时 |
| 全量 SFT 7B | 1× A100 80GB | ¥10-15/小时 |
| 蒸馏 teacher 推理 | 1× A100 40GB（加载 7B teacher + 缓存 logits） | ¥3-8/小时 |

> 云服务器统一为 **Linux** 环境（Ubuntu 20.04/22.04），通过 SSH 访问。
> 需要在 PLAN.md 中额外说明云环境的 conda/pip/CUDA 路径。

---

## 对 agent 的要求

1. **planner**：读本文后，在 PLAN.md 第 0 节中明确标注：
   - 本项目的 GPU 需求（显存下限）
   - 是否可在本地完成（Yes / No / Partial）
   - 如不可本地完成，是否需要租云 GPU（给出推荐配置和预估费用）
2. **builder**：启动时检查 PLAN 第 0 节的 GPU 需求，如不可本地完成则提醒用户先租机器。
3. **reviewer**：独立验证时，如果 PLAN 标注"可本地完成"但你的复现遇到 OOM，这是一条 must-fix。
