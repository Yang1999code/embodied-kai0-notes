# 来源和验证边界

生成日期：2026-09-28。

## 已核对

| 来源 | 地址 | 本仓库怎么用 |
| --- | --- | --- |
| 官方代码 | https://github.com/OpenDriveLab/kai0 | 模块、命令、硬件、许可证 |
| 论文 v3 | https://arxiv.org/abs/2602.09021 | 问题定义、三个模块、实验协议 |
| 论文 PDF | `papers/chi0-kai0-arXiv-2602.09021v3.pdf` | SHA256 `d472f231a0ec73b791ec3ca8b395ea72270bf939907be8b243d3e574974b63dd`，6 页 PDF |
| 项目页 | https://mmlab.hk/research/kai0 | 演示和宣传口径；不一致时以论文和仓库为准 |
| 数据集说明 | https://github.com/OpenDriveLab/kai0/blob/main/docs/dataset.md | 任务划分和小时数 |
| 硬件说明 | https://github.com/OpenDriveLab/kai0/blob/main/setup/README.md | Piper / ARX 布局 |
| Hugging Face 数据 | https://huggingface.co/datasets/OpenDriveLab-org/Kai0 | 确认存在；未下载 |

## 未读到 / 未做

- 飞书页 https://j17tak1mfe1.feishu.cn/wiki/Y2qrwsfYpigl36kpzNmcxXVznug ：Chrome 跳登录，本机飞书登录态无法自动取出正文。
- 没有下载数据集或 checkpoint。
- 没有在真机或 GPU 上复现训练和部署。
- 论文图里的具体成功率柱状数值没有逐根转录；正文只保留作者明确写出的相对提升口径。

## 不要混在一起的数字

1. 宣传实验：每任务约 20 小时演示 + 8 张 A100。
2. 开源数据集：base 约 134 小时，加上 DAgger 约 181 小时。
3. 论文图里的机器人叫两套双臂 ALOHA；开源 `setup/` 写的是 Agilex Piper 和 ARX X5。
4. `setup/README` 写推理机 `RTX 4090 (>= 48 GB VRAM)`，消费级 4090 实际是 24 GB，这条不能当硬规格。
