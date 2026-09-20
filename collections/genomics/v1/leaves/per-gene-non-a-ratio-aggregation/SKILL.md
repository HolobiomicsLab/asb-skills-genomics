---
name: per-gene-non-a-ratio-aggregation
description: Use when you have per-read internal non-A residue calls (position, count,
  tail length) from PAIso-seq CCS reads and need transcript/gene-level non-A fractions
  comparable to Fig. 2c of the PAIso-seq paper.
when_to_use_negative:
- The goal is quantifying non-A modifications at the very last 3'-end base of the
  poly(A) tail — PAIso-seq cannot accurately quantify terminal non-A residues because
  terminal G/C/U pair to the adaptor and are lost.
- Input data are short-read or non-PacBio (e.g., Illumina/TAIL-seq) where internal
  poly(A) tail bases are not directly read and per-read non-A positions cannot be
  called.
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3512
tools:
- name: ccs
  role: Generates circular consensus sequences (CCS) from subreads; pass counts (>=10)
    gate which reads contribute non-A residue calls
- name: minimap2
  role: Splice-aware alignment of cleaned CCS reads to the mouse GRCm38/mm10 genome
    so each read is assigned to a gene for per-gene aggregation
- name: GV_oocyte_PAIsoSeqAnalysis pipeline (run.pl, GetCCSpass.pl)
  role: Perl pipeline that trims adapters, calls poly(A) tails, and emits the per-transcript
    non-A residue table that is aggregated to per-gene ratios
  repo: github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-13228-9
    title: Poly(A) inclusive RNA isoform sequencing (PAIso−seq) reveals wide-spread
      non-adenosine residues within RNA poly(A) tails
  source_content_hash: sha256:d4c83d4dcbd91dbf1c3e83e85187765b3e4751070e6b58cb346f06597111914f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GV_oocyte_PAIsoSeqAnalysis pipeline (run.pl, GetCCSpass.pl)
  - ccs
  - ccs (version 3.4.1)
  - minimap2
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# Per-gene non-adenosine ratio aggregation in poly(A) tails

## Summary

Aggregates U/G/C residues detected inside poly(A) tail bodies (called by the PAIso-seq pipeline) into per-transcript and per-gene non-A ratios across mouse GV oocyte sequencing replicates, so that the fraction of mRNAs harboring internal non-A residues can be validated against the paper's 17% benchmark and per-gene ratios checked for replicate correlation.

## When to use

Use when you have per-read internal non-A residue calls (position, count, tail length) from PAIso-seq CCS reads and need transcript/gene-level non-A fractions comparable to Fig. 2c of the PAIso-seq paper.

## When NOT to use

- The goal is quantifying non-A modifications at the very last 3'-end base of the poly(A) tail — PAIso-seq cannot accurately quantify terminal non-A residues because terminal G/C/U pair to the adaptor and are lost.
- Input data are short-read or non-PacBio (e.g., Illumina/TAIL-seq) where internal poly(A) tail bases are not directly read and per-read non-A positions cannot be called.

## Inputs

- PAIso-seq run.pl output table of internal non-A residues per transcript (position, tail length) from mouse GV oocyte CCS reads
- CCS pass-count file (ccs.pass.txt from GetCCSpass.pl) for filtering reads with >=10 passes
- Replicate-level per-sample results from the 15 barcoded single GV oocytes

## Outputs

- Fraction of transcripts harboring internal non-A residues within poly(A) tail bodies (expected ~17%)
- Per-gene non-A residue ratios aggregated across replicates
- Replicate-by-replicate correlation matrix of per-gene non-A ratios (Fig. 2c analogue)

## How to apply

Start from the pipeline output of run.pl (run with --ccs sample.out.fasta --sample sampleName --species mm10 --minimap2_index Mus_musculus.mmi --minimap2_thread 10 --pass ccs.pass.txt), using only CCS reads with 10 or more passes to ensure base-calling accuracy. For each transcript, count U, G, and C residues inside the called poly(A) tail body, recording each non-A residue's position and the total tail length; tails qualify if the clipped sequence is at least 15 nt, contains at least five continuous adenosines, and has non-A residues below 50% of the tail and fewer than 20 in count. Compute the fraction of transcripts that contain at least one internal non-A residue and per-gene non-A ratios by aggregating transcript-level counts to gene level across the 15 barcoded single GV oocyte replicates (discarding CCSs assignable to multiple genes and ambiguous reads classified into two or more samples). Finally, compare the transcript fraction against the reported 17% and compute replicate-to-replicate correlation of per-gene non-A ratios as in Fig. 2c.

## Related tools

- **ccs** (Generates circular consensus sequences (CCS) from subreads; pass counts (>=10) gate which reads contribute non-A residue calls)
- **minimap2** (Splice-aware alignment of cleaned CCS reads to the mouse GRCm38/mm10 genome so each read is assigned to a gene for per-gene aggregation)
- **GV_oocyte_PAIsoSeqAnalysis pipeline (run.pl, GetCCSpass.pl)** (Perl pipeline that trims adapters, calls poly(A) tails, and emits the per-transcript non-A residue table that is aggregated to per-gene ratios) — github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis

## Examples

```
perl run.pl --ccs sample.out.fasta --sample sampleName --species mm10 --minimap2_index Mus_musculus.mmi --minimap2_thread 10 --pass ccs.pass.txt &>run.log
```

## Evaluation signals

- The fraction of transcripts with internal U/G/C residues inside poly(A) tail bodies is consistent with the reported 17% in mouse GV oocytes.
- Per-gene non-A ratios correlate across sequencing replicates, as shown in the paper's Fig. 2c.
- Only reads with CCS pass number >=10 are included; tails failing any filter (clipped sequence <15 nt, <5 continuous adenosines, non-A >=50% or >=20 in count) are excluded.
- Per-gene ratios are reproducible after discarding CCSs assigned to multiple genes and ambiguous reads classified into two or more samples.
- Every internal non-A residue record carries a position within the tail and the total tail length, so ratios can be recomputed independently.

## Limitations

- PAIso-seq cannot accurately quantify the frequency of non-A modifications at the very last 3'-end base of poly(A) tails.
- A minimal number of additional A residues may be introduced during pairing of the extension primer, biasing terminal base calling.
- Correlation between replicates (Fig. 2c) is limited by coverage and is expected to improve with deeper PacBio sequencing.

## Evidence

- [abstract] 17% of the mRNAs harbor non-A residues within the body of poly(A) tails in mouse GV oocytes: "we find that 17% of the mRNAs harbor non-A residues within the body of poly(A) tails in mouse GV oocytes"
- [discussion] Pass-number and tail filter thresholds: "We require a pass number of 10 or more to ensure the accuracy of base calling."
- [discussion] Non-A residue filter thresholds: "the percentage of the non-A residues was <50% and the number of non-A residues were <20"
- [readme] Pipeline invocation from README: "perl run.pl --ccs sample.out.fasta --sample sampleName --species mm10 --minimap2_index Mus_musculus.mmi --minimap2_thread  10 --pass ccs.pass.txt"
- [discussion] Replicate correlation limitation: "The correlation between different replicates as we shown in Fig. 2c can be expected to be further improved if we sequence more reads."
