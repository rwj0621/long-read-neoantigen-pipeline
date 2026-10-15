# HiFi 参考基因组比对

[← 返回项目总 README](../../README.md)

使用 pbmm2 将 PacBio HiFi reads 比对至 GRCh38 参考基因组，并输出按坐标排序的 BAM，供后续变异检测使用。建索引与比对使用相同的 HiFi 预设。

## 核心命令

```bash
pbmm2 index reference.fa reference.hifi.mmi --preset HIFI

pbmm2 align reference.hifi.mmi reads.fofn sample.sorted.bam \
    --preset HIFI --min-length 50 --sample sample \
    --sort --num-threads 56 -J 4 --log-level INFO
```

`reference.fa` 为参考基因组；`reads.fofn` 为同一样本的原始 BAM 清单，每行一个文件路径。单个 BAM 可直接作为输入。参考索引可复用，不同样本分别比对。

## 关键参数

| 参数 | 含义 |
|---|---|
| `--preset HIFI` | 使用 HiFi 比对预设 |
| `--min-length 50` | 最小比对读段长度为 50 bp |
| `--sample sample` | 设置 Read Group 中的样本名称 |
| `--sort` | 输出按坐标排序的 BAM，默认生成 BAI 索引 |
| `--num-threads 56` | 比对线程数；显式指定 `-J` 时，排序线程另计 |
| `-J 4` | 排序线程数 |
| `--log-level INFO` | 输出运行信息 |

主要输出为 `sample.sorted.bam` 及对应的 `.bai` 索引。线程数可按计算资源调整，参考基因组版本应与下游分析一致。

参考：[pbmm2 官方文档](https://github.com/PacificBiosciences/pbmm2)。
