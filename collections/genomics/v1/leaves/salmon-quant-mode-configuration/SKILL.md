---
name: salmon-quant-mode-configuration
description: Use when quantifying long-read DRS read abundance with Salmon and needing
  to decide which Salmon run mode recovers known transcript concentrations on spike-in
  benchmark data such as SIRV E2.
when_to_use_negative:
- Input reads are short-read RNA-seq (e.g. Illumina), where Salmon's default error
  model and length correction are appropriate
- The goal is isoform quantification with a DRS-specific filtering and EM approach,
  in which case NanoCount should be used directly rather than tuning Salmon
- No spike-in or ground-truth abundance table is available, making the mode-selection
  correlation benchmark impossible to score
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Salmon
  role: Transcript quantification run in aligner mode three times (default, --noErrorModel,
    --noErrorModel --noLengthCorrection) on the aligned BAM to compare run modes
- name: minimap2
  role: Aligns SIRV E2 DRS FASTQ reads to the SIRV transcriptome with '-ax map-ont
    -p 0 -N 10' to retain at least 10 secondary mappings for downstream EM quantification
  repo: https://github.com/lh3/minimap2
- name: SciPy
  role: scipy.stats.spearmanr computes Spearman's rank correlation between measured
    and known SIRV abundances for each Salmon mode
- name: samtools
  role: Converts minimap2 SAM output to BAM for Salmon aligner-mode input
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkab1129
    title: Accurate expression quantification from nanopore direct RNA sequencing
      with NanoCount
  source_content_hash: sha256:195eb84fd6a191257f9cbe958b2a52878623f13a00262a1414f15cec979714d6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - NanoCount
  - Salmon
  - SciPy
  - minimap2
  - samtools
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# salmon-quant-mode-configuration

## Summary

A procedure for benchmarking Salmon's three documented run modes (default, --noErrorModel, and --noErrorModel --noLengthCorrection) on nanopore direct RNA sequencing (DRS) reads, using Spearman correlation against known SIRV spike-in abundances to select the best quantification mode.

## When to use

Use when quantifying long-read DRS read abundance with Salmon and needing to decide which Salmon run mode best recovers known transcript concentrations on spike-in benchmark data such as SIRV E2.

## When NOT to use

- Input reads are short-read RNA-seq (e.g. Illumina), where Salmon's default error model and length correction are appropriate
- The goal is isoform quantification with a DRS-specific filtering and EM approach, in which case NanoCount should be used directly rather than tuning Salmon
- No spike-in or ground-truth abundance table is available, making the mode-selection correlation benchmark impossible to score

## Inputs

- SIRV E2 direct-RNA sequencing FASTQ reads (accession SRR6058583, ENA study PRJEB39347)
- SIRV transcriptome reference FASTA
- Known SIRV mixture molar concentration (abundance) table
- BAM file of minimap2 alignments to the SIRV transcriptome (produced with -N 10 secondary mappings)

## Outputs

- Three quant.sf files, one per Salmon mode (default, --noErrorModel, --noErrorModel --noLengthCorrection)
- A three-row TSV table with columns for the Salmon mode and the corresponding Spearman's rho, ordered default → --noErrorModel → --noErrorModel --noLengthCorrection

## How to apply

Download the SIRV E2 direct-RNA FASTQ reads (SRR6058583, ENA study PRJEB39347), the SIRV transcriptome FASTA, and the known SIRV mixture molar concentration table. Align reads to the SIRV transcriptome with minimap2 using '-ax map-ont -p 0 -N 10' to retain at least 10 secondary mappings (secondary alignments are required because multi-mapping reads must be resolved by Salmon's EM algorithm), and convert to BAM with samtools. Run salmon quant in aligner mode three times on the same BAM: default settings, with --noErrorModel, and with --noErrorModel --noLengthCorrection, keeping each quant.sf. Join measured transcript abundance to known SIRV molar concentrations by transcript identifier and compute Spearman's rank correlation (scipy.stats.spearmanr) per mode. On this paper's SIRV DRS data, correlations were highest when the error model was disabled, so --noErrorModel was selected for benchmarking — default Salmon is suboptimal for DRS reads because its short-read error model mismodels nanopore native RNA reads.

## Related tools

- **Salmon** (Transcript quantification run in aligner mode three times (default, --noErrorModel, --noErrorModel --noLengthCorrection) on the aligned BAM to compare run modes)
- **minimap2** (Aligns SIRV E2 DRS FASTQ reads to the SIRV transcriptome with '-ax map-ont -p 0 -N 10' to retain at least 10 secondary mappings for downstream EM quantification) — https://github.com/lh3/minimap2
- **SciPy** (scipy.stats.spearmanr computes Spearman's rank correlation between measured and known SIRV abundances for each Salmon mode)
- **samtools** (Converts minimap2 SAM output to BAM for Salmon aligner-mode input)

## Examples

```
minimap2 -ax map-ont -p 0 -N 10 SIRV_transcriptome.fa SRR6058583.fastq | samtools view -bh > aligned_reads.bam && salmon quant -t SIRV_transcriptome.fa -l A -a aligned_reads.bam --noErrorModel -o salmon_nem_out
```

## Evaluation signals

- The output TSV contains exactly three rows, one per Salmon mode, with numeric Spearman's rho values in [-1, 1]
- Correlations are highest for the modes with the error model disabled (--noErrorModel and --noErrorModel --noLengthCorrection), consistent with the paper's observation
- The minimap2 alignment retains at least 10 secondary mappings per read (-N 10) so Salmon's EM can resolve multi-mapping reads
- Measured quant.sf transcript identifiers join uniquely to the SIRV known molar concentration table before correlation
- The same BAM input is reused across all three Salmon runs so mode differences are the only variable

## Limitations

- Default Salmon is suboptimal for DRS reads; conclusions are specific to nanopore direct RNA sequencing data and may not transfer to short-read RNA-seq
- Caution is needed extrapolating spike-in (SIRV/sequin) validation results due to the lower complexity of the synthetic transcriptome compared to the human one
- Multi-mapping uncertainty in long reads means all quantification tools (Salmon, NanoCount, etc.) rely on EM to resolve ambiguous assignments

## Evidence

- [results] Salmon's correlations with known abundance on the SIRV DRS data were highest when the error model was disabled, so the –noErrorModel mode was selected for benchmarking.: [evidence span withheld — anchor status: not_found]
- [readme] NanoCount README on multi-mapping EM: "NanoCount estimates transcripts abundance from Oxford Nanopore *direct-RNA sequencing* datasets, using an expectation-maximization approach like RSEM, Kallisto, salmon, etc to handle the uncertainty"
- [readme] jts/nanopore-rna-analysis README on minimap2 -p0 secondary mappings: [evidence span withheld — anchor status: external_doc]
- [results] NanoCount filtering of secondary alignments: "and secondary alignments with an AS <95% that of the best alignment (highest AS), as these likely represent misalignments or reads from unannotated isoforms"
- [results] NanoCount outperforms comparison tools including Salmon: "We tested the performance of NanoCount against other transcript quantification programs, including Salmon, StringTie2, FLAIR and LIQA"
