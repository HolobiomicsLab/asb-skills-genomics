---
name: quast-contiguity-accuracy-evaluation
description: Use when comparing de novo assemblies (e.g., HiFiCCL-Hifiasm vs Hifiasm
  alone) on low-coverage (~5x) HiFi read sets and you need objective contiguity/accuracy
  metrics against a high-quality reference assembly.
when_to_use_negative:
- Evaluating assemblies generated at normal high coverage (>25x), where the reference-guided
  low-coverage rationale and misassembly reductions of HiFiCCL do not apply
- No high-quality published reference assembly (e.g., CHM13, GRCh38) is available
  for the organism, so reference-based QUAST/NGA50 evaluation cannot be performed
- Plant genomes beyond the two samples tested in the article, since the paper notes
  broader plant evaluation has not yet been performed
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: QUAST
  role: Contiguity and misassembly evaluation of assembled contigs against high-quality
    published assemblies; source of contig size, number of contigs, MCL, NG50 and
    NGA50 metrics
- name: BUSCO
  role: Gene completeness assessment (v5.4.7) using the vertebrate conserved single-copy
    ortholog database
- name: Merqury
  role: k-mer-based completeness and QV estimation complementing reference-based QUAST
    metrics
- name: HiFiCCL
  role: Produces the reference-guided chromosome-by-chromosome assemblies being evaluated
    (default mode via hificcl.py -f <reads> -r CHM13v2.0.fasta -t 20)
  repo: https://github.com/zjjbuqi/HiFiCCL
- name: hifiasm
  role: Baseline assembler (0.19.5-r592, -t20 --primary) whose contigs serve as the
    comparison point for MCL reduction
  repo: https://github.com/chhylp123/hifiasm
- name: seqtk
  role: Downsamples HiFi read sets to ~5x coverage (v1.3-r106) before assembly and
    evaluation
  repo: https://github.com/lh3/seqtk
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/advs.202515308
    title: Reference‐Guided Chromosome‐by‐Chromosome <i>de novo</i> Assembly at Scale
      Using Low‐Coverage High‐Fidelity Long‐Reads with HiFiCCL
  source_content_hash: sha256:80d5150d3235ff77bfec202ad5fc5fb86e7abe5b914924cca0d95839fa52e3b1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zjjbuqi/HiFiCCL
  tools:
  - BUSCO
  - HiFiCCL
  - Hifiasm
  - Merqury
  - QUAST
  - hifiasm
  - minimap2
  - seqtk
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# quast-contiguity-accuracy-evaluation

## Summary

Evaluate contig-level genome assemblies produced at low HiFi coverage with QUAST v5.2.0, complemented by BUSCO 5.4.7 and Merqury, to quantify contiguity and misassembly improvements (contig size, number of contigs, misassembled contig length, NG50, NGA50).

## When to use

Use when comparing de novo assemblies (e.g., HiFiCCL-Hifiasm vs Hifiasm alone) on low-coverage (~5x) HiFi read sets and you need objective contiguity/accuracy metrics against a high-quality reference assembly.

## When NOT to use

- Evaluating assemblies generated at normal high coverage (>25x), where the reference-guided low-coverage rationale and misassembly reductions of HiFiCCL do not apply
- No high-quality published reference assembly (e.g., CHM13, GRCh38) is available for the organism, so reference-based QUAST/NGA50 evaluation cannot be performed
- Plant genomes beyond the two samples tested in the article, since the paper notes broader plant evaluation has not yet been performed

## Inputs

- Assembled contig FASTA files from each pipeline being compared (e.g., HiFiCCL-Hifiasm and Hifiasm --primary outputs)
- High-quality reference assembly FASTA (e.g., CHM13 v2.0, GRCh38) for QUAST alignment
- Downsampled ~5x HiFi read sets (FASTQ/FASTA) with assembly logs for wall-time and memory recording

## Outputs

- QUAST report with contig size, number of contigs, misassembled contig length, NG50 and NGA50 per assembly
- BUSCO 5.4.7 completeness scores using the vertebrate conserved single-copy ortholog database
- Merqury completeness and QV values
- assembly_evaluation_metrics.csv containing MCL reduction percentages, BUSCO/Merqury/NG50 deltas, and runtime ratios

## How to apply

Assemble each downsampled dataset with both pipelines (HiFiCCL default mode via hificcl.py, and Hifiasm 0.19.5-r592 with -t20 --primary as baseline), then run QUAST v5.2.0 on the resulting contigs against high-quality published references (e.g., CHM13 v2.0, GRCh38) to extract contig size, number of contigs, misassembled contig length (MCL), NG50 and NGA50. Complement QUAST with BUSCO 5.4.7 using the vertebrate conserved single-copy ortholog database and with Merqury for completeness and QV. Record wall time and maximum resident set size for both pipelines so quality gains can be weighed against computational cost. Finally, compute the misassembled-contig-length reduction percentage (e.g., HG002: 331.84 to 274.16 Mb, a 17.38% reduction; NA19240: 245.96 to 179.15 Mb, 27.16%) and write all metrics to assembly_evaluation_metrics.csv. Validation against the article: HiFiCCL-Hifiasm should reduce MCL relative to Hifiasm by an average of 21.19% and up to 38.58% across the 45 ~5x human datasets, with improved BUSCO, Merqury and NG50 values.

## Related tools

- **QUAST** (Contiguity and misassembly evaluation of assembled contigs against high-quality published assemblies; source of contig size, number of contigs, MCL, NG50 and NGA50 metrics)
- **BUSCO** (Gene completeness assessment (v5.4.7) using the vertebrate conserved single-copy ortholog database)
- **Merqury** (k-mer-based completeness and QV estimation complementing reference-based QUAST metrics)
- **HiFiCCL** (Produces the reference-guided chromosome-by-chromosome assemblies being evaluated (default mode via hificcl.py -f <reads> -r CHM13v2.0.fasta -t 20)) — https://github.com/zjjbuqi/HiFiCCL
- **hifiasm** (Baseline assembler (0.19.5-r592, -t20 --primary) whose contigs serve as the comparison point for MCL reduction) — https://github.com/chhylp123/hifiasm
- **seqtk** (Downsamples HiFi read sets to ~5x coverage (v1.3-r106) before assembly and evaluation) — https://github.com/lh3/seqtk

## Examples

```
quast.py -r CHM13v2.0.fasta -t 20 -o quast_out hificcl_hifiasm.ctg.fa hifiasm.ctg.fa
```

## Evaluation signals

- Misassembled contig length (MCL) from QUAST should be lower for HiFiCCL-Hifiasm than Hifiasm alone, with expected average reduction of 21.19% (up to 38.58%) on ~5x human datasets
- Spot-check specific datasets: HG002 MCL drops from 331.84 to 274.16 Mb (17.38%) and NA19240 from 245.96 to 179.15 Mb (27.16%)
- BUSCO scores (vertebrate single-copy orthologs) and Merqury completeness/QV should be equal or improved for HiFiCCL-Hifiasm relative to the Hifiasm baseline
- NG50 values from QUAST should be equal or improved for the HiFiCCL-Hifiasm assemblies
- All metrics are present and parseable in assembly_evaluation_metrics.csv, including wall time and maximum resident set size per pipeline

## Limitations

- At ~5x coverage many genomic regions remain missing, so low completeness values may reflect coverage limits rather than assembler failure
- Misassembly reductions and completeness gains are validated on human datasets (45 samples at ~5x) and only two plant samples; broader plant evaluation has not been performed
- HiFiCCL pipelines carry higher runtime than Hifiasm alone, so quality improvements must be weighed against runtime and memory costs (record wall time and max RSS)

## Evidence

- [methods] Evaluate contigs with QUAST v5.2.0 against high-quality published assemblies to obtain contig size, number of contigs, misassembled contig length, NG50 and NGA50.: [evidence span withheld — anchor status: not_found]
- [methods] BUSCO and Merqury complement QUAST for completeness and quality value assessment.: [evidence span withheld — anchor status: not_found]
- [abstract] Expected benchmark result: HiFiCCL reduces misassembled contig length by an average of 21.19% versus hifiasm alone.: "reduces the length of misassembled contigs relative to hifiasm by an average of 21.19% and up to 38.58%"
- [methods] Low-coverage downsampling precedes assembly so QUAST evaluation is on ~5x data.: [evidence span withheld — anchor status: not_found]
