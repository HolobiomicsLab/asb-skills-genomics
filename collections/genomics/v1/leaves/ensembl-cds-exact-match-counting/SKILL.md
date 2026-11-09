---
name: ensembl-cds-exact-match-counting
description: Use when you have predicted transcript FASTA sequences from one or more
  error-correction/clustering methods and need to count exact nucleotide-level matches
  against a small reference CDS set to compare method precision on highly similar
  gene families.
when_to_use_negative:
- Predicted sequences do not span the target coding region end-to-end (short or partial
  reads cannot achieve exact full-CDS matches, since IsoCon assumes reads span the
  region of interest end-to-end)
- The benchmark goal is approximate similarity or gene-copy assignment rather than
  exact nucleotide identity counting
- Transcripts have already been reference-aligned/annotated by a splice-aware aligner,
  making de novo exact-match counting redundant
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: IsoCon
  role: Produces the de novo corrected candidate transcripts (final_candidates.fa)
    from FLNC Iso-Seq reads that are counted against the Ensembl CDS set; also accepts
    external candidate sets for stat_filter-style evaluation
  repo: https://github.com/ksahlin/IsoCon
- name: parasail
  role: SIMD pairwise sequence alignment library (Smith-Waterman local, Needleman-Wunsch
    global, semi-global) used to align candidate transcripts to reference CDSs and
    identify exact matches
  repo: https://github.com/jeffdaily/parasail-python
- name: edlib
  role: Edit-distance alignment library usable for the nucleotide-level exact-match
    check between transcripts and Ensembl CDSs
  repo: https://github.com/Martinsos/edlib
- name: ICE
  role: PacBio's standard Iso-Seq error-correction/clustering tool whose transcripts
    serve as the comparison baseline in the exact-match benchmark
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-018-06910-x
    title: Deciphering highly similar multigene family transcripts from Iso-Seq data
      with IsoCon
  source_content_hash: sha256:97e183115f370852ad03eead60e3fa255806ce5a2d6ac01cbcb5f2e00be4e945
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ksahlin/IsoCon
  tools:
  - ICE
  - IsoCon
  - edlib
  - parasail
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# ensembl-cds-exact-match-counting

## Summary

Benchmarking skill for quantifying how many de novo-reconstructed transcript sequences (e.g., IsoCon vs ICE candidates from targeted Iso-Seq of Y chromosome ampliconic gene families) exactly match, at the nucleotide level, a curated set of reference coding sequences (61 unique Ensembl CDSs from nine ampliconic gene families).

## When to use

Use when you have predicted transcript FASTA sequences from one or more error-correction/clustering methods and need to count exact nucleotide-level matches against a small reference CDS set to compare method precision on highly similar gene families.

## When NOT to use

- Predicted sequences do not span the target coding region end-to-end (short or partial reads cannot achieve exact full-CDS matches, since IsoCon assumes reads span the region of interest end-to-end)
- The benchmark goal is approximate similarity or gene-copy assignment rather than exact nucleotide identity counting
- Transcripts have already been reference-aligned/annotated by a splice-aware aligner, making de novo exact-match counting redundant

## Inputs

- FASTA of predicted transcripts per method (e.g., IsoCon final_candidates.fa, ICE-derived candidates)
- FASTA of 61 unique Ensembl coding sequences covering the nine Y chromosome ampliconic gene families
- Per-sample transcript sets from two testes samples for the shared-transcript intersection (FLNC reads in fastq preferred so IsoCon can use quality values)

## Outputs

- Count of transcripts shared between the two samples per method by exact sequence identity
- Count of exact nucleotide-level matches to the Ensembl CDS set per method
- Per-method comparison table (IsoCon vs ICE vs Illumina-corrected vs original reads)

## How to apply

Collect each method's predicted transcripts (e.g., IsoCon's final_candidates.fa produced by 'IsoCon pipeline -fl_reads reads.fastq -outfolder out', and ICE transcripts imported via a stat_filter-style evaluation so external candidate sets can be compared without originating from IsoCon's error-correction step). Optionally first intersect per-sample transcript sets by exact sequence identity to find transcripts shared between the two testes samples. Align every shared/predicted transcript against the 61 unique Ensembl coding sequences of the nine ampliconic gene families using fast pairwise alignment (parasail for Smith-Waterman/Needleman-Wunsch/semi-global alignments, or edlib for edit distance) and count only alignments with zero nucleotide differences as exact matches. Tally per-method metrics: number of shared transcripts and number of Ensembl exact matches; correct application reproduces IsoCon's ~121 shared transcripts (~32% of transcripts) and 21 exact Ensembl matches versus 8 for ICE.

## Related tools

- **IsoCon** (Produces the de novo corrected candidate transcripts (final_candidates.fa) from FLNC Iso-Seq reads that are counted against the Ensembl CDS set; also accepts external candidate sets for stat_filter-style evaluation) — https://github.com/ksahlin/IsoCon
- **parasail** (SIMD pairwise sequence alignment library (Smith-Waterman local, Needleman-Wunsch global, semi-global) used to align candidate transcripts to reference CDSs and identify exact matches) — https://github.com/jeffdaily/parasail-python
- **edlib** (Edit-distance alignment library usable for the nucleotide-level exact-match check between transcripts and Ensembl CDSs) — https://github.com/Martinsos/edlib
- **ICE** (PacBio's standard Iso-Seq error-correction/clustering tool whose transcripts serve as the comparison baseline in the exact-match benchmark)

## Examples

```
IsoCon pipeline -fl_reads reads.fastq -outfolder /path/to/output
```

## Evaluation signals

- Exact-match count for IsoCon transcripts against the 61 Ensembl CDSs equals ~21, while ICE yields ~8
- Shared-transcript intersection between the two testes samples yields ~121 transcripts for IsoCon (~32%), versus 11% for Illumina-corrected reads and <2% for ICE and original reads
- Matched transcripts show zero nucleotide differences (edit distance 0) across the full CDS length in parasail/edlib alignments
- Every method's candidate transcripts fall within the set of the nine ampliconic gene family targets (amplified via RT-PCR primers in the first and last coding exons)

## Limitations

- IsoCon results in the paper were obtained with commit 79589f3 on GitHub, so counts may differ with other versions
- IsoCon assumes reads are pre-processed with the Iso-Seq bioinformatic pipeline (primer/barcode removal, chimera filtering); unprocessed input will invalidate counts
- IsoCon does not distinguish alleles from distinct gene copies, and low-abundance transcripts may be missed because deep PacBio sequencing is cost-prohibitive; very low-coverage transcripts will not appear as exact matches

## Evidence

- [abstract] We apply IsoCon to nine Y chromosome ampliconic gene families and show that it outperforms existing methods on both experimental and simulated data.: "We apply IsoCon to nine Y chromosome ampliconic gene families and show that it outperforms existing methods on both experimental and simulated data."
- [discussion] IsoCon repo and fixed commit: "IsoCon is open-source and freely available at https://github.com/ksahlin/IsoCon. The results of IsoCon in this paper were obtained with commit 79589f3 on GitHub."
- [discussion] IsoCon_Eval scripts: "Scripts for all analyses are available at https://github.com/ksahlin/IsoCon_Eval. This repository also include a snakemake63 workflow"
- [readme] parasail alignment algorithms: [evidence span withheld — anchor status: external_doc]
- [readme] IsoCon output file: [evidence span withheld — anchor status: external_doc]
