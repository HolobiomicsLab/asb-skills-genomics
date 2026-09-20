---
name: transcript-quantification-tool-comparison
description: Use when comparing transcript quantification tools or Salmon run modes
  on nanopore DRS reads against a spike-in truth set (e.g. SIRV E2) to decide which
  quantification mode recovers known abundance.
when_to_use_negative:
- The data is short-read RNA-seq, where Salmon's default error model and length correction
  are appropriate rather than ablation targets
- The goal is quantification itself rather than tool/mode benchmarking — use NanoCount
  directly, which the paper shows outperforms Salmon, StringTie2, FLAIR and LIQA for
  DRS
- Results will be extrapolated to a complex human transcriptome without caution, given
  the lower complexity of the spike-in transcriptome
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0102
tools:
- name: Salmon
  role: Transcript quantification tool whose three run modes (default, --noErrorModel,
    --noErrorModel --noLengthCorrection) are ablated and benchmarked in aligner mode
- name: NanoCount
  role: DRS-specific EM-based quantifier that the benchmarked Salmon configuration
    is compared against; also the reference method for DRS transcript abundance estimation
  repo: https://github.com/a-slide/NanoCount
- name: minimap2
  role: Aligns FASTQ reads to the transcriptome with '-ax map-ont -p 0 -N 10' to retain
    secondary mappings needed for EM quantification
  repo: https://github.com/lh3/minimap2
- name: SciPy
  role: Computes Spearman's rank correlation (scipy.stats.spearmanr) between measured
    quant.sf abundance and known SIRV molar concentrations
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkab1129
    title: Accurate expression quantification from nanopore direct RNA sequencing
      with NanoCount
  source_content_hash: sha256:edb9069d6719ac7f1b8ded89fbdb37e48b20b788d742edea600880616f352442
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/a-slide/NanoCount
  tools:
  - NanoCount
  - Salmon
  - SciPy
  - minimap2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5882352941176471
n_claims: 17
---

# transcript-quantification-tool-comparison

## Summary

A benchmarking skill for evaluating transcript quantification tools and Salmon run-mode configurations on Oxford Nanopore direct RNA sequencing (DRS) data using SIRV spike-in transcripts with known molar abundances, scored by Spearman rank correlation between known and measured abundance.

## When to use

Use when comparing transcript quantification tools or Salmon run modes on nanopore DRS reads against a spike-in truth set (e.g. SIRV E2) to decide which quantification mode best recovers known abundance.

## When NOT to use

- The data is short-read RNA-seq, where Salmon's default error model and length correction are appropriate rather than ablation targets
- The goal is quantification itself rather than tool/mode benchmarking — use NanoCount directly, which the paper shows outperforms Salmon, StringTie2, FLAIR and LIQA for DRS
- Results will be extrapolated to a complex human transcriptome without caution, given the lower complexity of the spike-in transcriptome

## Inputs

- SIRV E2 direct-RNA sequencing FASTQ reads (SRR6058583, ENA PRJEB39347)
- SIRV transcriptome reference FASTA
- Known SIRV mixture molar concentration/abundance table
- minimap2 alignment BAM against the transcriptome (with secondary mappings retained)

## Outputs

- quant.sf from each of three Salmon runs (default, --noErrorModel, --noErrorModel --noLengthCorrection)
- Three-row TSV table with columns for Salmon mode and Spearman's rho

## How to apply

Download the SIRV E2 direct-RNA FASTQ reads (SRR6058583, ENA study PRJEB39347), the SIRV transcriptome reference FASTA, and the known SIRV mixture molar abundance table. Align reads to the transcriptome with minimap2 using '-ax map-ont -p 0 -N 10' so at least 10 secondary mappings are retained, then convert SAM to BAM with samtools — secondary mappings are essential because EM-based quantifiers like Salmon and NanoCount use an expectation-maximization approach to handle multi-mapping reads. Run salmon quant in aligner mode three times on the BAM: (a) default, (b) --noErrorModel, and (c) --noErrorModel --noLengthCorrection, keeping each quant.sf. Parse each quant.sf, join measured abundance to known SIRV molar concentrations by transcript identifier, and compute Spearman's rho with scipy.stats.spearmanr for each mode. Select the mode with the highest correlation — in the paper's data Salmon's correlations were highest when the error model was disabled, so --noErrorModel was selected for benchmarking against NanoCount.

## Related tools

- **Salmon** (Transcript quantification tool whose three run modes (default, --noErrorModel, --noErrorModel --noLengthCorrection) are ablated and benchmarked in aligner mode)
- **NanoCount** (DRS-specific EM-based quantifier that the benchmarked Salmon configuration is compared against; also the reference method for DRS transcript abundance estimation) — https://github.com/a-slide/NanoCount
- **minimap2** (Aligns FASTQ reads to the transcriptome with '-ax map-ont -p 0 -N 10' to retain secondary mappings needed for EM quantification) — https://github.com/lh3/minimap2
- **SciPy** (Computes Spearman's rank correlation (scipy.stats.spearmanr) between measured quant.sf abundance and known SIRV molar concentrations)

## Examples

```
minimap2 -ax map-ont --sam-hit-only SIRV_transcriptome.fasta SRR6058583.fastq | samtools view -bh > aligned_reads.bam
```

## Evaluation signals

- The output TSV contains exactly three rows, one per Salmon mode, with numeric Spearman rho values in [-1, 1]
- Correlations are highest for the modes with the error model disabled, consistent with the finding that default Salmon is suboptimal for DRS reads
- Each salmon quant run produced a valid quant.sf in aligner mode on the minimap2 BAM
- Secondary mappings were retained during alignment (minimap2 '-N 10'), without which multi-mapping-aware EM quantification would be degraded
- Measured abundances join to known SIRV molar concentrations on transcript identifiers without orphan or missing transcripts

## Limitations

- Default Salmon is suboptimal for direct-RNA sequencing reads; the error model must be disabled to match the paper's benchmark configuration
- Caution is needed extrapolating spike-in validation results to human data due to the lower complexity of the spike-in transcriptome compared to the human one
- A high proportion of DRS reads is non-full-length and incomplete reads largely consigned to 3'UTRs cannot specify the expressed isoform, limiting quantification accuracy

## Evidence

- [results] We tested the performance of NanoCount against other transcript quantification programs, including Salmon, StringTie2, FLAIR and LIQA: "We tested the performance of NanoCount against other transcript quantification programs, including Salmon, StringTie2, FLAIR and LIQA"
- [results] SIRV DRS data: "using SIRV RNA spike-in DRS data (Materials and Methods)."
- [supplementary] ENA PRJEB39347: "FAST5 and FASTQ files are publicly available to download from ENA: PRJEB39347."
- [readme] EM-based quantification rationale: "using an expectation-maximization approach like RSEM, Kallisto, salmon, etc to handle the uncertainty of multi-mapping reads"
- [readme] minimap2 secondary mappings: [evidence span withheld — anchor status: external_doc]
