---
name: read-depth-cnv-calling
description: Use when you have an aligned WGS BAM file and need to call CNVs/CNAs
  from read-depth signal, or when benchmarking CNVpytor against CNVnator call sets
  on the same alignment.
when_to_use_negative:
- Input is raw reads (FASTQ) rather than an aligned BAM — alignment must be done first
  since the pipeline tabulates read depth from mapped reads
- The goal is allele-imbalance or copy number-neutral LOH analysis rather than pure
  RD-based CNV calling (use the separate -snp/-baf workflow)
- Detecting somatic CNAs in cancer/mosaic samples requiring high accuracy — the somatic
  CNA caller in CNVpytor is only a prototype
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: CNVpytor
  role: 'Primary CNV caller: runs the read-depth binning, histogram, mean-shift partitioning,
    and calling steps from a BAM and stores results in a .pytor file'
  repo: https://github.com/abyzovlab/CNVpytor
- name: CNVnator
  role: Reference RD-based CNV caller run in parallel on the same BAM with matching
    bin sizes to benchmark CNVpytor calls via reciprocal overlap
- name: Python
  role: Implementation language of CNVpytor; can also be used for harmonizing call
    sets and computing reciprocal overlap between the two callers' outputs
  repo: https://github.com/abyzovlab/CNVpytor
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giab074
    title: 'CNVpytor: a tool for copy number variation detection and analysis from
      read depth and allele imbalance in whole-genome sequencing'
  source_content_hash: sha256:cc5b4aaa81cb96143db192bab2a7e6cab7b80937b2cd38789476ba9b3f471816
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/abyzovlab/CNVpytor
  tools:
  - CNVnator
  - CNVpytor
  - Python
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# read-depth-cnv-calling

## Summary

Detect copy number variations (CNVs) from whole-genome sequencing read depth using CNVpytor (a Python reimplementation and extension of CNVnator), running the RD binning → histogram → partition → call pipeline at bin sizes 1000, 10000, and 100000, with outputs stored in an HDF5-based .pytor file.

## When to use

Use when you have an aligned WGS BAM file and need to call CNVs/CNAs from read-depth signal, or when benchmarking CNVpytor against CNVnator call sets on the same alignment.

## When NOT to use

- Input is raw reads (FASTQ) rather than an aligned BAM — alignment must be done first since the pipeline tabulates read depth from mapped reads
- The goal is allele-imbalance or copy number-neutral LOH analysis rather than pure RD-based CNV calling (use the separate -snp/-baf workflow)
- Detecting somatic CNAs in cancer/mosaic samples requiring high accuracy — the somatic CNA caller in CNVpytor is only a prototype

## Inputs

- Aligned whole-genome sequencing BAM file (e.g., Platinum Genomes CEPH trio dataset alignment)
- Bin sizes to analyze (1000, 10000, 100000 bp)
- Reference genome build (GRCh37/hg19 or GRCh38) for coordinate harmonization
- Optional: VCF file with SNPs and single-letter indels for B-allele frequency analysis

## Outputs

- HDF5-based .pytor file storing RD signal compressed to 100-bp bins
- CNV call sets (deleted/duplicated regions with coordinates, e-values, Q0, pN, dG metrics) at each bin size
- Per-region reciprocal-overlap comparison table (TSV via 'set print_filename output.tsv; print calls') when benchmarking two callers
- Overall fraction of overlapping called regions between CNVnator and CNVpytor

## How to apply

Run the CNVpytor read-depth pipeline on the input BAM: 'cnvpytor -root file.pytor -rd file.bam' to tabulate read depth, then '-his', '-partition', and '-call' at bin sizes 1000, 10000, and 100000. Optionally filter calls in view mode using the five parameters (CNV size via 'set size_range', e-val1 via 'set p_range', q0 via 'set Q0_range', pN, and dG) before exporting. For benchmarking against CNVnator, run the CNVnator RD pipeline on the same BAM with matching bin sizes, harmonize both call sets to the same genome build and coordinate system, compute reciprocal overlap per CNV region, and compare the overall overlap fraction against the reported >99%. CNVpytor is preferred for production because it parses alignment files 2–20 times faster and produces 20–50 times smaller intermediate files than CNVnator.

## Related tools

- **CNVpytor** (Primary CNV caller: runs the read-depth binning, histogram, mean-shift partitioning, and calling steps from a BAM and stores results in a .pytor file) — https://github.com/abyzovlab/CNVpytor
- **CNVnator** (Reference RD-based CNV caller run in parallel on the same BAM with matching bin sizes to benchmark CNVpytor calls via reciprocal overlap)
- **Python** (Implementation language of CNVpytor; can also be used for harmonizing call sets and computing reciprocal overlap between the two callers' outputs) — https://github.com/abyzovlab/CNVpytor

## Examples

```
cnvpytor -root file.pytor -rd file.bam; cnvpytor -root file.pytor -his 1000 10000 100000; cnvpytor -root file.pytor -partition 1000 10000 100000; cnvpytor -root file.pytor -call 1000 10000 100000
```

## Evaluation signals

- Called regions from CNVnator and CNVpytor on the same BAM overlap by >99% — a substantially lower fraction indicates misconfigured parameters or build/coordinate mismatch
- The .pytor file is created and RD signal is stored using 100-bp bins as expected by the HDF5 compression scheme
- Calls are produced at each of the three bin sizes (1000, 10000, 100000) and per-call metrics (e-val1, q0, pN, dG) are populated
- Per-region comparison table exports successfully with 'set print_filename output.tsv; print calls' and each region has a match/mismatch status from reciprocal-overlap computation
- Both call sets are verified to be on the same genome build and coordinate system before overlap computation

## Limitations

- CNVpytor is significantly faster than CNVnator (2–20x for parsing alignment files) with 20–50x smaller intermediate files, so runtimes/storage differ between the two callers being compared
- In spite of slightly different implementations, a small fraction (<1%) of called regions may differ between CNVnator and CNVpytor
- A more accurate approach for discovering cancer and mosaic CNAs is under development and not yet available; the somatic CNA caller is only a prototype
- Some dependencies (e.g., ROOT for CNVnator root import/export, pyBigWig for JBrowse export) are optional and their absence limits interoperability

## Evidence

- [readme] RD pipeline commands: "cnvpytor -root file.pytor -rd file.bam
> cnvpytor -root file.pytor -his 1000 10000 100000
> cnvpytor -root file.pytor -partition 1000 10000 100000"
- [discussion] Overlap finding: "In spite of slightly different implementation, called regions by CNVnator and CNVpytor overlap by >99%."
- [abstract] Performance advantage: "CNVpytor is significantly faster than CNVnator—particularly for parsing alignment files (2–20 times faster)"
- [discussion] Storage scheme: "Additional compression is obtained by storing RD signal using 100-bp bins."
- [discussion] Filter parameters: "There are 5 parameters in viewer mode used for filtering calls: CNV size, e-val1, q0, pN, and dG."
