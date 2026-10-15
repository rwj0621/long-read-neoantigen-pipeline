# 基于长读长测序数据的肿瘤新抗原筛选研究

> **文档定位：** 本 README 是整个研究的**流程导航页**；每个工具的环境、实际运行命令、原生输出及检查方法，放在独立 Markdown 中。当前是**框架版**，尚未填写或验证具体运行命令。

## 1. 研究分析总流程

```text
二代 WGS（NGS）                 三代 WGS（TGS / HiFi）
      │                               │
      ├── SNV / INDEL 识别 ────────────┤
      │      （各工具原始结果）        │
      └── SV 识别 ─────────────────────┘
                    │
        原始输出收集与质量检查
                    │
       统一过滤、标准化、结果统计
                    │
         多工具整合与平台比较
                    │
           高可信候选变异集合
                 ┌──┴──┐
         SNV / INDEL    SV
              │         │
           pVACtools   NeoSV
                 └──┬──┘
          候选新抗原汇总与比较
```

**原则：** 保留各工具原始结果不修改；统一过滤和跨平台比较都在独立步骤完成；每步保存输入、输出、参数和变异数量，以保证分析可复现。

## 2. SNV / INDEL 识别（Calling）

| 平台 | 工具 | 运行文档 | 原生输出 | 运行状态 |
|---|---|---|---|---|
| TGS | DeepSomatic | [查看说明](docs/01_SNV_INDEL/DeepSomatic_TGS.md) | 待记录 | 待更新 |
| TGS | ClairS | [查看说明](docs/01_SNV_INDEL/ClairS_TGS.md) | 待记录 | 待更新 |
| NGS | DeepSomatic | [查看说明](docs/01_SNV_INDEL/DeepSomatic_NGS.md) | 待记录 | 待更新 |
| NGS | ClairS | [查看说明](docs/01_SNV_INDEL/ClairS_NGS.md) | 待记录 | 待更新 |
| NGS | Mutect2 | [查看说明](docs/01_SNV_INDEL/Mutect2_NGS.md) | 待记录 | 待更新 |

## 3. 结构变异（SV）识别（Calling）

| 平台 | 工具 | 运行文档 | 原生输出 | 运行状态 |
|---|---|---|---|---|
| TGS | Severus | [查看说明](docs/02_SV/Severus_TGS.md) | 待记录 | 待更新 |
| TGS | SAVANA | [查看说明](docs/02_SV/SAVANA_TGS.md) | 待记录 | 待更新 |
| TGS | nanomonsv | [查看说明](docs/02_SV/nanomonsv_TGS.md) | 待记录 | 待更新 |
| TGS | SVision-pro（复杂SV） | [查看说明](docs/02_SV/SVision_pro_TGS.md) | 待记录 | 待更新 |
| NGS | Manta | [查看说明](docs/02_SV/Manta_NGS.md) | 待记录 | 待更新 |
| NGS | GRIDSS | [查看说明](docs/02_SV/GRIDSS_NGS.md) | 待记录 | 待更新 |
| NGS | SvABA | [查看说明](docs/02_SV/SvABA_NGS.md) | 待记录 | 待更新 |

## 4. 统一预处理、过滤与比较（Post-processing）

> 该部分**不属于单个检测工具的运行步骤**，统一处理多个 Caller 的原始结果，并将过滤结果保存为新的文件。

| 阶段 | 主要任务 | 详细说明 | 产物 |
|---|---|---|---|
| SNV/INDEL 过滤 | 统一格式、质量与体细胞候选过滤 | [SNV/INDEL 过滤](docs/03_Postprocessing/SNV_INDEL_filtering.md) | 标准化 SNV/INDEL 集合 |
| SV 过滤 | PASS、VAF、染色体、长度、BND 事件等 | [SV 过滤](docs/03_Postprocessing/SV_filtering.md) | 标准化 SV 集合 |
| 平台内部整合 | 分别整合 NGS / TGS 多工具结果 | [平台整合与比较](docs/03_Postprocessing/Platform_integration.md) | NGS/TGS 各自的集合 |
| 跨平台比较 | NGS-only、TGS-only、Both；类型、大小、断点 | [平台整合与比较](docs/03_Postprocessing/Platform_integration.md) | 比较表及图件 |
| 准确性评估 | 公共标准集或其他可核验证据 | [评估与验证](docs/03_Postprocessing/Benchmark.md) | 指标、验证证据 |
| 预测前整理 | 注释、统一基因组版本、满足下游输入格式 | [新抗原输入准备](docs/03_Postprocessing/Neoantigen_handoff.md) | 预测输入文件 |

**过滤原则待定：** 各工具 FILTER/支持证据/VAF 字段可能不同；**不能**把所有 VCF 简单按相同字段过滤，更不能直接把 BND 记录行数当作 SV 事件数。

## 5. 新抗原预测（Neoantigen Prediction）

| 变异来源 | 工具 | 运行文档 | 输入交接 | 输出记录 |
|---|---|---|---|---|
| SNV / INDEL | pVACtools（pVACseq） | [查看说明](docs/04_Neoantigen/pVACtools_SNV_INDEL.md) | 待填写 | 待填写 |
| SV | NeoSV | [查看说明](docs/04_Neoantigen/NeoSV_SV.md) | 待填写 | 待填写 |

## 6. 结果汇总与绘图（待完善）

| 分析内容 | 目标图/表 | 数据来源 | 状态 |
|---|---|---|---|
| 各工具检测数量 | 分组柱状图 | Caller 原生及过滤后统计 | 待完成 |
| NGS/TGS SV 类型组成 | 堆叠柱状图 | 统一 SV 表 | 待完成 |
| SV 长度分布 | 箱线图/密度图 | 统一 SV 表 | 待完成 |
| 工具间一致性 | UpSet 图 | 平台内部整合集合 | 待完成 |
| NGS vs TGS 差异 | NGS-only/TGS-only/Both | 跨平台匹配结果 | 待完成 |
| 新抗原来源与结果 | 对比图 | pVACtools / NeoSV 输出 | 待完成 |

## 7. 文档填写规则

1. **工具 README**：只写工具安装/环境、输入、实际运行命令、工具原生输出、运行检查与日志，以及下游交接文件；不重复粘贴公共过滤代码。
2. **统一过滤 README**：按 SNV/INDEL 和 SV 分开，记录过滤前/后的统计、阈值及事件定义；始终保留原始结果。
3. **平台比较 README**：定义跨 Caller 和跨平台的匹配规则，再绘图；比较一致性不能直接等同于真实准确性。
4. **实际运行信息**：未运行写“待运行”，未知输出写“待确认”，不要用猜测文件名冒充原生输出。

## 8. 推荐文档目录

```text
README.md                         # 总方案 / 导航

docs/
  01_SNV_INDEL/                  # 五个调用工具页面
  02_SV/                         # 七个调用工具页面
  03_Postprocessing/             # 统一过滤、整合、评估、预测输入准备
  04_Neoantigen/                 # 两个新抗原预测工具页面
```

> 实际脚本、输入 BAM/VCF、输出目录可以继续存放在你现有的 `stepXX` 项目路径中。该文档目录仅用于组织 Markdown，不要求移动数据或脚本。
