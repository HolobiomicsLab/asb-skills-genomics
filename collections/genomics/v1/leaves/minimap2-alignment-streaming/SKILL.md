---
name: minimap2-alignment-streaming
description: Use when predicting HLA-I/-II alleles from whole-genome shotgun ONT or
  PacBio long-read FASTQ datasets, where a SAM-compatible aligner can be piped straight
  into HLAminer in stream mode.
when_to_use_negative:
- Input reads are Illumina-sized short reads (best results come from the HPTASR targeted-assembly
  path with reads >=100 bp, ideally 150 bp)
- Alignments already exist as a stored SAM/BAM file and you only need to run HLAminer
  scoring — rerunning minimap2 is unnecessary
- The aligner you must use cannot emit SAM-format output, since HLAminer v1.4 streaming
  depends on the (.sam) output of modern read aligners
edam_operation: http://edamontology.org/operation_0446
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: minimap2
  role: Long-read aligner run with the ONT preset (-ax map-ont --MD); its SAM output
    is piped directly into HLAminer in stream mode
  repo: https://github.com/lh3/minimap2
- name: HLAminer
  role: Receives the streamed SAM alignments and scores HLA class I and II allele
    predictions (HPRA stream mode)
  repo: https://github.com/bcgsc/HLAminer
- name: HLAminer.pl
  role: Perl driver invoked with stream-mode parameters (-h, -s 500, -q 1, -i 1, -p
    hla_nom_p.txt, -a stream) that produces HLAminer_HPRA.csv/.log
  repo: https://github.com/bcgsc/HLAminer
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/cpz1.70124
    title: Streaming Long‐Read Sequence Alignments for HLA Predictions Using HLAminer
  source_content_hash: sha256:d1e3e551f00874bcdc17910c9e423b720fd27efd54afd32b1bf719f01fd93a65
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - HLAminer
  - HLAminer.pl
  - minimap2
  edam_operation: http://edamontology.org/operation_0446
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# minimap2-alignment-streaming

## Summary

Streams minimap2 SAM alignment output of long-read WGS data directly into HLAminer v1.4 (HPRA stream mode) to predict HLA class I and II alleles without persisting bulky alignment files on disk.

## When to use

Use when predicting HLA-I/-II alleles from whole-genome shotgun ONT or PacBio long-read FASTQ datasets, where a SAM-compatible aligner can be piped straight into HLAminer in stream mode.

## When NOT to use

- Input reads are Illumina-sized short reads (best results come from the HPTASR targeted-assembly path with reads >=100 bp, ideally 150 bp)
- Alignments already exist as a stored SAM/BAM file and you only need to run HLAminer scoring — rerunning minimap2 is unnecessary
- The aligner you must use cannot emit SAM-format output, since HLAminer v1.4 streaming depends on the (.sam) output of modern read aligners

## Inputs

- Whole-genome shotgun long-read FASTQ (e.g., NA24385 ONT PromethION, guppy v5.0.6 at ~67x or dorado kit14-2022-12 at ~72x coverage)
- Combined reference FASTA: GRCh38 chr-only with chr6 excluded/MHC masked, concatenated with IMGT/HLA HLA-I and HLA-II genomic allele sequences (HLA-I_II_GEN.fasta)
- hla_nom_p.txt P-designation allele file from hla.alleles.org

## Outputs

- HLAminer_HPRA.csv — concise per-gene summary keeping the top two highest-scoring HLA-I/-II predictions
- HLAminer_HPRA.log — full log listing all scored predictions

## How to apply

Build a combined long-read reference by concatenating the GRCh38 chromosome-only genome (excluding chr6, or masking the MHC region 28510120-33480577 on chr6) with up-to-date HLA-I and HLA-II genomic allele FASTA sequences from the IMGT/HLA repository. Align each WGS FASTQ with minimap2 using the ONT preset (-ax map-ont --MD) and pipe the SAM stream directly into HLAminer.pl v1.4 in HPRA stream mode with -h HLA-I_II_GEN.fasta -s 500 -q 1 -i 1 -p hla_nom_p.txt -a stream (note the relaxed stream-mode thresholds -s 500, -q 1, -i 1 versus the stricter defaults -s 1000, -q 30, -i 99). Collect HLAminer_HPRA.csv and HLAminer_HPRA.log, then take the top two highest-scoring predictions per HLA gene for comparison against reference types. The rationale is that streaming removes the need to write large SAM/BAM files to disk while keeping the aligner's parameters the only decision point before HLA scoring.

## Related tools

- **minimap2** (Long-read aligner run with the ONT preset (-ax map-ont --MD); its SAM output is piped directly into HLAminer in stream mode) — https://github.com/lh3/minimap2
- **HLAminer** (Receives the streamed SAM alignments and scores HLA class I and II allele predictions (HPRA stream mode)) — https://github.com/bcgsc/HLAminer
- **HLAminer.pl** (Perl driver invoked with stream-mode parameters (-h, -s 500, -q 1, -i 1, -p hla_nom_p.txt, -a stream) that produces HLAminer_HPRA.csv/.log) — https://github.com/bcgsc/HLAminer

## Examples

```
minimap2 -ax map-ont --MD GRCh38+HLA-I_II_GEN.fa NA24385.fq.gz | ../bin/HLAminer.pl -h HLA-I_II_GEN.fasta -s 500 -q 1 -i 1 -p hla_nom_p.txt -a stream
```

## Evaluation signals

- HLAminer_HPRA.csv and HLAminer_HPRA.log are produced for each streamed dataset with no intermediate SAM file needing to be stored
- Top two highest-scoring predictions per HLA gene match clinically confirmed/PacBio reference types for NA24385 (e.g., B*35:01, DRB1*04:03P)
- Class I and II allele calls are concordant across independently base-called datasets (guppy v5.0.6 vs dorado kit14-2022-12), demonstrating robustness across base-callers
- minimap2 runs with the ONT preset (-ax map-ont --MD) against the combined reference in which chr6 (or at least MHC 28510120-33480577) is removed/masked so only HLA allele references are hit

## Limitations

- Stream-mode HLAminer thresholds (-s 500, -q 1, -i 1) are more permissive than the tool defaults (-s 1000, -q 30, -i 99); larger k values or stricter thresholds trade sensitivity for specificity
- Prediction quality depends on reference recency — the protocol requires continually updated IMGT/HLA sequences and the P-designation file hla_nom_p.txt
- Robustness claims rest on high-coverage PromethION data (39x-72x); very low-coverage or older-chemistry datasets may yield less accurate calls, though the paper reports robust predictions even at ~10x

## Evidence

- [abstract] streaming protocol: "a streamlined protocol for predicting HLA signatures from whole-genome shotgun (WGS) long-read sequencing data by directly streaming sequence alignments into HLAminer"
- [abstract] diskless streaming: "works with any read aligner compatible with the SAM file format—eliminating the need to store bulky alignment files on disk"
- [methods] workflow command: [evidence span withheld — anchor status: not_found]
- [readme] v1.4 streaming feature: "Ability to stream the (.sam) output of modern read  aligners, directly into HLAminer."
