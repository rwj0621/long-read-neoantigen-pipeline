# HiFi 原始数据质控

[← 返回项目总 README](../../README.md)

以下根据现有脚本提炼核心命令，适用于 **PacBio HiFi WGS**。命令在 Bash 环境运行；输入均为含义明确的占位名称，使用时替换即可。这里展示方法和参数，不代表已完成实际数据运行。

| 输入 | 含义 |
|---|---|
| `raw.hifi.bam` | 原始、未比对的 PacBio HiFi BAM；多个测序批次分别质控 |
| `reference.fa` | 参考基因组 FASTA，本流程使用 GRCh38；下游保持同一参考版本 |
| `reads.fofn` | 同一样本的原始 BAM 清单，每行一个文件路径 |
| `sample` | 样本名称或输出前缀 |

**工具：** SAMtools、Python（pysam、NumPy）。本步骤评估文件完整性、数据量、读长和读段质量，不执行质量过滤。

**BAM 完整性检查：**

```bash
samtools quickcheck -u -v raw.hifi.bam
samtools view -H raw.hifi.bam > raw.header.sam
samtools view -c raw.hifi.bam > raw.read_count.txt
```

| 命令 / 参数 | 含义 |
|---|---|
| `quickcheck -u -v` | 按未比对 BAM 检查文件头和文件末尾；`-u` 允许无参考序列，`-v` 报告异常文件 |
| `view -H` | 导出文件头，检查 PacBio `@RG` 信息并记录 `@SQ` 情况 |
| `view -c` | 完整扫描 BAM 并统计记录数，补充 quickcheck 不检查文件中间内容的限制 |

**数据量与读长统计：**

```bash
samtools stats -@ 32 raw.hifi.bam > raw.stats.txt
```

`-@ 32` 设置输入解压线程数。结果中的 `SN` 提供 reads 数、总碱基数、平均及最大读长；`RL` 提供读长频数分布。现有脚本利用 `RL` 进一步计算中位读长、N50、最短读长及各长度区间占比，无需再次扫描 BAM。

- 数据量（Gb）= 总碱基数 / 10⁹。
- N50：按读长从长到短累计碱基数，达到总碱基数 50% 时对应的读长。
- 同一样本多个批次可合并统计；中位数和 N50 应从合并后的长度分布重新计算。

**读段质量统计：** 从原始 BAM 的 `rq`（预测平均读段准确率）和 `np`（完整通过次数）标签提取指标，核心代码如下。

```bash
python - raw.hifi.bam > raw.quality.txt <<'PY'
import sys
import numpy as np
import pysam

rq, passes, total = [], [], 0
with pysam.AlignmentFile(sys.argv[1], "rb", check_sq=False,
                         require_index=False) as bam:
    for read in bam:
        total += 1
        if read.has_tag("rq"):
            rq.append(read.get_tag("rq"))
        if read.has_tag("np"):
            passes.append(read.get_tag("np"))

rq = np.asarray(rq, dtype=np.float32)
print("rq_tag_percent", 100 * len(rq) / total if total else "NA")
print("np_tag_percent", 100 * len(passes) / total if total else "NA")
if len(rq):
    print("median_rq", np.median(rq))
    qv = np.full(len(rq), np.nan)
    valid = (rq >= 0) & (rq < 1)
    qv[valid] = -10 * np.log10(1 - rq[valid].astype(np.float64))
    qv[rq >= 1] = 60  # 沿用原脚本的展示约定
    print("median_QV", np.nanmedian(qv))
    for q, cutoff in [(20, .99), (30, .999), (40, .9999), (50, .99999)]:
        print(f"Q{q}+_percent", 100 * np.mean(rq >= np.float32(cutoff)))
if passes:
    print("mean_passes", np.mean(passes))
    print("median_passes", np.median(passes))
PY
```

| 设置 / 指标 | 说明 |
|---|---|
| `check_sq=False`、`require_index=False` | 顺序读取未比对 BAM，不要求参考序列信息或 BAM 索引 |
| `QV = -10 × log10(1 - rq)` | 每条 read 的预测质量值；`rq ≥ 1` 时以 Q60 展示，不代表实测 Q60 |
| Q20+ / Q30+ / Q40+ / Q50+ | `rq` 分别达到 0.99 / 0.999 / 0.9999 / 0.99999 的 reads 比例；分母为具有 `rq` 标签的 reads，并非碱基比例 |
| `np.float32` | `rq` 和比较阈值均使用单精度，保持与 BAM 的 `rq:f` 类型一致 |
| `np` | 汇总通过次数的均值、中位数；标签完整率的分母为全部 BAM 记录 |

参考：[SAMtools quickcheck](https://www.htslib.org/doc/samtools-quickcheck.html)、[SAMtools stats](https://www.htslib.org/doc/samtools-stats.html)。
