# Seed Bio & Pharma 研究索引

核查日期：2026-10-06。

这是本次研究对话的导航页。它把项目清单、国外对标、benchmark 判断、实践项目和面试叙事放在一起；详细技术内容仍保留在各专题文档中。

## 先看结论

Seed STEM 生物方向最值得对标的不是一个单独模型，而是四种能力的组合：

1. **生物基础模型**：结构预测、蛋白生成、蛋白动力学和基因组建模。
2. **科研计算平台**：训练、推理、GPU 加速、模型服务和可复现实验环境。
3. **科研 Agent**：证据检索、工具调用、实验规划和结果追踪。
4. **计算到实验闭环**：候选生成、筛选、湿实验验证、失败分析和下一轮迭代。

Seed 当前最适合优先研究的实践项目是 **Protenix/PXMeter 独立 benchmark 审计**。原因是数据和评测工具相对成熟，能够在不训练大模型、不立即依赖湿实验的情况下，做出可复现、可解释、对模型选择有实际价值的研究。

核心研究问题不是“哪个模型平均分最高”，而是：

> 模型在什么任务上会失败？它是否知道自己会失败？增加采样和计算预算是否值得？评测结果能否改变下一轮科研决策？

## 文档地图

| 文档 | 用途 |
|---|---|
| [项目 benchmark 对比](benchmarks.md) | Protenix、PAR、CryoFM、Felis 的现状、国外同类参照和补强等级 |
| [Benchmark 方法论](benchmark-philosophy.md) | 从论文排行榜转向泛化、校准、成本、失败和真实决策价值 |
| [Protenix/PXMeter 审计方案](protenix-benchmark-audit.md) | 可直接执行的研究问题、数据、模型、指标、统计和 M0-M3 里程碑 |

## Seed 生物项目

| 项目 | 方向 | 当前判断 | 适合做什么 |
|---|---|---|---|
| [Protenix](https://github.com/bytedance/Protenix) | 生物分子复合物结构预测 | Seed 目前 benchmark 工程化最完整的项目 | 独立复现、confidence calibration、top-k 选择、质量-成本分析 |
| [PXMeter](https://github.com/bytedance/PXMeter) | 结构预测统一评测 | 支持 RecentPDB、PoseBusters、AF3-AB、RNA-Protein、dsDNA-Protein 等 | 作为统一 evaluator 和结果聚合基础 |
| [PAR](https://github.com/ByteDance-Seed/par-protein) | 多尺度蛋白骨架生成 | 有 unconditional generation 和 motif scaffolding 评测，但跨模型 harness 仍可加强 | 接入 PXDesignBench，比较 designability、novelty、diversity 和效率 |
| [CryoFM](https://github.com/ByteDance-Seed/cryofm) | 冷冻电镜密度图生成与增强 | 有 downstream task 和测试脚本，但缺少统一 leaderboard 和外部验证面板 | 对比 RELION、DeepEMhancer、EMReady，建立 half-map/map-model 评测 |
| [Felis](https://github.com/ByteDance-Seed/felis) | 蛋白-配体绝对结合自由能 | 有数据集、ABFE 示例和论文 benchmark，但运行昂贵、收敛和重复性需加强 | 与 OpenFE/SAMPL 统一 protocol，逐步做 prospective 验证 |
| THEMol / JoltQC | 分子建模 / 量子化学 | 本研究索引暂不展开 | 另行维护，不纳入当前生物 benchmark 主线 |

## 国外对标地图

| 能力层 | 对标团队/项目 | 对 Seed 的启发 |
|---|---|---|
| 结构预测 | Google DeepMind [AlphaFold 2/3](https://deepmind.google/science/alphafold/)、Isomorphic Labs、[OpenFold3](https://github.com/aqlaboratory/openfold-3-benchmarking)、[Boltz](https://github.com/jwohlwend/boltz) | 复合物、蛋白-配体、核酸结构预测，以及统一评测和推理效率 |
| 蛋白语言模型与设计 | Evolutionary Scale/Biohub [ESM](https://github.com/Biohub/esm)、Baker Lab [RFdiffusion](https://github.com/RosettaCommons/RFdiffusion)、[ProteinMPNN](https://github.com/dauparas/ProteinMPNN)、RoseTTAFold All-Atom | 生成分布、结构条件设计、binder 设计和实验命中率 |
| 蛋白动力学 | Microsoft Research [BioEmu](https://github.com/microsoft/bioemu)、[BioEmu-Benchmarks](https://github.com/microsoft/bioemu-benchmarks) | 从静态结构扩展到构象集合、自由能和动力学分布 |
| 平台与部署 | NVIDIA [BioNeMo](https://docs.nvidia.com/bionemo-framework/latest/main/index.html)、[BioNeMo Recipes](https://github.com/NVIDIA-BioNeMo/bionemo-recipes)、[Agent Toolkit](https://github.com/NVIDIA-BioNeMo/bionemo-agent-toolkit) | 训练配方、GPU 加速、NIM 服务、工具编排和企业部署 |
| 科研 Agent | OpenAI [Science](https://openai.com/science/)、[GPT-Rosalind](https://openai.com/index/introducing-gpt-rosalind/)、[FrontierScience](https://openai.com/index/frontierscience/)，Anthropic [Life Sciences](https://www.anthropic.com/research/claude-for-life-sciences) | 通用模型连接文献、数据库、实验工具和安全评测；不是 Protenix 的直接模型对手 |

## Benchmark 现状判断

### Protenix/PXMeter：最适合先做

官方已经提供 benchmark 结果包和 PXMeter 评测流程。公开入口包括：

- [Protenix v1 benchmark data](https://github.com/bytedance/Protenix/blob/main/docs/model_1.0.0_benchmark.md)
- [PXMeter benchmark usage](https://github.com/bytedance/PXMeter/blob/main/docs/benchmark.md)
- [PXMeter releases](https://github.com/bytedance/PXMeter/releases)

当前缺口不是基础指标缺失，而是：

- confidence 是否校准；
- top-1/top-k 是否真的选出好结构；
- 采样预算和 GPU 成本是否值得；
- 低同源、抗体-抗原、蛋白-配体子集是否改变结论；
- 官方 artifact 和独立 inference 的差异；
- 失败样本是否被系统分类。

### PAR：创新空间大，但容易重复已有工作

PAR 已经有 motif benchmark 和 designability 评测；Seed 还发布了 [PXDesignBench](https://github.com/bytedance/PXDesignBench)，支持 monomer/binder 任务、ProteinMPNN、ESMFold、AlphaFold2 和 Protenix。单纯再做一张生成质量排行榜，新增价值有限。

更好的切入是：统一 RFdiffusion、ProteinMPNN、ESM3、PAR 的生成预算，增加 novelty、diversity、结构分布覆盖、计算效率和功能代理指标。

### CryoFM：缺口明显，但项目成本高

冷冻电镜 benchmark 需要同时处理 map、half-map、atomic model、mask、分辨率和结构验证。应关注 FSC-0.5、map-model FSC、Q-score、CC、局部分辨率、下游建模成功率、速度、显存和失败率，而不能只展示增强前后的图像。

可参考 [EMReady 评测](https://www.nature.com/articles/s41467-023-39031-1)、DeepEMhancer 和 RELION。

### Felis：科研价值高，但不适合作为第一项目

Felis 的 ABFE 计算本身昂贵，还需要收敛诊断、多重复、不确定性和实验参考。应与 [OpenFE](https://github.com/OpenFreeEnergy/openfe) 的 protocol、SAMPL/FEP+ 参考和 prospective hit rate 对齐。

## Benchmark 应具备的思想

成熟 benchmark 不是“准备数据、跑指标、排排行榜”，而是一个科学测量和决策系统。至少应覆盖：

1. **泛化**：时间切分、低同源、分布外和污染审计。
2. **决策质量**：top-k 选择、候选淘汰、模型选择和实际命中率。
3. **置信度校准**：模型是否知道自己什么时候会错。
4. **质量-成本**：GPU 时间、显存、MSA/模板成本、采样收益曲线。
5. **失败分析**：结构错位、配体 pose、化学无效、高 confidence 错误和不收敛。
6. **评测解耦**：独立数据、evaluator、manifest、模型 runner、原始结果和统计脚本。
7. **真实闭环**：从离线分数走向实验命中率、失败数和下一轮迭代收益。

一句话概括 Seed 项目共同的不足：

> 目前更像是在做证明模型有效的论文 benchmark，还没有完全升级为衡量泛化、可信度、成本、失败风险和真实科研决策价值的 benchmark 基础设施。

## 推荐实践项目

项目名称：**Independent Reproducibility and Confidence Audit for Biomolecular Structure Predictors**。

### 第一版范围

- 数据：`PXM-2025-H2`、`PXM-22to25-Ab-Ag`、`PXM-22to25-Ligand`；
- 模型：Protenix v1，资源允许时加入 Protenix v2、Boltz、OpenFold3；
- 指标：LDDT、DockQ、ligand RMSD、pocket RMSD、LDDT-PLI、PoseBusters；
- 分析：confidence calibration、top-1/top-k/oracle、bootstrap CI、失败分类、GPU seconds 和峰值显存。

### 最小执行顺序

1. 下载 3–5 个公开 target，跑通一个模型和 PXMeter。
2. 扩展到 20–50 个 target、至少两个模型和三个 seed。
3. 统一模型版本、权重 hash、MSA/template、硬件和预算。
4. 输出 per-sample 原始结果、summary CSV、失败日志和复现 manifest。
5. 再扩展抗体-抗原和蛋白-配体子集，给出模型选择策略。

不要把官方结果直接写成自己的推理结果。结果中区分 `official_artifact`、`independent_inference` 和 `re-evaluated_output`。

## 湿实验验证思路

计算 benchmark 不能代替湿实验。建议采用分阶段闭环：

1. 先用计算模型生成候选，并预注册筛选规则；
2. 进行小批量表达、纯化、结合和稳定性验证；
3. 对失败和边界样本保留记录，而不是只展示命中候选；
4. 根据实验结果更新 ranking、数据切分和模型选择规则；
5. 达到稳定命中率后再扩大实验规模。

外包 CRO 可以解决通量和设备问题，但不应把所有能力外包出去。较好的组合通常是：内部保留 assay 设计、样本选择、数据标准、质量控制和结果解释；把标准化、高通量、设备密集型实验交给有资质的外部平台；关键机制和失败复核保留在内部或合作实验室。

## 面试叙事

如果只完成资料整理，应说：

> 我设计了一套 benchmark 审计方案，重点验证模型 confidence、采样预算、外部结构质量和计算成本之间的关系，并为 Protenix/PXMeter 独立复现定义了数据、模型和统计协议。

完成 M1 以上后，可以说：

> 我基于 PXMeter 建立了一个独立、同预算的结构预测评测流程，比较 Protenix、Boltz 和 OpenFold3 的外部结构质量、confidence 校准、化学有效性和 GPU 成本，并按 target 类型分析模型排序和失败模式，最后输出模型选择策略，而不是只给一个总分。

面试中应能回答：

- 如何防止数据泄漏和结构同源污染？
- 为什么选择这些指标和 target 子集？
- confidence 和真实质量是否校准？
- top-k 与 oracle 的差距是什么？
- 哪些失败案例改变了你的结论？
- 计算预算增加后收益是否递减？
- 哪些结论必须通过湿实验验证？

## 不要混淆的边界

- 公开仓库不等于完整模型开源；代码、权重、训练数据、数据库和托管服务分别判断。
- AlphaFold 3、OpenAI GPT-Rosalind、Anthropic Claude for Life Sciences、Isomorphic Labs 和 Schrödinger FEP+ 不能简单视为可 fork 的开源项目。
- benchmark 方案、benchmark runner 和已完成的 benchmark 实验是三种不同状态。
- 没有真实实验命中率时，不要把结构分数写成药物发现成功率。
- 没有运行 M1 以上实验时，面试中应说“设计并实现部分 benchmark”，不要说“完整做过 benchmark”。

## 下一步清单

- [ ] 完成 M0：3–5 个 target 的单模型最小复现。
- [ ] 固定 `data/manifest.yaml`、模型 commit、权重 hash 和环境版本。
- [ ] 设计 per-sample 结果 schema 和 `failure.jsonl`。
- [ ] 接入第二个模型并做同 target intersection。
- [ ] 加入 confidence calibration 和 top-k/oracle 分析。
- [ ] 输出第一版 reproducibility report。
- [ ] 再决定是否投入 CryoFM 或 Felis 的高成本 benchmark。
