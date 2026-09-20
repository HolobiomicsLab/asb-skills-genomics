---
name: ena-read-dataset-preprocessing
description: Use when preparing basecalled Nanopore FASTQ + FAST5 signal data from
  ENA accessions for signal-level methylation calling, needing a read-to-signal index
  and minimap2/samtools-sorted BAM per dataset.
when_to_use_negative:
- The data are already indexed and aligned (a sorted, indexed BAM and caller index
  exist) — re-running preprocessing only wastes time before benchmarking.
- The input is SLOW5/POD5 signal data rather than FAST5 — f5c's recommended path uses
  slow5tools/blue-crab conversion and BLOW5 format instead.
- The task is methylation calling performance comparison itself rather than input
  preparation — use the benchmarking skill on the preprocessed outputs.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: minimap2
  role: Aligns basecalled Nanopore reads to the reference genome with '-a -x map-ont'
    before sorting
  repo: https://github.com/lh3/minimap2
- name: samtools v1.x
  role: Sorts minimap2 SAM output into a sorted BAM and indexes it for call-methylation
    input
  repo: https://www.htslib.org/
- name: Galaxy-methyl
  role: Provides the index step ('./Galaxy-methyl index -d <fast5_dir> <fastq.gz>')
    linking read ids to FAST5 signal data
  repo: https://github.com/fengyilin118/Galaxy-methyl
- name: Nanopolish
  role: Alternative index step creating the read-id-to-signal index consumed by nanopolish
    call-methylation
  repo: https://github.com/jts/nanopolish
- name: f5c
  role: Alternative index step ('f5c index') producing the same read-to-signal index
    for CPU-baseline benchmarking
  repo: https://github.com/hasindu2008/f5c
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbac088
    title: GPU-accelerated and pipelined methylation calling
  source_content_hash: sha256:faf308badca79459d732e042fbc5ba06141117cffec3733cc24ce64a84424813
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - Galaxy-methyl
  - Nanopolish
  - f5c
  - minimap2
  - samtools
  - samtools v1.x
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# ena-read-dataset-preprocessing

## Summary

Downloads the six ENA Nanopore datasets (ERR2184700/ERR2184710/ERR2184719 from PRJEB23027; ERR1676719/ERR1676724/ERR167672 from PRJEB13021), builds signal-level indexes, and produces sorted, indexed BAM alignments with minimap2 and samtools so methylation callers (Nanopolish, f5c, Galaxy-methyl) can be benchmarked on identical inputs.

## When to use

Use when preparing basecalled Nanopore FASTQ + FAST5 signal data from ENA accessions for signal-level methylation calling, needing a read-to-signal index and minimap2/samtools-sorted BAM per dataset.

## When NOT to use

- The data are already indexed and aligned (a sorted, indexed BAM and caller index exist) — re-running preprocessing only wastes time before benchmarking.
- The input is SLOW5/POD5 signal data rather than FAST5 — f5c's recommended path uses slow5tools/blue-crab conversion and BLOW5 format instead.
- The task is methylation calling performance comparison itself rather than input preparation — use the benchmarking skill on the preprocessed outputs.

## Inputs

- ENA Nanopore read accessions ERR2184700, ERR2184710, ERR2184719 (PRJEB23027, D1-D3)
- ENA Nanopore read accessions ERR1676719, ERR1676724, ERR167672 (PRJEB13021, D4-D6)
- Basecalled FASTQ files (.fastq.gz) per dataset
- Raw signal FAST5 files per dataset
- Reference genomes: hg38 FASTA (D1-D3, D5-D6), Escherichia_coli_str_k_12_substr_mg1655.GCA_000005845.2 FASTA (D4)

## Outputs

- Index file linking read ids with their signal-level FAST5 data per dataset
- Sorted BAM alignment of basecalled reads to the matching reference genome
- BAM index (.bai) produced by samtools index
- Preprocessing artifacts ready as call-methylation inputs (-r fastq, -b sorted.bam, -g reference.fasta)

## How to apply

Download the six ENA read sets (ERR2184700, ERR2184710, ERR2184719 under PRJEB23027; ERR1676719, ERR1676724, ERR167672 under PRJEB13021) and the matching references: hg38 FASTA for D1-D3, the Escherichia coli str. k_12 substr. MG1655 genome (GCA_000005845.2) for D4, and GRCh38.primary_assembly.genome.fa for D5-D6. For each dataset, run the caller's index step (e.g. './Galaxy-methyl index -d <fast5_dir> <fastq.gz>') to create an index file linking read ids with their signal-level data in the FAST5 files. Align basecalled reads to the matching reference with 'minimap2 -a -x map-ont', piping the output directly into 'samtools sort' to get a sorted BAM, then run 'samtools index'. The resulting indexed FASTQ/FAST5 plus sorted BAM are the canonical inputs consumed by nanopolish/f5c/Galaxy-methyl call-methylation, ensuring all tools are timed on identical preprocessed inputs.

## Related tools

- **minimap2** (Aligns basecalled Nanopore reads to the reference genome with '-a -x map-ont' before sorting) — https://github.com/lh3/minimap2
- **samtools v1.x** (Sorts minimap2 SAM output into a sorted BAM and indexes it for call-methylation input) — https://www.htslib.org/
- **Galaxy-methyl** (Provides the index step ('./Galaxy-methyl index -d <fast5_dir> <fastq.gz>') linking read ids to FAST5 signal data) — https://github.com/fengyilin118/Galaxy-methyl
- **Nanopolish** (Alternative index step creating the read-id-to-signal index consumed by nanopolish call-methylation) — https://github.com/jts/nanopolish
- **f5c** (Alternative index step ('f5c index') producing the same read-to-signal index for CPU-baseline benchmarking) — https://github.com/hasindu2008/f5c

## Examples

```
./Galaxy-methyl index -d <fast5_dir> <fastq.gz> && minimap2 -a -x map-ont <reference.fasta> <fastq.gz> | samtools sort -o sorted.bam && samtools index sorted.bam
```

## Evaluation signals

- An index file exists per dataset that links each read id with its signal-level data in the FAST5 files before any call-methylation run.
- The minimap2 output is piped directly into samtools sort, yielding a coordinate-sorted BAM that passes 'samtools quickcheck' and has a matching .bai index.
- Each dataset was aligned against its matching reference (hg38 for D1-D3 and D5-D6, E. coli K-12 MG1655 GCA_000005845.2 for D4) — not a mismatched reference.
- All three callers (Nanopolish, f5c, Galaxy-methyl) are invoked on the same indexed FASTQ/FAST5 and the same sorted BAM, so timing comparisons are on identical inputs.
- call-methylation accepts the preprocessed inputs without error using the documented flags (-r <fastq.gz> -b <sorted.bam> -g <reference.fasta>, -t 4 -B 4.0M).

## Limitations

- Presently Nanopolish does not support R10.4 flowcells for variant and methylation calling, so this preprocessing path targets chemistries where signal-level analysis is still required.
- FAST5 is not f5c's recommended format — BLOW5 (via slow5tools/blue-crab conversion) is recommended for best performance, and f5c v1.2+ requires --pore r10 for FAST5 R10.4.1 input.
- Once a minimap2 index is built with -d, indexing parameters such as -k, -w, -H and -I cannot be changed during mapping, so separate indexes are needed for different data types.

## Evidence

- [methods] create an index file linking read ids with signal-level data in FAST5 files (index): "create an index file that links read ids with their signal-level data in the FAST5 files"
- [methods] minimap2 alignment piped into samtools sort: "align the basecalled reads to the reference genome.  minimap2 is used to map reads to the human genome"
- [methods] sorted BAM via samtools sort: "The output is piped directly into samtools sort to get a sorted bam file"
- [readme] f5c index/call-methylation workflow: [evidence span withheld — anchor status: external_doc]
- [readme] Nanopolish R10 support limitation: [evidence span withheld — anchor status: external_doc]
