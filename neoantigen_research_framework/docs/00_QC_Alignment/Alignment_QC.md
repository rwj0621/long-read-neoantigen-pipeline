# HiFi 比对后质控

[← 返回项目总 README](../../README.md)

使用 SAMtools 检查比对 BAM 的完整性，使用 mosdepth 评估覆盖深度及分布。输入为按坐标排序且具有索引的 BAM。

## 核心命令

```bash
samtools quickcheck -v sample.sorted.bam
mosdepth --threads 4 --no-per-base sample sample.sorted.bam
```

`sample` 为输出前缀。若比对阶段未生成索引，可使用 `samtools index sample.sorted.bam` 建立索引。

## 关键参数

| 参数 / 默认设置 | 含义 |
|---|---|
| `quickcheck -v` | 检查 BAM 文件头和文件末尾，报告异常文件 |
| `--threads 4` | BAM 解压线程数，可按计算资源调整 |
| `--no-per-base` | 保留深度汇总和分布，不输出逐碱基深度文件 |
| 默认 `--mapq 0` | 不额外设置 MAPQ 下限 |
| 默认 `--flag 1796` | 排除未比对、次级比对、QC-fail 和重复标记记录，保留 supplementary 比对 |

## 主要指标

| 输出 | 内容 |
|---|---|
| `sample.mosdepth.summary.txt` | 各参考序列及总体的平均覆盖深度 |
| `sample.mosdepth.global.dist.txt` | 达到各覆盖深度的碱基比例 |

统计范围由 BAM 头中的参考序列决定，未额外限定常染色体或目标区域。覆盖深度反映参考序列上的数据覆盖情况，不能直接替代原始 reads 比对率。

参考：[SAMtools quickcheck](https://www.htslib.org/doc/samtools-quickcheck.html)、[mosdepth 官方文档](https://github.com/brentp/mosdepth)。
