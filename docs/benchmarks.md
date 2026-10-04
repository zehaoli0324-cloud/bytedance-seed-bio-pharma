# ByteDance Seed 生物项目 Benchmark 对比

核查日期：2026-10-04。本页比较 `Protenix`、`PAR`、`CryoFM` 和 `Felis` 四个生物方向项目。`THEMol` 与 `JoltQC` 属于分子建模/量子化学，本页不展开。

这里的“有 benchmark”分成两种：

- **论文 benchmark**：论文或 README 报告了结果，但未必提供完整的一键复现实验。
- **工程化 benchmark**：同时提供固定数据、评测脚本、指标实现、配置/权重和可追溯输出，第三方可以复跑并比较新模型。

下表的 A/B/C 是工程化程度，不是模型科学质量排名。

## Seed 项目现状

| 项目 | 当前自带内容 | 工程化等级 | 与国外同类相比的主要缺口 |
|---|---|---:|---|
| [Protenix](https://github.com/bytedance/Protenix) | 官方有 v1/v2 benchmark 文档；同团队 [PXMeter](https://github.com/bytedance/PXMeter) 提供 RecentPDB、PoseBusters、AF3-AB、RNA-Protein、dsDNA-Protein，以及 LDDT、DockQ、RMSD 和化学有效性检查 | **A-** | 已经是 Seed 最完整的 benchmark。仍需把模型版本、权重、MSA/模板、随机种子、硬件预算和原始输出固化成可复现实验包；与 OpenFold3、Boltz 的统一盲测和置信度校准还不够系统 |
| [PAR](https://github.com/ByteDance-Seed/par-protein) | 有 unconditional generation 评测和 motif scaffolding 脚本（`motif_scaffold_par.sh`、`compute_motif_success.py`），使用 designability 等指标 | **B** | 评测偏论文实验，缺少长期维护的跨模型 harness；应补 RFdiffusion、Chroma、FrameFlow、ESM3 等统一 baseline，报告 novelty、diversity、效率、长度/拓扑分层及置信区间 |
| [CryoFM](https://github.com/ByteDance-Seed/cryofm) | 有 CryoFM1/2 测试和 denoising、inpainting、anisotropy correction、style enhancement 等 downstream task | **C+/B-** | README 和代码没有统一 leaderboard、公开版本化 test manifest 和跨方法聚合脚本；需对比 RELION、DeepEMhancer、EMReady，并同时报告 half-map FSC、map-model FSC、FSC-0.5、Q-score、CC、局部分辨率、速度和显存 |
| [Felis](https://github.com/ByteDance-Seed/felis) | 有 `pl_bfe_dataset`、完整 ABFE 示例和大规模 benchmark 论文；代码可以跑工作流，但不是通用跨模型评测框架 | **B-** | 缺少统一的重复运行、收敛诊断、不确定性和失败分类；需要与 OpenFE/OpenMM 协议、SAMPL/FEP+ 参考结果对齐，并做 blind prospective hit-rate，而不只比较离线 ΔG 误差 |

## 国外同类型参照

| 方向 | 参照项目 | benchmark 能力 | Seed 应学习的做法 |
|---|---|---|---|
| 复合物结构预测 | [AlphaFold 3](https://github.com/google-deepmind/alphafold3)、[OpenFold3 benchmarking](https://github.com/aqlaboratory/openfold-3-benchmarking)、[Boltz](https://github.com/jwohlwend/boltz) | AF3 官方仓库主要提供性能/运行测试；OpenFold3 benchmarking 提供预测、OpenStructure 评估和 CSV 汇总；Boltz 持续维护结构与 affinity 评测入口 | 统一模型接口、测试集和输出格式；将结构精度、化学有效性、推理成本和 ranking calibration 放在同一张结果表里 |
| 蛋白骨架/序列设计 | [RFdiffusion](https://github.com/RosettaCommons/RFdiffusion)、[ProteinMPNN](https://github.com/dauparas/ProteinMPNN)、[Scaffold-Lab](https://github.com/Immortals-33/Scaffold-Lab) | RFdiffusion 提供 motif、binder、对称生成等任务；Scaffold-Lab 将 designability、diversity、novelty、efficiency 和结构性质放进统一框架 | PAR 需要从单一 success rate 扩展到生成质量、分布覆盖、计算效率和跨任务鲁棒性，并避免只用模型自己的打分器筛选 |
| 蛋白构象分布 | [BioEmu](https://github.com/microsoft/bioemu) + [bioemu-benchmarks](https://github.com/microsoft/bioemu-benchmarks) | 有独立 benchmark 包，覆盖 OOD 构象变化、domain motion、local unfolding、cryptic pocket、MD 分布和 folding free energy | Felis/PAR/CryoFM 也应将测试集、指标实现和结果文件独立发布；模型仓库升级不应悄悄改变 benchmark 数字 |
| 冷冻电镜 | [EMReady 评测](https://www.nature.com/articles/s41467-023-39031-1)、DeepEMhancer、RELION | 公开 test maps、half-map 和 map-model validation，常用 FSC-0.5、Q-score、CC 等，并报告失败样本和处理时间 | CryoFM 应发布固定 test manifest、独立 half-map、mask 规则、外部结构验证和失败/崩溃统计，而不只展示增强后的可视化图 |
| 自由能 | [OpenFE](https://github.com/OpenFreeEnergy/openfe)、[OpenFE ABFE protocol](https://docs.openfree.energy/en/v1.7.0/guide/protocols/absolutebinding.html)、SAMPL/FEP+ | 重点记录 thermodynamic protocol、采样/收敛诊断、MBAR 不确定性、重复计算和实验参考 | Felis 需要把 ΔG 误差、重复性、收敛性、计算成本和化学体系分层同时报告；商业 FEP+ 只能作为参考，不能直接当作开源 baseline |

## 主要欠缺

### 1. Benchmark 分散在项目中，没有统一入口

Protenix 已有 PXMeter，但 PAR、CryoFM 和 Felis 的评测方式、输出字段和版本记录各不相同。国外项目也并非全部完善，但 OpenFold3 benchmarking、BioEmu-Benchmarks 和 Scaffold-Lab 已经体现出“评测代码独立于模型代码”的方向。

应在本索引仓库维护一个统一 manifest，至少记录：

```yaml
project: protenix
model_version: protenix_base_default_v1.0.0
weights_sha256: required
dataset_version: required
split_rule: temporal_and_sequence_cluster
seeds: [101, 102, 103]
hardware: A100-80GB
budget: gpu_seconds
metrics: [lddt, dockq, ligand_rmsd, posebusters_valid]
```

### 2. 数据泄漏和泛化边界没有形成统一审计

结构预测至少需要时间切分、序列同源聚类、链/复合物去重和训练数据污染检查。设计任务还要避免把目标结构、已知 binder 或后处理结果泄漏到测试集。每个项目应该同时报告 overall、low-homology、长序列、抗体/肽/核酸/配体等子集，而不是只给一个均值。

### 3. 指标还没有覆盖真实下游目标

- Protenix 已覆盖 LDDT、DockQ、RMSD 和 PoseBusters，但还应报告 confidence calibration、top-k selection、GPU 成本和不同采样预算下的收益曲线。
- PAR 不应只看 motif success 或 designability；还需要 novelty、diversity、序列可合成性、表达/溶解度、结合和功能命中率。
- CryoFM 不能只看图像增强指标；要验证 map-model agreement、局部分辨率、半图一致性、结构建模成功率和下游拾取/建模收益。
- Felis 不能只看平均 ΔG 误差；必须报告重复计算方差、收敛失败、化学体系分层、实验相关性和 prospective hit rate。

### 4. 缺少跨模型、同预算的强 baseline

Seed 项目常以论文中的固定模型和固定设置展示结果。面向用户的 benchmark 应固定推理预算，并至少加入一个强基线、一个开放复现基线和一个简单基线。例如：Protenix 对比 AlphaFold 3/OpenFold3/Boltz，PAR 对比 RFdiffusion/ProteinMPNN/ESM3，CryoFM 对比 RELION/DeepEMhancer/EMReady，Felis 对比 OpenFE 和公开实验参考。

### 5. 离线 benchmark 与湿实验结果没有闭环

结构相似度、模型置信度和 Rosetta/打分器结果都不能直接等价于真实结合或功能。蛋白设计最终应有盲态表达、结合、特异性、稳定性和功能验证；药物项目应有 prospective synthesis/assay 和 hit rate；冷冻电镜应有独立颗粒或外部结构验证。实验预算有限时，也应预注册候选选择规则，避免只验证最漂亮的结果。

## 补强程度

### 第一层：可复现（必须完成）

适合公开 release 和面试展示，目标是让第三方能复跑：

- 固定 benchmark manifest、数据版本、时间/同源切分、模型 commit、权重哈希和硬件配置。
- 统一 CLI 和结果 schema，保存每个 target/seed/sample 的原始结果，而不只保存平均表格。
- 提供 Docker/conda lockfile、最小 smoke test、失败日志和 CI 数据完整性检查。
- 报告均值、分位数、bootstrap 置信区间、运行时间、显存和失败率。

### 第二层：科研可信（Seed STEM 应达到）

- 建立跨模型矩阵和 blinded holdout；按照 target 类型、同源性、长度和难度分层。
- 做 contamination audit、消融实验、采样预算曲线和 confidence calibration。
- 对 PAR 增加设计分布指标，对 CryoFM 增加 half-map/外部结构验证，对 Felis 增加 convergence/uncertainty，形成失败案例库。
- 建立版本化 leaderboard：任何新模型只能提交固定格式的预测和 metadata，评测服务器或受控脚本统一打分。

### 第三层：真正有影响力（领先水平）

- 对蛋白设计和药物发现做预注册的 prospective validation，公开候选选择规则和完整失败数。
- 建立外部实验合作，按批次记录 expression、binding、specificity、stability、functional assay 和复测结果。
- 把 benchmark 从“模型排名”升级为“下一轮实验决策”：记录计算成本、候选淘汰原因、实验命中率和迭代后的模型收益。
- 对关键结果发布数据、评测容器、原始预测、统计脚本和实验审计记录，形成可复核的 benchmark release。

## 建议的落地顺序

1. **先做结构预测公共面板**：以 PXMeter 为核心，加入 Protenix、Boltz、OpenFold3，固定时间切分和采样预算。
2. **再做 PAR 设计面板**：整合 RFdiffusion、ProteinMPNN、ESM3，并输出 designability、novelty、diversity、效率和 binder 代理指标。
3. **并行补 CryoFM 面板**：固定 half-map 和 map-model validation，加入 RELION、DeepEMhancer、EMReady 的失败与耗时统计。
4. **最后做 Felis 发现面板**：统一 ABFE protocol、重复数、收敛判据和实验参考，随后进入盲态 prospective 验证。

本页是 benchmark 工程与科研验证的路线，不代表已经运行上述全部模型。每次发布结果时，应把实际运行的模型版本、数据、权重、硬件、日志和原始输出一并归档。
