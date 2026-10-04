# ByteDance Seed Bio & Pharma

集中维护 ByteDance Seed 团队公开的生物医药、蛋白质、分子建模和量子化学项目 fork。

GitHub 的 fork 网络按上游项目分别管理，因此每个项目会以独立仓库存在；本仓库用于索引、记录来源和后续维护。

## 项目索引

| 项目 | 方向 | 我的 fork | 上游 |
| --- | --- | --- | --- |
| `Protenix` | 生物分子复合物结构预测 | — | [bytedance/Protenix](https://github.com/bytedance/Protenix) |
| `PXMeter` | 生物分子结构预测统一评测 | — | [bytedance/PXMeter](https://github.com/bytedance/PXMeter) |
| `cryofm` | cryo-EM 密度图生成基础模型 | [zehaoli0324-cloud/cryofm](https://github.com/zehaoli0324-cloud/cryofm) | [ByteDance-Seed/cryofm](https://github.com/ByteDance-Seed/cryofm) |
| `par-protein` | 蛋白质自回归建模 | [zehaoli0324-cloud/par-protein](https://github.com/zehaoli0324-cloud/par-protein) | [ByteDance-Seed/par-protein](https://github.com/ByteDance-Seed/par-protein) |
| `felis` | 配体-蛋白质相互作用自由能 | [zehaoli0324-cloud/felis](https://github.com/zehaoli0324-cloud/felis) | [ByteDance-Seed/felis](https://github.com/ByteDance-Seed/felis) |
| `THEMol` | 分子建模 | [zehaoli0324-cloud/THEMol](https://github.com/zehaoli0324-cloud/THEMol) | [ByteDance-Seed/THEMol](https://github.com/ByteDance-Seed/THEMol) |
| `JoltQC` | 量子化学 GPU 内核 | [zehaoli0324-cloud/JoltQC](https://github.com/zehaoli0324-cloud/JoltQC) | [ByteDance-Seed/JoltQC](https://github.com/ByteDance-Seed/JoltQC) |

## 维护说明

- 各 fork 遵循对应上游仓库的许可证和贡献规则。
- 上游变更通过各自 fork 的 GitHub fork 网络同步。
- 本索引仓库只保存项目清单和维护信息，不复制上游代码。

## Benchmark

按项目整理的 benchmark 入口、国外同类项目对比、评测缺口和补强路线见 [docs/benchmarks.md](docs/benchmarks.md)。本页只比较生物方向；THEMol 和 JoltQC 的量子化学内容不纳入该分析。
