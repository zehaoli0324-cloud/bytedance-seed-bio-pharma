# Protenix/PXMeter 独立 Benchmark 审计方案

状态：研究计划，尚未声称已运行结果。

这是本仓库建议优先实现的第一个 benchmark 项目。目标不是重新训练 Protenix，也不是重复官方排行榜，而是验证：**在固定数据、采样预算和硬件条件下，模型的 confidence/ranking 是否真的能挑出结构质量更高、化学有效性更好的预测。**

官方入口：

- [Protenix](https://github.com/bytedance/Protenix)
- [Protenix v1 benchmark results](https://github.com/bytedance/Protenix/blob/main/docs/model_1.0.0_benchmark.md)
- [PXMeter benchmark documentation](https://github.com/bytedance/PXMeter/blob/main/docs/benchmark.md)
- [PXMeter releases](https://github.com/bytedance/PXMeter/releases)

## 为什么先做这个

1. PXMeter 已有可下载的预构建数据，文档说明数据集可以直接使用，且预构建数据采用 CC0；不需要先自己做 PDB 清洗。
2. 评测器已经覆盖 LDDT、DockQ、配体/口袋 RMSD、PoseBusters、LDDT-PLI 和 CDR-H3 等指标，主要工作转为实验设计、统一运行和误差分析。
3. PXMeter 的评测输入和结果格式支持扩展模型，近期版本已经加入 OpenFold3 evaluator，适合接入 Boltz 和其他结构预测器。
4. Protenix 官方已经发布了 PXM-2024、PXM-2025、抗体-抗原、蛋白-配体和 FoldBench 等结果包，因此可以先做小规模复现，再逐步扩大。

相比重新设计一个蛋白设计指标，这个方向更容易在有限 GPU 和时间内完成一个可检查的成果；相比 Felis 的自由能模拟，也不需要长时间的多重复采样。

## 核心研究问题

### Q1：模型自己的 confidence 能否预测外部结构质量？

对每个 target 和每个 seed 保存模型 confidence，并与外部的 DockQ、LDDT、ligand RMSD、PoseBusters validity 对齐，计算 Spearman/Pearson、分桶校准曲线和 ECE/Brier 等校准指标。

### Q2：top-1 选择是否真的优于简单选择？

同时报告以下 ranker：

- 模型 confidence 的 top-1；
- top-5/top-10 中的 best-of-N；
- random、median 和 worst 控制组；
- 只允许固定 seed 数的公平选择；
- 使用真实结构指标选择的 oracle 上界。

这可以量化“增加采样预算”和“模型会不会挑样本”分别贡献了多少收益。

### Q3：平均分是否掩盖了真实弱点？

按以下子集分别报告结果：

- protein-protein、protein-ligand、antibody-antigen；
- low-homology 与高同源 target；
- 单体、复合物、肽、核酸和含配体结构；
- 结构长度、分辨率和可用 MSA 深度；
- 化学有效与 PoseBusters 失败样本。

### Q4：更高质量是否值得额外计算成本？

记录 GPU seconds、峰值显存、CPU 数据准备时间、失败率和每个有效预测的成本，画出质量-成本 frontier，而不是只给 accuracy 排名。

## 第一版数据与模型矩阵

### 数据

第一版不追求全量，建议先用 20-50 个 target 做 smoke/reproducibility run，再扩展到完整子集：

1. `PXM-2025-H2`：时间较新的结构预测集合；
2. `PXM-22to25-Ab-Ag`：抗体-抗原专项集合；
3. `PXM-22to25-Ligand`：蛋白-配体专项集合。

数据目录、下载地址、SHA256 和数据许可写入 `data/manifest.yaml`。如果直接复用 Protenix 已发布的预测结果，必须在结果中标记 `inference_source: official_artifact`，不能把它写成自己的推理结果。

### 模型

建议矩阵如下：

| 模型 | 作用 | 第一版要求 |
|---|---|---|
| Protenix v1 | 与官方报告对齐的 Seed 主模型 | 必跑，固定版本和权重哈希 |
| Protenix v2 | 当前应用模型 | 资源允许时加入，单独报告权重和条款 |
| Boltz-1/2 | 开放的 AF3 类结构预测 baseline | 至少选择一个可稳定运行的版本 |
| OpenFold3 | 另一套 AF3 类开放实现 | PXMeter 已有 evaluator 时优先接入 |
| 简单 baseline | 防止复杂模型比较失去参照 | 可使用官方已发布结构或低预算配置 |

AlphaFold 3 可以作为论文/服务参考，但第一版不应把受限权重当作项目必需依赖。所有模型都要记录代码 commit、权重来源、MSA/模板来源、采样数和实际硬件。

## 统一评测协议

### 输入和推理

- 固定输入格式和 reference assembly 规则；
- 固定 seed 列表，例如 `[101, 102, 103]`；
- 固定每个 target 的样本数；
- MSA 和 template 缓存到版本化目录，避免远程搜索服务变化；
- 记录 GPU 型号、CUDA、驱动、容器 digest 和运行时间；
- 任何失败都写入 `failure.jsonl`，不能静默跳过。

### 外部指标

| 类别 | 指标 |
|---|---|
| 蛋白结构 | LDDT、DockQ、界面接触恢复、主链 RMSD |
| 蛋白-配体 | ligand RMSD、pocket RMSD、LDDT-PLI、PoseBusters validity |
| 抗体-抗原 | DockQ、界面 LDDT、CDR-H3 backbone RMSD |
| 模型选择 | confidence 与外部指标的相关性、top-k success rate、calibration error |
| 系统效率 | GPU seconds、峰值显存、CPU 准备时间、失败率、有效预测成本 |

### 统计

- 以 target 或低同源 cluster 为统计单位，避免大 cluster 过度加权；
- 报告均值、中位数、分位数和 bootstrap 95% CI；
- 对成功率提供分母和阈值，不能只报百分比；
- 对多个模型使用同一 target intersection，明确缺失结果处理规则；
- 不用测试集指标选择 ranker 后再在同一测试集报告结果；oracle 只作为上界。

## 预期的 benchmark 漏洞与独立贡献

这些是待验证的假设，不应在实验前写成结论：

1. **confidence calibration 可能不足**：模型排序分数可能适合挑 top-1，但不能直接解释为成功概率。
2. **采样预算会改变排名**：5 个 seed 和 100 个 seed 的结论可能不同，需要画收益曲线。
3. **平均分可能掩盖子集退化**：总体 LDDT 上升，不代表抗体-抗原或配体 pose 同时改善。
4. **化学有效性不能由几何分数替代**：低 RMSD 结构仍可能通过不了 PoseBusters 的价态、手性或平面性检查。
5. **外部 MSA 服务会破坏复现**：同一输入在不同时间获得的 MSA 可能不同，必须缓存或记录服务版本。
6. **官方结果与独立运行可能有偏差**：需要区分官方 artifact、独立 inference 和同一 evaluator 的差异来源。

独立贡献应落在“审计和决策”上，而不是宣称重新发明结构指标：

- 一个可以接入新模型的统一 runner；
- 一个包含输入、权重、MSA、硬件和日志的 manifest；
- confidence/ranking calibration 分析；
- target-level failure taxonomy；
- 质量-成本和采样预算曲线；
- 一份能回答“哪个模型适合哪个 target 类型”的选择报告。

## 建议的仓库结构

```text
protenix-benchmark-audit/
├── configs/
│   ├── models.yaml
│   └── datasets.yaml
├── data/
│   └── manifest.yaml
├── runners/
│   ├── run_model.py
│   └── normalize_outputs.py
├── evaluators/
│   ├── run_pxmeter.py
│   ├── calibration.py
│   └── cost.py
├── analysis/
│   ├── summarize.py
│   ├── stratify.py
│   └── plots.py
├── results/
│   ├── raw/
│   └── summary/
├── reports/
│   └── reproducibility_report.md
├── Dockerfile
└── README.md
```

原始结构文件和权重不应直接提交到索引仓库；提交下载脚本、manifest、哈希和结果摘要。大文件使用外部 artifact，并记录永久版本号。

## 分阶段完成标准

### M0：安装与最小复现

- 下载 3-5 个公开 target；
- 跑通一个模型和 PXMeter；
- 生成一份 per-sample JSON 与 summary CSV；
- 验证手工抽查的 LDDT/DockQ/RMSD 与工具输出一致。

### M1：小规模独立审计

- 20-50 个 target；
- 至少两个模型、三个 seed；
- 完成 top-1/top-k/oracle、confidence calibration 和成本统计；
- 所有失败都有记录，结果可从 manifest 重建。

### M2：跨模型面板

- 扩展到抗体-抗原和蛋白-配体子集；
- 引入 Boltz/OpenFold3；
- 增加低同源、长度和结构类型分层；
- 发布 Docker、运行命令、原始结果索引和 CI smoke test。

### M3：科研报告

- 给出置信度是否可信的结论；
- 给出每类 target 的模型选择策略；
- 报告结果不确定性、失败模式和质量-成本 frontier；
- 明确哪些结论需要湿实验才能验证。

## 面试时如何描述

可以把项目概括为：

> 我没有只复述 Protenix 的官方排行榜，而是基于 PXMeter 建立了一个独立、同预算的结构预测评测流程。项目同时记录模型 confidence、外部结构质量、化学有效性、采样预算和 GPU 成本，并用 target-level bootstrap 和子集分析检查模型排序是否稳定。最后输出的是不同 target 类型下的模型选择策略和失败案例，而不是一个不带上下文的总分。

只有真正完成 M1 以上，才应使用“我做过 benchmark”这个表述；如果只完成了资料整理，应说“我设计了 benchmark 方案并实现了部分 runner”。

## 参考资料

- [PXMeter benchmark usage](https://github.com/bytedance/PXMeter/blob/main/docs/benchmark.md)
- [PXMeter evaluation details](https://github.com/bytedance/PXMeter/blob/main/docs/pxmeter_eval_details.md)
- [Protenix benchmark data and results](https://github.com/bytedance/Protenix/blob/main/docs/model_1.0.0_benchmark.md)
- [OpenFold3 benchmarking pipeline](https://github.com/aqlaboratory/openfold-3-benchmarking)
- [Boltz repository](https://github.com/jwohlwend/boltz)
