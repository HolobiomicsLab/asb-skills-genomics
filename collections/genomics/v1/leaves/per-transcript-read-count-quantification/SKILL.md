---
name: per-transcript-read-count-quantification
description: Use when comparing per-transcript read or nucleotide abundance between
  two sequencing conditions (e.g., RISER-deplete vs control channels) to quantify
  percent change per transcript, as for globin mRNA depletion in whole blood.
when_to_use_negative:
- The comparison is between independent sequencing runs without a shared channel/split-flow-cell
  design — random read splitting alone cannot separate RISER effects from run-to-run
  variation without the described randomization control.
- The goal is whole-sample gene-level abundance (e.g., a count matrix for differential
  expression) rather than measuring depletion/enrichment percent change of targeted
  transcripts.
- Reads are not high-confidence alignments (e.g., secondary/supplementary/unmapped
  mappings not removed with samtools -F 2324), so per-transcript counts would be unreliable.
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3512
tools:
- name: RISER
  role: Produces decisions.csv classifying reads as accepted/rejected during live
    globin-depletion; used to split accepted vs rejected reads when computing per-transcript
    percent changes
  repo: github.com/comprna/riser
- name: Guppy (v4.0.14)
  role: Basecalls FAST5 files (–flowcell FLO-MIN106 –kit SQK-RNA002) prior to mapping
    and counting
- name: minimap2 (v2.17)
  role: Maps basecalled reads to the GENCODE reference transcriptome with -ax map-ont
    --secondary=no to assign reads to transcripts for counting
- name: samtools (v1.10)
  role: Filters alignments with option -F 2324 to retain only high-confidence mapped
    reads before per-transcript counting
- name: BoostNano
  role: Trims sequencing adapter and poly(A) tail from raw nanopore signals before
    basecalling
  repo: https://github.com/haotianteng/BoostNano
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-48673-8
    title: Biochemical-free enrichment or depletion of RNA classes in real-time during
      direct RNA sequencing with RISER
  source_content_hash: sha256:80e03eabfaa944314be58d0ac5c52fa3083066ae6b3781529983d0f515198072
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BoostNano
  - Guppy (v4.0.14)
  - RISER
  - minimap2 (v2.17)
  - samtools (v1.10)
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# per-transcript read count quantification

## Summary

Quantify per-transcript read and nucleotide counts from nanopore direct RNA sequencing BAM alignments under two conditions (e.g., RISER globin-deplete vs control split-flow-cell) and compute per-transcript percent changes to measure depletion efficiency.

## When to use

Use when comparing per-transcript read or nucleotide abundance between two sequencing conditions (e.g., RISER-deplete vs control channels) to quantify percent change per transcript, as for globin mRNA depletion in whole blood.

## When NOT to use

- The comparison is between independent sequencing runs without a shared channel/split-flow-cell design — random read splitting alone cannot separate RISER effects from run-to-run variation without the described randomization control.
- The goal is whole-sample gene-level abundance (e.g., a count matrix for differential expression) rather than measuring depletion/enrichment percent change of targeted transcripts.
- Reads are not high-confidence alignments (e.g., secondary/supplementary/unmapped mappings not removed with samtools -F 2324), so per-transcript counts would be unreliable.

## Inputs

- BAM file of DRS reads basecalled with Guppy (v4.0.14), mapped to GENCODE release 43 (GRCh38.p13) with minimap2 (v2.17) and filtered with samtools -F 2324
- RISER decisions.csv file(s) from the globin-deplete and control split-flow-cell conditions
- Transcript-to-gene annotations (GENCODE release 43) with biotype information to identify globin genes (HBA1, HBA2, HBB), non-globin mRNAs, and lncRNAs

## Outputs

- Per-transcript read counts and nucleotide (nt) counts per condition (deplete vs control)
- Per-transcript percent change in read and nt counts (RISER vs control), written to percent_change_results.csv
- Mean percent change for globin transcripts and for non-globin transcripts, plus Wilcoxon signed rank test result against the randomized split control

## How to apply

Basecall FAST5 files with Guppy (v4.0.14), map reads to the GENCODE reference transcriptome (release 43, GRCh38.p13) with minimap2 (v2.17), and filter with samtools (v1.10) using -F 2324 to remove unmapped/low-confidence alignments. Compute per-transcript read counts and nucleotide counts in each condition, splitting reads by channel assignment (even channels = RISER deplete, remaining = control) and using the RISER decisions.csv to separate accepted from rejected reads. Restrict analysis to transcripts occupying at least 95% of control-condition reads with at least 30 read counts in both conditions, excluding lncRNAs whose exons overlap protein-coding exons. Calculate per-transcript percent change (RISER vs control), then average separately for globin genes (HBA1, HBA2, HBB) and non-globin transcripts; for the whole-blood globin run this reproduces a 93% average globin read decrease (n=4) and a 16% average non-globin increase (n=100). Validate by recomputing percent changes between two randomly split halves of an independent non-RISER run and testing with a paired Wilcoxon signed rank test.

## Related tools

- **RISER** (Produces decisions.csv classifying reads as accepted/rejected during live globin-depletion; used to split accepted vs rejected reads when computing per-transcript percent changes) — github.com/comprna/riser
- **Guppy (v4.0.14)** (Basecalls FAST5 files (–flowcell FLO-MIN106 –kit SQK-RNA002) prior to mapping and counting)
- **minimap2 (v2.17)** (Maps basecalled reads to the GENCODE reference transcriptome with -ax map-ont --secondary=no to assign reads to transcripts for counting)
- **samtools (v1.10)** (Filters alignments with option -F 2324 to retain only high-confidence mapped reads before per-transcript counting)
- **BoostNano** (Trims sequencing adapter and poly(A) tail from raw nanopore signals before basecalling) — https://github.com/haotianteng/BoostNano

## Examples

```
minimap2 -ax map-ont --secondary=no GENCODE_v43.fa reads.fastq | samtools view -b -F 2324 | samtools sort -o filtered.bam; python3 count_percent_change.py --bam filtered.bam --decisions decisions.csv --gencode gencode.v43.annotation.gtf --out percent_change_results.csv
```

## Evaluation signals

- Globin transcript (HBA1, HBA2, HBB) mean read-count decrease should exceed the reported >90% depletion (task card: 93% average, n=4).
- Non-globin mRNA transcripts should show ~16% average read increase (task card: 16% average, n=100) after depletion.
- Only transcripts taking ≥95% of control-condition reads and having ≥30 read counts in both conditions should enter the percent-change calculation, with overlapping lncRNAs excluded.
- Percent changes between two randomly split halves of an independent non-RISER run should not show a systematic direction (paired Wilcoxon signed rank test), confirming changes are RISER-specific.
- All computed values must be present in percent_change_results.csv.

## Limitations

- Bioinformatically discarding globin reads post-sequencing is insufficient to counteract the problem of limited coverage of non-globin RNAs; only real-time depletion recovers sequencing capacity.
- Lag between the read-until API receiving a reject command and requesting voltage reversal, and the efficiency of the voltage reversal itself, vary across platforms and are beyond RISER's control, affecting depletion efficiency measured per transcript.
- RNA secondary structures forming on the trans side of the pore could impede or inhibit reverse translocation by voltage reversal, reducing effective depletion of some transcripts.

## Evidence

- [methods] basecalled, mapped and filtered using the GENCODE reference transcriptome (release 43, assembly GRCh38.p13): "basecalled, mapped and filtered using the GENCODE reference transcriptome (release 43, assembly GRCh38.p13)"
- [methods] unmapped reads were removed using samtools34 (v1.10) with option -F 2324: "unmapped reads were removed using samtools34 (v1.10) with option -F 2324"
- [abstract] achieving a decrease in globin reads by more than 90% as well as an increase in non-globin reads by 16% on average: "achieving a decrease in globin reads by more than 90% as well as an increase in non-globin reads by 16% on average"
- [methods] Fast5 files were basecalled with Guppy (v4.0.14) using options –flowcell FLO-MIN106 –kit SQK-RNA002: "Fast5 files were basecalled with Guppy (v4.0.14) using options –flowcell FLO-MIN106 –kit SQK-RNA002"
- [discussion] bioinformatically discarding globin reads post-sequencing is insufficient to counteract the problem of limited coverage of non-globin RNAs: "bioinformatically discarding globin reads post-sequencing is insufficient to counteract the problem of limited coverage of non-globin RNAs"
