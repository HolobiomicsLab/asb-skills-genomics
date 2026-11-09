---
name: replicate-count-summarization
description: Use when full-length PAIso-seq CCS reads have been mapped and gene-assigned
  per library, and you need to report mappable transcript counts and gene coverage
  per replicate for QC or comparison.
when_to_use_negative:
- Input is raw subread BAM or untrimmed CCS BAM that has not been demultiplexed, adapter-trimmed,
  and gene-assigned by the run.pl pipeline
- The goal is poly(A) tail length or non-A residue analysis rather than transcript
  detection counts per replicate
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3512
tools:
- name: ccs
  role: Generates circular consensus reads from PacBio subreads that are subsequently
    mapped and counted per replicate
  repo: https://github.com/PacificBiosciences/IsoSeq3/blob/master/README_v3.2.md
- name: minimap2
  role: Aligns clean CCS reads to the mouse reference genome (mm10/GRCm38) whose unique
    gene-assigned alignments are tallied per replicate
  repo: https://github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-13228-9
    title: Poly(A) inclusive RNA isoform sequencing (PAIso−seq) reveals wide-spread
      non-adenosine residues within RNA poly(A) tails
  source_content_hash: sha256:3ad39659bfcb157c60d4cbeac339697388145923bfac509e3d8497fbc51284d2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ccs
  - minimap2
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# Replicate count summarization

## Summary

Tabulates per-replicate detected full-length transcript counts and total genes covered after CCS generation, trimming, concatemer splitting, and gene assignment in PAIso-seq GV oocyte libraries, yielding the summary detection table.

## When to use

Use when full-length PAIso-seq CCS reads have been mapped and gene-assigned per library, and you need to report mappable transcript counts and gene coverage per replicate for QC or comparison.

## When NOT to use

- Input is raw subread BAM or untrimmed CCS BAM that has not been demultiplexed, adapter-trimmed, and gene-assigned by the run.pl pipeline
- The goal is poly(A) tail length or non-A residue analysis rather than transcript detection counts per replicate

## Inputs

- Gene-assigned full-length CCS reads per replicate (output of run.pl with --species mm10 and Mus_musculus.mmi minimap2 index)
- ccs.pass.txt CCS pass information from GetCCSpass.pl
- Trimmed, demultiplexed CCS FASTA files per sample (e.g., sample.out.fasta)

## Outputs

- Summary table of per-replicate detected transcript counts (e.g., 79,994 transcripts rep 1; 227,902 transcripts rep 2; 191,023 transcripts for 15 single cells combined)
- Total number of genes covered (e.g., 11,538 genes)

## How to apply

After the run.pl pipeline assigns clean CCS reads to Ensembl GRCm38 exons, collect the count of unique full-length CCS transcripts mapped to the mouse genome (GRCm38/mm10) per replicate, discarding CCSs assigned to multiple genes and ignoring alignments with supplementary alignment tags. Sum counts across single-cell barcodes where applicable (e.g., 15 single GV oocytes combined). Tabulate per-replicate transcript detections alongside the total number of distinct genes covered into the summary table. Correct application is judged by reproducing the published detection counts, which serve as a validation invariant.

## Related tools

- **ccs** (Generates circular consensus reads from PacBio subreads that are subsequently mapped and counted per replicate) — https://github.com/PacificBiosciences/IsoSeq3/blob/master/README_v3.2.md
- **minimap2** (Aligns clean CCS reads to the mouse reference genome (mm10/GRCm38) whose unique gene-assigned alignments are tallied per replicate) — https://github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis

## Examples

```
perl run.pl --ccs sample.out.fasta --sample sampleName --species mm10 --minimap2_index Mus_musculus.mmi --minimap2_thread 10 --pass ccs.pass.txt &>run.log
```

## Evaluation signals

- Replicate 1 yields exactly 79,994 mappable transcripts after mapping CCSs to the mouse genome (GRCm38 build)
- Replicate 2 yields exactly 227,902 mappable transcripts
- The combined 15 single-cell libraries yield 191,023 transcripts
- The summary table reports 11,538 total genes covered
- No counted CCS is assigned to multiple genes and no counted alignment carries a supplementary alignment tag

## Limitations

- Current coverage limits replicate correlation; deeper sequencing is expected to improve it
- High cost of PacBio sequencing (~5 dollars per 1-K reads) limits achievable coverage and thus detection counts

## Evidence

- [results] Article finding: [evidence span withheld — anchor status: not_found]
- [other] Validation criterion: [evidence span withheld — anchor status: not_found]
- [discussion] Filtering rule: "CCSs assigned to multiple genes were discarded."
- [discussion] Mapping step: "Clean CCS reads were aligned to mouse reference genome (mm10) by using minimap2 (v2.15-r905)"
