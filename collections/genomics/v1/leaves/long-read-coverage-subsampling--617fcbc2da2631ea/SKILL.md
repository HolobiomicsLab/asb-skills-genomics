---
name: long-read-coverage-subsampling
description: Use when assessing whether an SV caller retains recall/precision/F1 and
  genotype accuracy as long-read coverage drops, e.g. subsampling HG002 Hifi/CLR/ONT
  BAMs to 10x to compare against assembly-based baselines.
when_to_use_negative:
- The goal is absolute SV detection accuracy at native coverage rather than sensitivity
  to depth — run each caller on the full BAMs instead.
- 'Applying VolcanoSV to sex chromosomes: the pipeline is designed only for autosomes
  because haplotype phasing differs for sex chromosomes.'
- Evaluating small indel calling on ONT data, where assembly-based small indel detection
  was ineffective for all three tools (low precision).
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: rasusa
  role: Subsamples long-read datasets to fixed coverage levels (40x-5x) for robustness
    testing
- name: VolcanoSV
  role: Assembly-based SV caller under evaluation; run in WGS mode per chromosome
    with platform-specific -dtype and merged via Merge_VCF.py
  repo: https://github.com/maiziezhoulab/VolcanoSV
- name: Dipcall
  role: Assembly-based baseline SV caller run on the same subsampled BAMs
  repo: https://github.com/lh3/dipcall
- name: SVIM-asm
  role: Assembly-based baseline SV caller run on the same subsampled BAMs
  repo: https://github.com/eldariont/svim-asm
- name: PAV
  role: Assembly-based baseline SV caller run on the same subsampled BAMs
  repo: https://github.com/EichlerLab/pav
- name: Truvari
  role: Benchmarks each caller's DEL/INS VCFs against the GIAB HG002 gold standard
    to produce recall, precision, F1, and genotype accuracy
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-51282-0
    title: VolcanoSV enables accurate and robust structural variant calling in diploid
      genomes from single-molecule long read sequencing
  source_content_hash: sha256:738fb0ddcbcffc8201600adce0802f1235ab44b9a263208dddedcd0ecb62b957
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/maiziezhoulab/VolcanoSV
  tools:
  - Dipcall
  - PAV
  - SVIM-asm
  - Truvari
  - VolcanoSV
  - rasusa
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# long-read-coverage-subsampling

## Summary

Downsamples HG002 long-read BAMs (Hifi_L1-L5, CLR_L1-L3, ONT_L1-L6) with rasusa to graded coverage levels (40x, 30x, 20x, 10x, 5x) and benchmarks assembly-based SV callers (VolcanoSV, PAV, SVIM-asm, Dipcall) against the GIAB HG002 gold standard with Truvari to quantify robustness at low depth.

## When to use

Use when assessing whether an SV caller retains recall/precision/F1 and genotype accuracy as long-read coverage drops, e.g. subsampling HG002 Hifi/CLR/ONT BAMs to 10x to compare against assembly-based baselines.

## When NOT to use

- The goal is absolute SV detection accuracy at native coverage rather than sensitivity to depth — run each caller on the full BAMs instead.
- Applying VolcanoSV to sex chromosomes: the pipeline is designed only for autosomes because haplotype phasing differs for sex chromosomes.
- Evaluating small indel calling on ONT data, where assembly-based small indel detection was ineffective for all three tools (low precision).

## Inputs

- HG002 long-read BAM files (Hifi_L1-L5, CLR_L1-L3, ONT_L1-L6)
- Reference genome FASTA (hg19/GRCh38) with index
- Target coverage levels (40x, 30x, 20x, 10x, 5x)
- GIAB HG002 SV gold standard VCF and high-confidence regions BED

## Outputs

- Subsampled read/BAM sets per coverage level per dataset
- Per-tool DEL/INS VCFs for each subsampled dataset
- Recall, precision, F1, and genotype accuracy per tool, dataset, SV type, and coverage level
- Consolidated metrics table (sv_benchmark_metrics.csv)

## How to apply

Obtain the HG002 long-read BAMs (Hifi_L1-L5 at ~56x, 30x, 34x, 28x, 41x; CLR_L1-L3 at 89x, 65x, 29x; ONT_L1-L6 at ~48-57x) plus the hg19/GRCh38 reference, then subsample read sets with rasusa v0.6.0 to 40x, 30x, 20x, 10x, and 5x coverage. Run each caller on every subsampled set: for VolcanoSV in WGS mode, build a chrom_list.txt of contig names matching the reference, run volcanosv-asm.py per chromosome with the platform-appropriate -dtype (Hifi/CLR/ONT), then run volcanosv-vc-large-indel.py, volcanosv-vc-complex-sv.py, and volcanosv-vc-small-indel.py with -cl and merge VCFs via Merge_VCF.py; run Dipcall, SVIM-asm, and PAV on the same subsampled BAMs. Benchmark every DEL/INS VCF against the GIAB HG002 SV gold standard using Truvari v4.0.0 in high-confidence regions with p=0.5, P=0.5, r=500, O=0.01 (r=500, S=30 per the task card). Extract recall, precision, F1, and genotype accuracy per tool, dataset, SV type, and coverage level — explicitly including the 10x runs — and compile them into a single table (sv_benchmark_metrics.csv). Robustness is judged by how little F1 degrades as coverage falls: VolcanoSV preserved high F1 for both insertions and deletions even at 10x on Hifi_L1, unlike PAV, SVIM-asm, and Dipcall.

## Related tools

- **rasusa** (Subsamples long-read datasets to fixed coverage levels (40x-5x) for robustness testing)
- **VolcanoSV** (Assembly-based SV caller under evaluation; run in WGS mode per chromosome with platform-specific -dtype and merged via Merge_VCF.py) — https://github.com/maiziezhoulab/VolcanoSV
- **Dipcall** (Assembly-based baseline SV caller run on the same subsampled BAMs) — https://github.com/lh3/dipcall
- **SVIM-asm** (Assembly-based baseline SV caller run on the same subsampled BAMs) — https://github.com/eldariont/svim-asm
- **PAV** (Assembly-based baseline SV caller run on the same subsampled BAMs) — https://github.com/EichlerLab/pav
- **Truvari** (Benchmarks each caller's DEL/INS VCFs against the GIAB HG002 gold standard to produce recall, precision, F1, and genotype accuracy)

## Examples

```
rasusa -i Hifi_L1.fastq.gz -c 10x -g 3.1Gb -t 2 -o Hifi_L1_10x.fq.gz && python volcanosv-asm.py -bam Hifi_L1_10x.bam -ref genome.fa -chr chr1 -dtype Hifi
```

## Evaluation signals

- Per-dataset subsampled BAM depths match the target coverages (40x, 30x, 20x, 10x, 5x) as computed by read-depth summation.
- Truvari runs use the specified parameter set (p=0.5, P=0.5, r=500, O=0.01) restricted to GIAB high-confidence regions for all tools.
- Metrics table contains rows for every tool × dataset × SV type × coverage level, with 10x entries explicitly present.
- Reproduced pattern: VolcanoSV retains high F1 for insertions and deletions at 10x on Hifi_L1 while baseline assembly callers degrade.
- Sanity check against reported ranges: VolcanoSV average F1 across Hifi datasets of 91.03% (INS) and 94.19% (DEL) with 98.32%/99.01% genotyping accuracy.

## Limitations

- The GIAB gold standard SV callset is not complete and could contain false positives, biasing recall/precision estimates.
- With respect to insertion/deletion recall, VolcanoSV achieved the best recall in only 3-4 out of 6 ONT datasets.
- Detecting small indels via assembly was ineffective for ONT data (low precision for all three assembly-based tools).

## Evidence

- [results] Datasets subsampled with rasusa to graded coverage levels for robustness testing: "All three datasets were subsampled to 40x, 30x, 20x, 10x, and 5x coverage using rasusa (v0.6.0)"
- [results] Truvari benchmarking parameters used to score SV calls against the GIAB gold standard: "The following metric/parameter setting was used in Truvari: p = 0.5, P = 0.5, r = 500, O = 0.01"
- [intro] Reported superior performance across datasets including low-coverage 10x data: "It outperforms state-of-the-art assembly-based SV callers, demonstrating higher recall, precision, F1 scores, and genotype accuracy across a diverse range of datasets, including low-coverage (10x)"
- [readme] VolcanoSV is restricted to autosomes because sex-chromosome phasing differs: "VolcanoSV is designed only for autosome chromosomes since the phasing procedure for sex chromosomes is very different!"
