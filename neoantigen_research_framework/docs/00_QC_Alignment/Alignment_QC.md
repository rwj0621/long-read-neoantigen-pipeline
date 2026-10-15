# HiFi 比对后质控 — SAMtools / mosdepth

[← 返回项目总 README](../../README.md)

以下根据现有脚本提炼核心命令，适用于 **PacBio HiFi WGS**。命令在 Bash 环境运行；输入均为含义明确的占位名称，使用时替换即可。这里展示方法和参数，不代表已完成实际数据运行。

| 输入 | 含义 |
|---|---|
| `raw.hifi.bam` | 原始、未比对的 PacBio HiFi BAM；多个测序批次分别质控 |
| `reference.fa` | 参考基因组 FASTA，本流程使用 GRCh38；下游保持同一参考版本 |
| `reads.fofn` | 同一样本的原始 BAM 清单，每行一个文件路径 |
| `sample` | 样本名称或输出前缀 |

**工具：** SAMtools、mosdepth。检查比对 BAM 完整性，并统计覆盖深度。

```bash
samtools quickcheck -v sample.sorted.bam

# 若比对时未生成 BAM 索引，先运行：
# samtools index -@ 4 sample.sorted.bam

mosdepth --threads 4 --no-per-base sample sample.sorted.bam
```

| 参数 / 默认设置 | 含义 |
|---|---|
| `--threads 4` | BAM 解压线程数；原脚本为 128，这里采用简化示例值 4 |
| `--no-per-base` | 不输出逐碱基深度文件，保留深度汇总与分布 |
| 默认 `--mapq 0` | 不额外设置 MAPQ 下限 |
| 默认 `--flag 1796` | 排除未比对、次级比对、QC-fail 和重复标记记录；不排除 supplementary 比对 |

主要查看 `sample.mosdepth.summary.txt` 中各参考序列及总体的平均深度，以及 `sample.mosdepth.global.dist.txt` 中达到各深度的碱基比例。统计范围取决于 BAM 头中的参考序列，本命令未限定到常染色体或目标区域。mosdepth 主要评估覆盖情况，不直接给出原始 reads 的比对率。

mosdepth 官方文档指出，超过约 4 个解压线程通常没有明显收益。软件版本需在实际运行时记录，不预填未确认版本。

参考：[SAMtools quickcheck](https://www.htslib.org/doc/samtools-quickcheck.html)、[mosdepth 官方文档](https://github.com/brentp/mosdepth)。
