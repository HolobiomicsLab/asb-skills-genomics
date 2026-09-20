---
name: coefficient-of-variation-precision-computation
description: Use when benchmarking transcript quantification tools on spike-in DRS
  data with an equal-concentration mix (e.g. SIRV E0 from SRR6058583), where lower
  variability in measured isoform abundances indicates better quantification precision.
when_to_use_negative:
- Input is mix E2 or any spike-in mix with varied concentrations — variability there
  reflects true abundance differences, so use Spearman's rank correlation rho between
  known and measured abundance instead
- Data have no spike-in controls with known equal concentrations, so a CV across transcripts
  cannot be interpreted as quantification precision
- Goal is differential expression of human/SH-SY5Y genes and isoforms (DESeq2, DEXSeq/IsoformSwitchAnalyzeR)
  rather than quantifier benchmarking on synthetic controls
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: NanoCount
  role: DRS-specific quantifier under test whose E0 CV is compared against Salmon,
    StringTie2, FLAIR and LIQA; applies 3' end <=50nt and secondary AS >=95% filtering
    before EM quantification
  repo: https://github.com/a-slide/NanoCount
- name: minimap2
  role: Aligns SIRV E0/E2 direct-RNA reads to the transcriptome with -ax map-ont and
    -N 10 to retain >=10 secondary mappings before quantification
- name: Salmon
  role: Competing quantifier run with --noErrorModel on the same alignments to compare
    E0 CV
- name: StringTie2
  role: Competing quantifier run in default mode on the same alignments to compare
    E0 CV
- name: FLAIR
  role: Competing quantifier run in default mode on the same alignments to compare
    E0 CV
- name: LIQA
  role: Competing quantifier run in default mode on the same alignments to compare
    E0 CV
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkab1129
    title: Accurate expression quantification from nanopore direct RNA sequencing
      with NanoCount
  source_content_hash: sha256:873b137de08968f071afefad1743356a61f74c058abd7b67a6681bfac87b4a20
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/a-slide/NanoCount
  tools:
  - FLAIR
  - LIQA
  - NanoCount
  - Salmon
  - StringTie2
  - minimap2
  - samtools
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# coefficient-of-variation-precision-computation

## Summary

Computes the coefficient of variance (CV) of measured isoform abundances across transcripts of a constant-concentration spike-in mix (SIRV mix E0) to score the precision of direct RNA sequencing (DRS) transcript quantification tools such as NanoCount versus Salmon, StringTie2, FLAIR and LIQA.

## When to use

Use when benchmarking transcript quantification tools on spike-in DRS data with an equal-concentration mix (e.g. SIRV E0 from SRR6058583), where lower variability in measured isoform abundances indicates better quantification precision.

## When NOT to use

- Input is mix E2 or any spike-in mix with varied concentrations — variability there reflects true abundance differences, so use Spearman's rank correlation rho between known and measured abundance instead
- Data have no spike-in controls with known equal concentrations, so a CV across transcripts cannot be interpreted as quantification precision
- Goal is differential expression of human/SH-SY5Y genes and isoforms (DESeq2, DEXSeq/IsoformSwitchAnalyzeR) rather than quantifier benchmarking on synthetic controls

## Inputs

- SIRV E0/E2 direct-RNA sequencing reads (ENA accession SRR6058583, FASTQ/FAST5 from project PRJEB39347)
- SIRV transcriptome reference FASTA
- SIRV annotation variants (complete C, insufficient I, over-annotation O)
- Transcript abundance tables from NanoCount, Salmon (--noErrorModel), StringTie2, FLAIR and LIQA on the same alignments

## Outputs

- Per-tool/per-annotation-variant coefficient of variance across transcripts for mix E0
- Benchmarking TSV table comparable to Supplementary Table S1 (CV for E0 plus Spearman rho for E2)

## How to apply

Align the SIRV E0/E2 direct-RNA reads (SRR6058583) to the SIRV transcriptome reference with minimap2 using -ax map-ont and -N 10 to retain at least 10 secondary mappings, converting output to BAM with samtools. Quantify transcript abundance with each competing quantifier on the same alignments: NanoCount v1.0.0 with default DRS-specific filtering (3' ends within 50nt of the transcript 3' end and secondary alignments with AS >=95% of the best) and expectation-maximization, Salmon with --noErrorModel, and StringTie2, FLAIR and LIQA in default modes. For each quantifier and each SIRV annotation variant (C, I, O), compute the coefficient of variance across transcripts for mix E0, where all known concentrations are equal, so the CV directly reflects measurement variability independent of true abundance differences. Assemble the per-tool CV values into a benchmarking table comparable to Supplementary Table S1; pair this metric with Spearman's rank correlation rho between known and measured abundance for mix E2, which has varied concentrations.

## Related tools

- **NanoCount** (DRS-specific quantifier under test whose E0 CV is compared against Salmon, StringTie2, FLAIR and LIQA; applies 3' end <=50nt and secondary AS >=95% filtering before EM quantification) — https://github.com/a-slide/NanoCount
- **minimap2** (Aligns SIRV E0/E2 direct-RNA reads to the transcriptome with -ax map-ont and -N 10 to retain >=10 secondary mappings before quantification)
- **Salmon** (Competing quantifier run with --noErrorModel on the same alignments to compare E0 CV)
- **StringTie2** (Competing quantifier run in default mode on the same alignments to compare E0 CV)
- **FLAIR** (Competing quantifier run in default mode on the same alignments to compare E0 CV)
- **LIQA** (Competing quantifier run in default mode on the same alignments to compare E0 CV)

## Examples

```
minimap2 -ax map-ont -N 10 transcriptome.fasta SRR6058583.fastq | samtools view -bh > aligned_reads.bam
```

## Evaluation signals

- Per-tool CV and Spearman rho values in the output table match the corresponding entries of Supplementary Table S1 (output_matches_reference / expert_review)
- NanoCount shows the least variability (lowest CV) in measured isoform abundances for mix E0 among NanoCount, Salmon, StringTie2, FLAIR and LIQA
- NanoCount has the highest Spearman rho for mix E2 among the tested quantifiers
- Ranking holds across all three SIRV annotation variants (complete, insufficient, over-annotation)
- Alignments retain >=10 secondary mappings (-N 10) so multi-mapping reads are available to every quantifier

## Limitations

- Caution extrapolating spike-in benchmark results to human transcriptomes due to the lower complexity of the spike-in transcriptome compared to the human one
- Many incomplete DRS reads are largely consigned to 3'UTRs and cannot specify the expressed isoform, inflating multi-mapping uncertainty that the EM step must resolve
- Moderate sequencing depth limits detection and quantification of lowly expressed genes and isoforms

## Evidence

- [other] NanoCount showed the best quantification performance on both SIRV mixes: it consistently displayed the least variability in measured isoform abundances for mix E0 (equal concentrations): [evidence span withheld — anchor status: not_found]
- [results] We tested the performance of NanoCount against other transcript quantification programs, including Salmon, StringTie2, FLAIR and LIQA: "We tested the performance of NanoCount against other transcript quantification programs, including Salmon, StringTie2, FLAIR and LIQA"
- [results] using SIRV RNA spike-in DRS data (Materials and Methods).: "using SIRV RNA spike-in DRS data (Materials and Methods)."
- [readme] NanoCount estimates transcripts abundance from Oxford Nanopore direct-RNA sequencing datasets, using an expectation-maximization approach like RSEM, Kallisto, salmon, etc to handle the uncertainty of multi-mapping reads: "using an expectation-maximization approach like RSEM, Kallisto, salmon, etc to handle the uncertainty of multi-mapping reads"
- [results] NanoCount performs DRS specific filtering steps to improve transcript assignment prior to quantifying DRS reads using an expectation-maximisation algorithm: "NanoCount performs DRS specific filtering steps to improve transcript assignment prior to quantifying DRS reads using an expectation-maximisation algorithm"
