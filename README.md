# χ₀ / kai0 精简学习笔记

个人对照笔记，不是官方仓库。

把三份材料压成一份能面试复述的说明：

1. 官方代码：[OpenDriveLab/kai0](https://github.com/OpenDriveLab/kai0)
2. 官方论文：[arXiv:2602.09021](https://arxiv.org/abs/2602.09021)
3. 指定飞书页：[wiki/Y2qrwsfYpigl36kpzNmcxXVznug](https://j17tak1mfe1.feishu.cn/wiki/Y2qrwsfYpigl36kpzNmcxXVznug)（当前读不到正文，见 [docs/feishu.md](docs/feishu.md)）

本仓库不收录数据集和 checkpoint。官方代码请直接看上游。本地 PDF 在 [papers/chi0-kai0-arXiv-2602.09021v3.pdf](papers/chi0-kai0-arXiv-2602.09021v3.pdf)。

## 一句话

kai0 不是新的 VLA 架构。它在 `π₀ / π₀.₅`（openpi）上全参微调，用三个模块把「人演示、模型偏好、真机执行」三条分布对齐，目标是把叠衣服、分拣、挂衣跑稳。

读音：χ₀ = kai0。

## 它在解决什么

长程真机衣服操作会滑、会皱、会遮挡。作者认为主瓶颈不是再堆数据和卡，而是这三条分布对不齐：

| 符号 | 含义 | 直观 |
| --- | --- | --- |
| `P_train` | 人演示 | 专家当时怎么做 |
| `Q_model` | 策略学到的动作偏好 | 模型觉得该怎么动 |
| `P_test` | 真机实际轨迹 | 延迟、摩擦、控制之后真正发生的事 |

```text
P_train  --Model Arithmetic-->  Q_model
P_train  --Train-Deploy Alignment-->  P_test
Q_model  --Stage Advantage-->  P_test
```

衣服任务被选中，是因为接触多、物体会变形，这三条错位会被放大。

## 三个模块

### 1. Model Arithmetic（模型算术）

不同衣服外观、不同初始状态各训一个 checkpoint，再在权重空间混合。不是 Mixture-of-Experts，也不是推理时多个模型投票。

代码：`model_arithmetic/`

方法：average、inverse_loss、gradient_descent、adaptive_gradient_descent、greedy、手动权重。

论文口径：用 DAgger 这类分布外数据定混合权重，比用训练域验证更稳；混合后的模型能超过「单子集最好」和「全数据一起训」。

### 2. Stage Advantage（阶段优势）

把长程任务拆成语义阶段，直接预测「这一步有没有推进当前阶段」，再二值化，做优势加权行为克隆（AWBC）。

代码：`stage_advantage/`

对比对象：`π*_0.6` 那种用价值差算 advantage 的做法。作者认为长程任务里价值差数值不稳。

缺口：阶段标注 `stage_progress_gt` 需要人工写，官方没给标注脚本。

### 3. Train-Deploy Alignment（训练-部署对齐）

三件事：

- 时空增强：抽帧变速、左右臂镜像
- Heuristic DAgger：策略在环，人只在失败附近纠错
- 动作块时间平滑，可叠加 RTC（实时动作块）

代码：`train_deploy_alignment/`

论文口径：Stage Advantage 更拉动单位时间完成量；TDA 更拉动成功率，但重试次数会升高。

## 三个任务

| 代码 / Task | 论文叫法 | 做什么 | 开源硬件 |
| --- | --- | --- | --- |
| FlattenFold / Task A | 展平 + 折叠 | 乱扔的 T 恤，180 秒内叠好放到桌心 | Agilex Piper + 3 路 RealSense D435 |
| TeeShirtSort / Task B | 条件分拣 | T 恤折叠码放；衬衫翻领后推到一侧 | 同上 |
| HangCloth / Task C | 挂衣 | 把衬衫套上衣架再挂杆 | ARX X5 + 3 路 D435 |

论文图写的是两套双臂 ALOHA 协作。开源 `setup/` 写的是 Piper / ARX。复现以仓库硬件说明为准。

## 作者给出的结果口径

这些是论文和 README 的写法，不是本仓库复现：

- 每任务约 20 小时演示，8 张 A100
- 成功率相对开源 `π₀.₅` 提升约 250%
- 可从任意初始状态连跑 24 小时
- 评估：每种衣服 10 次；看成功率、throughput、retry cost、分阶段得分
- GO-1、X-VLA、DexVLA：作者说在这套 20 小时数据上没跑出可用效果

开源数据集是另一组数字：base 约 15,997 条 / 134 小时，加上 DAgger 约 20,909 条 / 181 小时。不要把「实验用的 20 小时」和「开源全集」写成同一个数。

## 许可证

- 官方代码：Apache-2.0
- 数据和 checkpoint：CC BY-NC-SA 4.0，不能直接商用
- 本笔记：Apache-2.0

## 作者自己写的限制

- 没系统测微调后，预训练出来的通用能力还剩多少
- Model Arithmetic 目前是同任务子集混合，不是跨任务通用策略
- 数据质量还得靠完整训练或回放来验，缺便宜的预筛选指标

## 怎么往下读

1. 先读本页，建立「三条分布 + 三个模块 + 三个任务」。
2. 要跑命令，看 [docs/pipeline.md](docs/pipeline.md)。
3. 要对数字和来源，看 [docs/sources.md](docs/sources.md)。
4. 飞书正文还没进来，看 [docs/feishu.md](docs/feishu.md)。
5. 需要对照源码时，打开官方仓库，不要把本笔记当成实现。

## 明确没做的事

- 没复现训练
- 没上真机
- 没下载权重和数据
- 没读到指定飞书页正文
