---
name: mapper-benchmark-precision-sensitivity-evaluation
description: Use when comparing read mappers for long, error-prone nanopore data and
  you need downstream-task metrics — SNV-calling precision and sensitivity against
  a truth VCF — rather than alignment-level accuracy alone.
when_to_use_negative:
- Input data are short Illumina reads or the goal is consensus error correction rather
  than comparing mappers via variant calling
- No truth set (known SNV calls) exists for the sample, making precision/sensitivity
  computation impossible
- The task is de novo assembly or read overlapping, where mapper choice is evaluated
  by overlap quality instead of variant metrics
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
tools:
- name: GraphMap
  role: Primary long-read mapper benchmarked; its alignments yield the highest SNV
    precision (~96%)
  repo: https://github.com/isovic/graphmap
- name: BWA-MEM
  role: Comparison mapper whose variant precision GraphMap improves by 76%
- name: LAST
  role: Comparison mapper in the per-mapper precision/sensitivity table
- name: BLASR
  role: Comparison mapper for the nanopore amplicon reads
- name: DALIGNER
  role: Next best mapper; reference for the 15% sensitivity delta and 93% precision
- name: LoFreq
  role: SNV caller run on GraphMap and DALIGNER alignments
- name: marginAlign
  role: Package supplying marginCaller, the nanopore-tailored SNV caller run on GraphMap
    alignments (36% precision)
  repo: https://github.com/isovic/marginAlign
- name: GCC/G++
  role: Compiler toolchain required to build GraphMap (>= 4.7)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/ncomms11307
    title: Fast and sensitive mapping of nanopore sequencing reads with GraphMap
  source_content_hash: sha256:c9c6cbef4f801053833d5cf1a9257d770c8d2dd52c2123aa8910af2cded81406
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/isovic/graphmap
  tools:
  - BLASR
  - BWA-MEM
  - DALIGNER
  - GCC/G++
  - GraphMap
  - LAST
  - LoFreq
  - marginAlign
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 21
---

# mapper-benchmark-precision-sensitivity-evaluation

## Summary

Benchmarks long-read mappers (GraphMap, BWA-MEM, LAST, BLASR, DALIGNER) by routing their alignments into SNV calling and scoring each mapper's variant calls against a truth set for precision and sensitivity, the downstream measure used in the GraphMap paper to show 96% SNV precision and a 15% sensitivity gain over the next best mapper.

## When to use

Use when comparing read mappers for long, error-prone nanopore data and you need downstream-task metrics — SNV-calling precision and sensitivity against a truth VCF — rather than alignment-level accuracy alone.

## When NOT to use

- Input data are short Illumina reads or the goal is consensus error correction rather than comparing mappers via variant calling
- No truth set (known SNV calls) exists for the sample, making precision/sensitivity computation impossible
- The task is de novo assembly or read overlapping, where mapper choice is evaluated by overlap quality instead of variant metrics

## Inputs

- Targeted Oxford Nanopore amplicon reads (FASTQ; CYP2D6, HLA-A, HLA-B from NA12878)
- Reference sequences hg19 chr6 and chr22 (FASTA)
- Amplicon SNV truth set (VCF) for the sequenced sample

## Outputs

- One alignment file (SAM) per mapper (GraphMap, BWA-MEM, LAST, BLASR, DALIGNER)
- Per-mapper SNV call sets (VCF) from LoFreq and marginCaller
- snv_precision_sensitivity.csv with per-mapper precision and sensitivity values

## How to apply

Build each mapper from source (for GraphMap: `make modules` then `make` with GCC/G++ >= 4.7, overriding via GCC=/usr/local/bin/g++ make if needed), then map the same amplicon FASTQ inputs to the relevant reference sequences (hg19 chr6/chr22 for the CYP2D6/HLA-A/HLA-B NA12878 nanopore data), producing one alignment file per mapper. From each alignment set, call SNVs with a common caller (LoFreq) plus a nanopore-tailored caller (marginCaller from marginAlign) so that mapper quality, not caller choice, drives differences. Compare every variant set against the amplicon truth set (e.g. Ammar et al.) and compute per-mapper precision and sensitivity, recording them in a csv such as snv_precision_sensitivity.csv. The rationale is that mapping sensitivity propagates to variant sensitivity, so variant-level metrics discriminate mappers on high-error nanopore reads where callers were reported to fail.

## Related tools

- **GraphMap** (Primary long-read mapper benchmarked; its alignments yield the highest SNV precision (~96%)) — https://github.com/isovic/graphmap
- **BWA-MEM** (Comparison mapper whose variant precision GraphMap improves by 76%)
- **LAST** (Comparison mapper in the per-mapper precision/sensitivity table)
- **BLASR** (Comparison mapper for the nanopore amplicon reads)
- **DALIGNER** (Next best mapper; reference for the 15% sensitivity delta and 93% precision)
- **LoFreq** (SNV caller run on GraphMap and DALIGNER alignments)
- **marginAlign** (Package supplying marginCaller, the nanopore-tailored SNV caller run on GraphMap alignments (36% precision)) — https://github.com/isovic/marginAlign
- **GCC/G++** (Compiler toolchain required to build GraphMap (>= 4.7))

## Examples

```
git clone https://github.com/isovic/graphmap.git; cd graphmap; make modules; make; ./bin/Linux-x64/graphmap align -r hg19_chr6.fa -d amplicons.fastq -o graphmap.sam
```

## Evaluation signals

- snv_precision_sensitivity.csv shows GraphMap ~96% SNV precision, within tolerance of the reported figure
- GraphMap sensitivity exceeds the next best mapper (DALIGNER, 93% precision) by ~15%
- marginCaller on GraphMap alignments lands near 36% precision, matching the reported value
- One alignment file per mapper (GraphMap, BWA-MEM, LAST, BLASR, DALIGNER) exists for the same read and reference inputs
- Variant call sets are compared against the Ammar et al. amplicon truth set without mismatched coordinates or references

## Limitations

- GraphMap's hash-table index is memory-hungry: ~14x reference size for the final index (~100 GB for the human genome, peaking ~42 GB at construction), limiting large-genome mapping to large-memory machines
- The exact benchmark numbers depend on the caller; marginCaller reached only 36% precision, so caller choice strongly affects results
- Improved sensitivity benefits for consensus callers (Nanopolish, PoreSeq) were asserted but not demonstrated in the paper

## Evidence

- [abstract] SNV calling gains over DALIGNER/BWA-MEM/LAST: "single-nucleotide variant calling on the human genome with increased sensitivity (15%) over the next best mapper"
- [abstract] High precision via progressive refinement: "progressively refines candidate alignments to robustly handle potentially high-error rates and a fast graph traversal to align long reads with speed and high precision (>95%)"
- [discussion] Comparison mapper set: "is comparable to BLAST and other mappers (BWA-MEM, LAST, BLASR and DALIGNER)"
- [discussion] marginAlign as downstream beneficiary: "GraphMap mappings provided a better starting point for the realigner marginAlign2"
- [methods] Build order for GraphMap: "After running `git pull`, please `make modules` before `make`."
- [readme] GraphMap align invocation: [evidence span withheld — anchor status: external_doc]
- [discussion] Memory footprint caveat: "requirement of large-memory machines for mapping to large genomes (∼100 GB for the human genome)"
- [readme] Reproduction scripts: [evidence span withheld — anchor status: not_found]
