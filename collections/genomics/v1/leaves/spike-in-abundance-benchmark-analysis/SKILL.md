---
name: spike-in-abundance-benchmark-analysis
description: Use when evaluating or selecting quantification tool settings on direct-RNA
  or long-read data with a synthetic spike-in control (SIRV/sequin) reference transcriptome
  and a known mixture abundance table.
when_to_use_negative:
- Input data is short-read RNA-seq without spike-ins — Salmon's error model and length
  correction are appropriate there and the DRS-specific ablation rationale does not
  apply
- The goal is de novo isoform discovery or differential expression between conditions
  rather than absolute quantification accuracy against known concentrations
- No known abundance table exists, so correlation between known and measured abundance
  cannot be computed
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: minimap2
  role: Aligns DRS reads to the SIRV transcriptome with '-ax map-ont -p 0 -N 10' to
    retain secondary mappings needed for EM quantification
  repo: https://github.com/lh3/minimap2
- name: Salmon
  role: Run in aligner mode on the BAM three times (default, --noErrorModel, --noErrorModel
    --noLengthCorrection) to compare quantification modes against known abundance
- name: SciPy
  role: scipy.stats.spearmanr computes Spearman's rank correlation between known SIRV
    concentrations and measured Salmon abundance per mode
- name: NanoCount
  role: DRS-specific EM quantifier benchmarked as an alternative; performs DRS-specific
    filtering (3' end ≤50nt, secondary alignments with AS ≥95% of best) before quantification
  repo: https://github.com/a-slide/NanoCount
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkab1129
    title: Accurate expression quantification from nanopore direct RNA sequencing
      with NanoCount
  source_content_hash: sha256:5296eaf7d81462dc81f30f3d57762539f280c24ff5d5decc5528ddc6675a9b95
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - NanoCount
  - Salmon
  - SciPy
  - minimap2
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# spike-in-abundance-benchmark-analysis

## Summary

Benchmarks transcript quantification modes against known synthetic spike-in molar concentrations using SIRV direct-RNA sequencing (DRS) data, by aligning reads with minimap2 and computing Spearman rank correlations between known and measured abundance for each quantification configuration (e.g. Salmon default vs --noErrorModel vs --noErrorModel --noLengthCorrection).

## When to use

Use when evaluating or selecting quantification tool settings on direct-RNA or long-read data with a synthetic spike-in control (SIRV/sequin) reference transcriptome and a known mixture abundance table.

## When NOT to use

- Input data is short-read RNA-seq without spike-ins — Salmon's error model and length correction are appropriate there and the DRS-specific ablation rationale does not apply
- The goal is de novo isoform discovery or differential expression between conditions rather than absolute quantification accuracy against known concentrations
- No known abundance table exists, so correlation between known and measured abundance cannot be computed

## Inputs

- SIRV E2 direct-RNA sequencing FASTQ reads (ENA accession SRR6058583, study PRJEB39347)
- SIRV transcriptome reference FASTA
- Known SIRV mixture molar concentration (abundance) table

## Outputs

- Three quant.sf files (one per Salmon run mode)
- Three-row TSV table with Salmon mode and corresponding Spearman's rho

## How to apply

Download the SIRV E2 direct-RNA FASTQ reads (SRR6058583, ENA study PRJEB39347) together with the SIRV transcriptome FASTA and the known SIRV mixture abundance table. Align reads to the SIRV transcriptome with minimap2 using the map-ont preset with '-p 0' and '-N 10' to retain at least 10 secondary mappings, which is essential because EM-based quantifiers must handle multi-mapping reads, and convert to BAM with samtools. Run Salmon quant in aligner mode three times on the same BAM: default settings, with --noErrorModel, and with --noErrorModel --noLengthCorrection, keeping the quant.sf from each run. Parse each quant.sf, join measured transcript abundance to the known SIRV molar concentrations by transcript identifier, and compute Spearman's rank correlation with scipy.stats.spearmanr. Report a three-row table of mode versus rho; the paper's finding was that correlations were highest when the error model was disabled, so --noErrorModel was selected for benchmarking against NanoCount, StringTie2, FLAIR and LIQA.

## Related tools

- **minimap2** (Aligns DRS reads to the SIRV transcriptome with '-ax map-ont -p 0 -N 10' to retain secondary mappings needed for EM quantification) — https://github.com/lh3/minimap2
- **Salmon** (Run in aligner mode on the BAM three times (default, --noErrorModel, --noErrorModel --noLengthCorrection) to compare quantification modes against known abundance)
- **SciPy** (scipy.stats.spearmanr computes Spearman's rank correlation between known SIRV concentrations and measured Salmon abundance per mode)
- **NanoCount** (DRS-specific EM quantifier benchmarked as an alternative; performs DRS-specific filtering (3' end ≤50nt, secondary alignments with AS ≥95% of best) before quantification) — https://github.com/a-slide/NanoCount

## Examples

```
minimap2 -ax map-ont -p 0 -N 10 sirv_transcriptome.fasta SRR6058583.fastq | samtools view -bh > sirv.bam && salmon quant -t sirv_transcriptome.fasta -a sirv.bam -p 8 --noErrorModel -o salmon_nem
```

## Evaluation signals

- The results table contains exactly three rows, one per Salmon mode (default, --noErrorModel, --noErrorModel --noLengthCorrection), each with a numeric Spearman's rho in [-1, 1]
- Correlations are highest for the modes with the error model disabled, consistent with the paper's observation that default Salmon is suboptimal for DRS reads
- minimap2 was run with -N 10 (and -p 0) so secondary alignments are retained; verify the BAM contains secondary records before quantification
- Each measured transcript identifier in quant.sf successfully joins to an entry in the known SIRV molar concentration table (no unmatched transcripts dominating the correlation)
- Same BAM input is used for all three Salmon runs so the only varying factor is the Salmon mode

## Limitations

- Default Salmon settings are suboptimal for nanopore DRS reads, so results from the default mode should not be treated as Salmon's best achievable performance on this data
- Multi-mapping reads on the SIRV transcriptome create assignment uncertainty that EM-based quantifiers handle differently, affecting comparability
- Spike-in transcriptomes are less complex than the human transcriptome, so caution is needed extrapolating benchmark results

## Evidence

- [results] SIRV RNA spike-ins used to assess quantification accuracy on DRS data: "using SIRV RNA spike-in DRS data (Materials and Methods)."
- [results] NanoCount benchmarked against Salmon and other quantification tools: "We tested the performance of NanoCount against other transcript quantification programs, including Salmon, StringTie2, FLAIR and LIQA"
- [supplementary] SIRV/sequin DRS data publicly available from ENA under PRJEB39347: "FAST5 and FASTQ files are publicly available to download from ENA: PRJEB39347."
- [readme] NanoCount uses EM like Salmon to handle multi-mapping reads: "using an expectation-maximization approach like RSEM, Kallisto, salmon, etc to handle the uncertainty of multi-mapping reads"
- [readme] minimap2 command for producing transcriptome BAM from long-read data: [evidence span withheld — anchor status: external_doc]
