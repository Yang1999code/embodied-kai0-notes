# 官方流水线精简

对照仓库：https://github.com/OpenDriveLab/kai0

这是公开 README 和模块文档的压缩版，不是飞书内部 Pipeline 文档。

## 目录对应

| 目录 | 作用 |
| --- | --- |
| `src/openpi/` | 从 openpi 来的训练和推理底座 |
| `scripts/train.py` | JAX 全参微调 |
| `scripts/train_pytorch.py` | PyTorch 训练，Stage Advantage estimator 用这个 |
| `scripts/serve_policy.py` | GPU 上的 policy server |
| `model_arithmetic/` | 权重混合 |
| `stage_advantage/` | 阶段优势标注、离散化、AWBC |
| `train_deploy_alignment/` | 增强、DAgger、真机推理 |
| `setup/` | 相机支架和夹爪的 3D 打印件 |
| `docs/dataset.md` | 数据集结构 |

## 主链

```text
下载数据到 ./data
    -> 改 src/openpi/training/config.py 里的 repo_id 和 weight_loader
    -> compute_norm_states_fast.py
    -> train.py 全参微调 pi0.5
    -> model_arithmetic 混多个 checkpoint
    -> serve_policy.py + 工控机 client 真机推理
    -> 失败时 DAgger 收恢复轨迹，再训练
```

Stage Advantage 是旁路：

```text
人工写 stage_progress_gt
    -> train_pytorch.py 训 advantage estimator
    -> annotation/eval.py 打 absolute/relative advantage
    -> discretize_advantage.py 变成正负 task_index
    -> train.py 的 pi05_*_awbc 配置做优势加权行为克隆
```

## 常用命令

安装：

```bash
git clone --recurse-submodules git@github.com:OpenDriveLab/kai0.git
GIT_LFS_SKIP_SMUDGE=1 uv sync
GIT_LFS_SKIP_SMUDGE=1 uv pip install -e .
```

训练：

```bash
uv run python scripts/compute_norm_states_fast.py --config-name pi05_flatten_fold_normal
XLA_PYTHON_CLIENT_MEM_FRACTION=0.9 uv run scripts/train.py pi05_flatten_fold_normal --exp_name=flatten_fold_run1
```

混权重（inverse_loss，不必梯度）：

```bash
python model_arithmetic/dump_data.py --dataset pi05_hang_cloth --output hang_cloth_val.pkl
python model_arithmetic/arithmetic.py   --config pi05_hang_cloth   --data-path hang_cloth_val.pkl   --checkpoints /path/ckpt1 /path/ckpt2 /path/ckpt3   --output /path/mixed_ckpt   --optimize_method inverse_loss --use_gpu --gpu_ids "0"
```

真机推理是双机：GPU 主机跑 server，工控机跑 client。

## 计算需求

| 模式 | 显存 | 例子 |
| --- | --- | --- |
| 推理 | > 8 GB | RTX 4090 |
| LoRA 微调 | > 22.5 GB | 4090，官方写未充分测试 |
| 全参微调 | > 70 GB | A100 80GB / H100 |
