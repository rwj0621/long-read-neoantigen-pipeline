# pbmm2 — HiFi 参考基因组比对

[← 返回项目总 README](../../README.md)

以下根据现有脚本提炼核心命令，适用于 **PacBio HiFi WGS**。命令在 Bash 环境运行；输入均为含义明确的占位名称，使用时替换即可。这里展示方法和参数，不代表已完成实际数据运行。

| 输入 | 含义 |
|---|---|
| `raw.hifi.bam` | 原始、未比对的 PacBio HiFi BAM；多个测序批次分别质控 |
| `reference.fa` | 参考基因组 FASTA，本流程使用 GRCh38；下游保持同一参考版本 |
| `reads.fofn` | 同一样本的原始 BAM 清单，每行一个文件路径 |
| `sample` | 样本名称或输出前缀 |

**工具：** pbmm2。多个 BAM 必须属于同一样本；不同样本分别运行。

```bash
# 同一样本有多个原始 BAM 时，逐行写入清单
printf '%s\n' raw_run1.hifi.bam raw_run2.hifi.bam > reads.fofn

# 同一参考和 preset 的索引只需建立一次
pbmm2 index reference.fa reference.hifi.mmi --preset HIFI

pbmm2 align reference.hifi.mmi reads.fofn sample.sorted.bam \
    --preset HIFI \
    --min-length 50 \
    --sample sample \
    --sort \
    --num-threads 56 \
    -J 4 \
    --log-level INFO \
    2> sample.pbmm2.log
```

只有一个原始 BAM 时，可直接用 `raw.hifi.bam` 替换 `reads.fofn`。

| 参数 | 含义 |
|---|---|
| `--preset HIFI` | HiFi 比对预设；建索引与比对使用同一预设 |
| `--min-length 50` | 最小比对读段长度为 50 bp |
| `--sample sample` | 设置所有 Read Group 的样本名称 |
| `--sort` | 输出按坐标排序的 BAM，默认同时生成 BAI 索引 |
| `--num-threads 56` | 设置比对线程数；显式指定 `-J` 时，排序线程另计 |
| `-J 4` | 设置 4 个排序线程 |
| `--log-level INFO` | 将运行信息写入日志 |

主要输出：`sample.sorted.bam`、对应的 `.bai` 索引和 `sample.pbmm2.log`。线程数按计算资源调整。

参考：[pbmm2 官方文档](https://github.com/PacificBiosciences/pbmm2)。
