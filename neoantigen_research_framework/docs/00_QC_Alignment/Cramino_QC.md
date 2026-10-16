# 比对后总体统计 — Cramino / SAMtools

[← 返回项目总 README](../../README.md)

使用 Cramino 汇总长读长比对结果的读长、数据量及序列一致性，使用 SAMtools flagstat 统计比对记录的 FLAG 分类。输入为比对后的 BAM；pbmm2 与 minimap2 的结果分别运行，使用不同输出前缀。

## 核心命令

```bash
cramino \
    -t 16 \
    --format tsv \
    sample.sorted.bam \
    > sample.cramino.tsv

samtools flagstat \
    -@ 16 \
    sample.sorted.bam \
    > sample.flagstat.txt
```

## 关键参数

| 工具 / 参数 | 含义 |
|---|---|
| Cramino `-t 16` | 并行解压线程数 |
| Cramino `--format tsv` | 以 TSV 格式输出统计结果 |
| flagstat `-@ 16` | 输入解压附加线程数 |

## 指标与统计范围

| 输出 | 核心指标 |
|---|---|
| `sample.cramino.tsv` | Reads 数、比对数据量、读长 N50、平均 / 中位读长、平均 / 中位一致性 |
| `sample.flagstat.txt` | 输入记录总数、mapped、primary、secondary、supplementary 等分类及相应比例 |

Cramino 默认排除 secondary 和未比对记录；reads 数按 primary 记录统计，比对数据量及 aligned 读长指标还纳入 supplementary 记录。一致性采用 gap-compressed identity，依赖 `de` 标签，或 `NM` 标签与 CIGAR。该指标反映比对一致性，与原始 BAM 的 `rq` 预测准确率含义不同。

原始数据质控描述测序数据本身，本步骤描述比对后纳入统计的记录；两者的 reads 数、数据量和 N50 不应直接视为相同口径。全长读长指标与 aligned 读长指标也应分别解读。

flagstat 的 mapped 比例以输入 BAM 记录为统计基础。若比对输出未保留未比对 reads，该比例不能代表原始 reads 的整体比对率；包含 supplementary 的记录数也不能直接作为独立 reads 数。

覆盖深度统一查看 [mosdepth 结果](Alignment_QC.md)，读长及质量分布图查看 [NanoPlot 报告](NanoPlot_QC.md)。

参考：[Cramino 官方文档](https://github.com/wdecoster/cramino)、[SAMtools flagstat](https://www.htslib.org/doc/samtools-flagstat.html)。
