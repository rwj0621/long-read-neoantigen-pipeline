# HiFi 原始数据质控

[← 返回项目总 README](../../README.md)

使用 SAMtools 和 Python（pysam、NumPy）评估 PacBio HiFi WGS 数据的完整性、数据量、读长和读段质量。输入为原始未比对 BAM；多个测序批次分别统计后，再汇总同一样本的指标。本步骤不进行 reads 过滤。

## BAM 完整性

```bash
samtools quickcheck \
    -u \
    -v \
    raw.hifi.bam

samtools view \
    -H \
    raw.hifi.bam \
    > raw.header.sam

samtools view \
    -c \
    raw.hifi.bam \
    > raw.read_count.txt
```

| 命令 / 参数 | 作用 |
|---|---|
| `quickcheck -u -v` | 检查未比对 BAM 的文件头和文件末尾，报告异常文件 |
| `view -H` | 检查文件头中的 PacBio Read Group（`@RG`）及参考序列信息（`@SQ`） |
| `view -c` | 完整扫描 BAM 并统计记录数，补充 quickcheck 对文件中间内容的检查 |

## 数据量与读长

```bash
samtools stats \
    -@ 32 \
    raw.hifi.bam \
    > raw.stats.txt
```

`-@ 32` 指定输入解压线程数，可按计算资源调整。解析统计结果中的 `SN`（汇总指标）和 `RL`（读长频数分布），获得以下指标：

| 指标 | 计算方法 |
|---|---|
| Reads 数、总碱基数 | 从 `SN` 提取 |
| 数据量（Gb） | 总碱基数 / 10⁹ |
| 平均、中位、最短及最长读长 | 根据读长频数分布计算 |
| Read N50 | 按读长从长到短累计碱基数，达到总量 50% 时对应的读长 |
| 读长分布 | 按长度区间统计 reads 数及碱基数占比 |

多个批次汇总时，中位读长和 N50 从合并后的频数分布重新计算。

## 读段质量

使用 pysam 顺序读取 BAM 中的 `rq`（预测平均读段准确率）和 `np`（完整通过次数）标签，使用 NumPy 汇总质量指标。未比对 BAM 无需建立索引。

| 指标 | 计算方法 / 设置 |
|---|---|
| 标签完整率 | 分别统计具有 `rq`、`np` 标签的记录占全部 BAM 记录的比例 |
| 读段准确率 | `rq` 的中位数 |
| 读段质量 QV | 每条 read 按 `QV = -10 × log10(1 - rq)` 转换后，取中位数 |
| Q20+ / Q30+ / Q40+ / Q50+ | 统计 `rq ≥ 0.99 / 0.999 / 0.9999 / 0.99999` 的 reads 比例 |
| 测序通过次数 | `np` 的均值、中位数及分布 |

质量阈值比例以具有 `rq` 标签的 reads 为分母，表示读段比例。`rq` 与阈值均按 float32 比较，以保持与 BAM 的 `rq:f` 类型一致。对于 `rq ≥ 1` 的记录，QV 汇总采用 Q60 作为展示约定。

参考：[SAMtools quickcheck](https://www.htslib.org/doc/samtools-quickcheck.html)、[SAMtools stats](https://www.htslib.org/doc/samtools-stats.html)。
