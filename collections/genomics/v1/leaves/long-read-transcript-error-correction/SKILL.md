---
name: long-read-transcript-error-correction
description: Use when long noisy reads from targeted PacBio Iso-Seq (or ONT/R2C2 CCS-style)
  sequencing of highly similar gene copies or amplicons must be error-corrected and
  collapsed into finished consensus transcripts, including isoform-capture versus
  sequencing-depth analysis.
when_to_use_negative:
- The input consists of short reads (e.g., Illumina) or reads that do not span the
  region of interest end-to-end — IsoCon requires reads spanning the full transcript/amplicon
  for iterative clustering to converge.
- The goal is to assign transcripts to individual gene copies or determine gene family
  copy number from transcriptome-only data — the article explicitly states IsoCon
  does not allow conclusive gene-copy assignment and downstream analysis is needed.
- A reference-based transcriptome reconstruction (e.g., with a complete reference
  genome) is available and sufficient; IsoCon is designed for de novo reconstruction
  of highly similar copies without a reference.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0080
tools:
- name: IsoCon
  role: Core de novo error-correction and candidate-transcript reconstruction tool
    run on pre-processed PacBio CCS reads
  repo: https://github.com/ksahlin/IsoCon
- name: parasail
  role: SIMD Smith-Waterman/semi-global pairwise alignments of each read to the cluster
    center during the correction phase
  repo: https://github.com/jeffdaily/parasail-python
- name: edlib
  role: Lightweight edit-distance alignment library used in the IsoCon toolchain
- name: IsoCon_Eval
  role: Snakemake pipeline for the simulated depth experiments (gene members, read
    counts, replicates) used to evaluate capture thresholds
  repo: https://github.com/ksahlin/IsoCon_Eval
- name: ICE
  role: PacBio's standard de novo error-correction baseline against which IsoCon precision/recall
    is compared
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-018-06910-x
    title: Deciphering highly similar multigene family transcripts from Iso-Seq data
      with IsoCon
  source_content_hash: sha256:907aa88e5c9431f0031c5727af414b1e94a2948c1ed0135e764d5f3d2875db05
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ksahlin/IsoCon
  tools:
  - ICE
  - IsoCon
  - IsoCon_Eval
  - edlib
  - parasail
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# long-read-transcript-error-correction

## Summary

De novo error correction and redundancy removal of PacBio CCS Iso-Seq reads with IsoCon to reconstruct end-to-end transcripts from highly similar multigene family copies (e.g., TSPY, HSFY, DAZ) at nucleotide-level precision, without a reference genome.

## When to use

Use when long noisy reads from targeted PacBio Iso-Seq (or ONT/R2C2 CCS-style) sequencing of highly similar gene copies or amplicons must be error-corrected and collapsed into finished consensus transcripts, including isoform-capture versus sequencing-depth analysis.

## When NOT to use

- The input consists of short reads (e.g., Illumina) or reads that do not span the region of interest end-to-end — IsoCon requires reads spanning the full transcript/amplicon for iterative clustering to converge.
- The goal is to assign transcripts to individual gene copies or determine gene family copy number from transcriptome-only data — the article explicitly states IsoCon does not allow conclusive gene-copy assignment and downstream analysis is needed.
- A reference-based transcriptome reconstruction (e.g., with a complete reference genome) is available and sufficient; IsoCon is designed for de novo reconstruction of highly similar copies without a reference.

## Inputs

- FASTQ or FASTA file of pre-processed PacBio Iso-Seq CCS / fl reads (primers, barcodes and chimeras already removed)
- Optional ccs BAM via --ccs flag
- Per-isoform read counts table for capture-depth classification

## Outputs

- final_candidates.fa — converged, statistically filtered candidate transcripts
- not_converged.fa — reads/candidates not converged or lost in error correction
- cluster_info.tsv — per-cluster/candidate read assignment information
- Isoform fate classification (captured / stat-filtered / error-correction-lost) as a function of experiment depth and per-isoform depth

## How to apply

Pre-process reads with the Iso-Seq pipeline (remove primers, barcodes, chimeras), then run 'IsoCon pipeline -fl_reads <reads.fastq/fasta> -outfolder <dir>' to produce final_candidates.fa, not_converged.fa, and cluster_info.tsv. IsoCon runs a cautiously iterative clustering and error-correction phase (pairwise parasail alignments to a cluster center, correcting only the half of variant positions with lowest support) followed by iterative statistical filtering that greedily assigns reads to closest-neighbor candidates and performs pairwise hypothesis testing, removing candidates with significance value above alpha (default 0.01), at most tau candidates per iteration; candidates with more than 10 variant positions relative to all others are not statistically evaluated. Classification of isoform fate (captured in final_candidates.fa, filtered in the statistical step, or lost in error correction / present only in not_converged.fa) can then be computed from cluster_info.tsv together with per-isoform read counts. Results are reproducible with commit 79589f3 of the IsoCon repository, and full depth-experiment workflows are scripted with snakemake in IsoCon_Eval (config: simulated_experiments.json with GENE_MEMBERS, READ_COUNTS, RUNS, ABUNDANCE, TOOLS fields).

## Related tools

- **IsoCon** (Core de novo error-correction and candidate-transcript reconstruction tool run on pre-processed PacBio CCS reads) — https://github.com/ksahlin/IsoCon
- **parasail** (SIMD Smith-Waterman/semi-global pairwise alignments of each read to the cluster center during the correction phase) — https://github.com/jeffdaily/parasail-python
- **edlib** (Lightweight edit-distance alignment library used in the IsoCon toolchain)
- **IsoCon_Eval** (Snakemake pipeline for the simulated depth experiments (gene members, read counts, replicates) used to evaluate capture thresholds) — https://github.com/ksahlin/IsoCon_Eval
- **ICE** (PacBio's standard de novo error-correction baseline against which IsoCon precision/recall is compared)

## Examples

```
IsoCon pipeline -fl_reads reads.fastq -outfolder out/ && snakemake --configfile simulated_experiments.json -n
```

## Evaluation signals

- Isoform-capture thresholds reproduced: most TSPY transcripts captured at per-isoform depth >3 reads; HSFY threshold approximately 10 reads.
- final_candidates.fa should contain converged candidates while reads lost in error correction appear only in not_converged.fa, consistent with cluster_info.tsv read assignments.
- Statistical filtering invariant: at most tau candidates with p-value above alpha (0.01) are removed per iteration; candidates with >10 variant positions relative to all others bypass the statistical test.
- Precision and recall on simulated data should exceed ICE across ranges of sequencing depths, transcript lengths, similarities and abundance levels.
- Snakemake dry run of IsoCon_Eval ('snakemake --configfile simulated_experiments.json -n') resolves without missing paths after config modification.

## Limitations

- IsoCon requires pre-processing: it assumes reads have had primers, barcodes and chimeras removed by the Iso-Seq pipeline beforehand.
- It cannot distinguish alleles from distinct gene copies without further downstream analysis, nor determine gene family copy number from transcriptome data alone.
- Novel gene copies are difficult to distinguish from splice variants or RNA editing.
- RT-PCR error sources cannot be excluded; replicate cDNA libraries and further validation are still needed.
- Very low-abundance transcripts may not be captured since deep PacBio sequencing can be cost-prohibitive.
- Candidate transcripts failing to converge end up in not_converged.fa and are lost from the final candidate set.

## Evidence

- [intro] we develop IsoCon, a de novo algorithm for error-correcting and removing redundancy of PacBio circular consensus sequence (CCS) reads: "we develop IsoCon, a de novo algorithm for error-correcting and removing redundancy of PacBio circular consensus sequence (CCS) reads"
- [readme] IsoCon pipeline command and outputs: "IsoCon pipeline -fl_reads <reads.fastq> -outfolder </path/to/output>"
- [readme] outputs final_candidates.fa and not_converged.fa: [evidence span withheld — anchor status: external_doc]
- [discussion] significance threshold alpha default 0.01: "Then, we identify the candidates with pi greater than a significance threshold α. This α is a parameter to our algorithm, set by default to 0.01."
- [discussion] pairwise alignments to center using parasail: "We first create pairwise alignments from each read to the center using parasail60"
- [discussion] pre-processing requirement: "IsoCon assumes the reads have been pre-processed with the Iso-Seq bioinformatic pipeline to remove primers, barcodes, and reads that are chimeras"
- [discussion] results obtained with commit 79589f3: "The results of IsoCon in this paper were obtained with commit 79589f3 on GitHub."
- [results] capture depth finding from task card: [evidence span withheld — anchor status: not_found]
- [discussion] candidates with >10 variant positions not statistically evaluated: "Candidate transcripts that have more than a threshold of variant positions (default of 10) relative to all other candidate transcripts are not statistically evaluated"
