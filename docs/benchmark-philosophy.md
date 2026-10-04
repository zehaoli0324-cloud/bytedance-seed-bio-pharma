# Benchmark 思想：从论文分数到科研决策系统

本页总结 ByteDance Seed 生物项目现有 benchmark 的共同短板。这里讨论的是评测设计成熟度，不是对模型科学质量的否定；国外项目本身也并不都完善。

## 核心判断

目前很多项目把 benchmark 作为论文证据：准备一组测试集，运行若干指标，给出平均分和排行榜。更成熟的 benchmark 应该是一套科学测量和决策基础设施：在明确的任务分布、计算预算和不确定性条件下，判断哪个模型在真实科研决策中更可靠。

因此，缺少的不是更多单项指标，而是以下问题的系统回答：

> 模型在什么任务上会失效？它是否知道自己会失效？增加计算预算是否值得？评测结果能否改变下一轮实验或模型选择？

## 七个缺失的 benchmark 观念

### 1. Benchmark 首先要测泛化

固定测试集上的高平均分不能代表对新任务的泛化能力。生物项目至少需要区分：

- 时间切分后的新结构；
- 低同源 target；
- 长序列、抗体、配体、核酸等困难子集；
- 训练数据污染和结构泄漏；
- 分布外任务。

[PXMeter](https://github.com/bytedance/PXMeter) 已有 RecentPDB、低同源和专项数据，但数据污染审计、训练截止时间和泛化边界还应成为正式结果，而不是附加说明。

### 2. Benchmark 要评估决策质量

科研人员最终不是要一个平均 LDDT，而是要决定哪个结构可以交给下游、哪个 binder 值得合成、哪个候选值得做实验。评测因此应包括：

- top-1、top-k 和候选淘汰质量；
- 模型排序与真实外部指标的一致性；
- 不同 target 类型下的模型选择；
- 盲态候选的实际命中率。

平均结构分数高，但不能正确挑选候选，实际科研价值可能有限。

### 3. Benchmark 要验证 confidence 是否可信

模型 confidence 不能只作为输出字段，应该验证：

- confidence 和真实结构质量的相关性；
- 高 confidence 样本的实际成功概率；
- 不同 target 子集上的 calibration error；
- 多次采样是否稳定；
- confidence 是否只是模型内部的自洽分数。

对 Protenix/PXMeter，最有价值的补强不是再增加一个 RMSD 指标，而是比较 confidence 是否能可靠预测 DockQ、LDDT、ligand RMSD 和 PoseBusters 结果。

### 4. Benchmark 要报告质量与成本的权衡

真实使用还需要知道：

- GPU seconds 和峰值显存；
- MSA/模板搜索成本；
- 采样数量和收益曲线；
- 失败率；
- 每个有效候选的计算成本。

如果一个模型需要数十倍计算预算才略高于另一个模型，单一 accuracy 排名会误导模型选择。应报告 quality-cost frontier。

### 5. Benchmark 应优先分析失败案例

平均分不能解释模型为什么失败。评测系统应建立失败分类，例如：

- 配体 pose 错误；
- antibody-antigen 界面错位；
- 高 confidence 但外部质量差；
- 价态、手性或平面性错误；
- cryo-EM map 视觉增强但 map-model agreement 下降；
- 自由能计算不收敛或重复运行差异过大。

失败案例库比继续堆叠成功案例更能推动模型和数据改进。

### 6. Benchmark 与模型代码应该解耦

更成熟的形式是独立的：

- 数据仓库和版本化 manifest；
- evaluator 和指标实现；
- 模型 runner 与统一输入输出 schema；
- 权重、MSA、硬件和环境记录；
- 原始结果、统计脚本和可复现实验容器。

[BioEmu-Benchmarks](https://github.com/microsoft/bioemu-benchmarks)、[OpenFold3 benchmarking](https://github.com/aqlaboratory/openfold-3-benchmarking) 和 [Scaffold-Lab](https://github.com/Immortals-33/Scaffold-Lab) 都体现了评测代码独立于模型代码的方向。

### 7. Benchmark 要形成离线结果到真实结果的闭环

结构相似度、模型置信度和内部打分不能直接等价于真实结合或功能。不同 Seed 项目需要的闭环不同：

| 项目 | 当前主要评估 | 需要补上的真实决策层 |
|---|---|---|
| Protenix | 结构相似度和化学有效性 | confidence 校准、采样预算、模型选择和下游成功率 |
| PAR | motif success、designability | novelty、diversity、表达、结合、功能和实验命中率 |
| CryoFM | density map 和图像增强指标 | map-model agreement、局部分辨率、独立颗粒验证和建模收益 |
| Felis | 平均 ΔG 误差 | 收敛性、不确定性、重复性、化学体系分层和 prospective hit rate |

在实验预算有限时，也应预注册候选选择规则，避免只验证最漂亮的结果。

## 对四个项目的抽象判断

| 项目 | 主要问题不是缺什么指标，而是缺什么思想 |
|---|---|
| Protenix/PXMeter | 从“结构预测排行榜”升级为“置信度、预算和模型选择系统” |
| PAR | 从“生成成功率”升级为“生成分布、可制造性和真实功能概率” |
| CryoFM | 从“图像增强质量”升级为“结构解释和下游建模收益” |
| Felis | 从“平均自由能误差”升级为“收敛、不确定性和药物决策价值” |

## 最值得做的 benchmark 贡献

如果只做一个项目，建议把 Protenix/PXMeter 做成独立审计，而不是复制官方排行榜：

1. 固定时间切分、低同源集合和同一 target intersection；
2. 比较 Protenix、Boltz、OpenFold3 的 confidence 与外部质量；
3. 报告 top-1/top-k/oracle、采样预算曲线和 GPU 成本；
4. 分析抗体-抗原、蛋白-配体和低同源 target 的失败模式；
5. 给出“什么 target 该选什么模型”的选择策略。

这类工作真正的贡献不是声称重新发明结构指标，而是建立一套可接入新模型、能解释失败、能支持实验选择的评测系统。

## 面试中的准确表述

只有完成可复现实验、统计和失败分析后，才适合说“我做过 benchmark”。如果目前完成的是资料整理和方案设计，更准确的说法是：

> 我设计了一套 benchmark 审计方案，重点验证模型 confidence、采样预算、外部结构质量和计算成本之间的关系，并为后续的 Protenix/PXMeter 独立复现实现了数据、模型和统计协议。
这比只说“我整理过几个模型的 benchmark 分数”更能体现 benchmark 思维。
