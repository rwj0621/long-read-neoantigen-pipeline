# 比对后质量可视化 — NanoPlot

[← 返回项目总 README](../../README.md)

使用 NanoPlot 展示比对后长读长数据的读长及质量分布，辅助识别异常分布。输入为坐标排序的 BAM；pbmm2 与 minimap2 结果分别运行，并使用独立输出目录。

## 核心命令

```bash
NanoPlot \
    --bam sample.sorted.bam \
    --outdir nanoplot_output \
    --threads 16 \
    --tsv_stats \
    --loglength
```

## 关键参数

| 参数 | 含义 |
|---|---|
| `--bam sample.sorted.bam` | 输入比对 BAM |
| `--outdir nanoplot_output` | 输出目录 |
| `--threads 16` | 处理线程数 |
| `--tsv_stats` | 将统计摘要写为制表符分隔格式 |
| `--loglength` | 同时展示读长对数坐标图，便于观察跨度较大的分布 |

## 输出与解读

| 输出 | 内容 |
|---|---|
| `NanoPlot-report.html` | 汇总统计与图形报告 |
| `NanoStats.txt` | TSV 格式统计摘要；扩展名仍为 `.txt` |
| 图形文件 | 读长分布及可用的质量、比对相关图形；具体图形取决于输入字段和软件版本 |

NanoPlot 侧重分布可视化，总体统计口径见 [Cramino / flagstat](Cramino_QC.md)。BAM 模式下的质量指标与原始 `rq` 标签统计来源不同，不直接替代 [原始读段质量评估](Raw_HiFi_QC.md)。

参考：[NanoPlot 官方文档](https://github.com/wdecoster/NanoPlot)。
