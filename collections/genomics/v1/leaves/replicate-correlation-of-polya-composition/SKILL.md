---
name: replicate-correlation-of-polya-composition
description: Use when PAIso-seq CCS reads have been tail-called per transcript and
  you need per-gene non-A ratios across replicates plus a replicate-correlation check
  (Fig. 2c) to validate the 17% internal non-A finding.
when_to_use_negative:
- Estimating the frequency of non-A residues at the very last 3′-end base of the tail
  — PAIso-seq cannot accurately quantify terminal non-A modifications because terminal
  G/C/U pair to the adaptor and are lost
- Short-read poly(A) tail data (e.g. TAIL-seq-style or Illumina PAL-seq) that cannot
  resolve tail bodies or tails longer than ~231–250 nt
- Datasets without multiple biological replicates (e.g. a single oocyte library),
  where per-gene ratio correlation cannot be assessed
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_0622
tools:
- name: minimap2
  role: Splice-aware alignment of trimmed CCS reads to the mouse GRCm38 genome to
    assign reads to genes before per-gene non-A ratio aggregation
  repo: https://github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis
- name: ccs
  role: Generates circular consensus sequences from PacBio Sequel subreads (v3.4.1,
    --richQVs recommended) whose pass counts gate which reads enter the non-A residue
    analysis
  repo: https://github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis
- name: run.pl (PAIso-Seq pipeline)
  role: Perl pipeline that calls poly(A) tails and non-A residues per transcript using
    the minimap2 index and per-read pass counts
  repo: https://github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis
- name: GetCCSpass.pl
  role: Collects per-CCS-read pass counts from the movie CCS BAM into ccs.pass.txt,
    used to enforce the ≥10-pass accuracy filter
  repo: https://github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-13228-9
    title: Poly(A) inclusive RNA isoform sequencing (PAIso−seq) reveals wide-spread
      non-adenosine residues within RNA poly(A) tails
  source_content_hash: sha256:05d521c30e930376d4db22631adf0c1c658b4626b2752f91f76b630d0d798079
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis
  tools:
  - GetCCSpass.pl
  - ccs
  - ccs (version 3.4.1)
  - minimap2
  - run.pl (PAIso-Seq pipeline)
  edam_operation: http://edamontology.org/operation_3478
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# replicate-correlation-of-polya-composition

## Summary

Aggregates internally detected U/G/C residues in PAIso-seq poly(A) tails to per-gene non-A ratios across sequencing replicates of mouse GV oocytes, and assesses replicate correlation (Fig. 2c) to validate the reported 17% of mRNAs harboring internal non-A residues.

## When to use

Use when PAIso-seq CCS reads have been tail-called per transcript and you need per-gene non-A ratios across replicates plus a replicate-correlation check (Fig. 2c) to validate the 17% internal non-A finding.

## When NOT to use

- Estimating the frequency of non-A residues at the very last 3′-end base of the tail — PAIso-seq cannot accurately quantify terminal non-A modifications because terminal G/C/U pair to the adaptor and are lost
- Short-read poly(A) tail data (e.g. TAIL-seq-style or Illumina PAL-seq) that cannot resolve tail bodies or tails longer than ~231–250 nt
- Datasets without multiple biological replicates (e.g. a single oocyte library), where per-gene ratio correlation cannot be assessed

## Inputs

- Per-read poly(A) tail calls from PAIso-seq CCS reads (clipped tail sequences with per-base identities and positions), restricted to reads with ≥10 passes
- CCS BAM file and ccs.pass.txt from GetCCSpass.pl
- Gene assignments for CCS reads (minimap2 splice alignments to Mus_musculus.GRCm38.dna.toplevel.chromosome.fa)

## Outputs

- Per-transcript table of internal non-A residue types (U/G/C), positions within the tail, and tail lengths
- Per-gene non-A ratios per sequencing replicate
- Replicate-versus-replicate correlation of per-gene non-A ratios (Fig. 2c equivalent)
- Overall fraction of transcripts harboring internal non-A residues (expected ≈17%)

## How to apply

For each CCS read passing ≥10 passes with a poly(A) tail ≥15 nt containing ≥5 continuous adenosines and with non-A residues <50% and <20 in count, record the type (U/G/C) and position of every non-A residue inside the tail body along with the tail length. Assign each CCS to a gene, discarding CCSs assignable to multiple genes, and aggregate per-transcript non-A counts into per-gene non-A ratios for each replicate (each replicate is a barcoded single GV oocyte). Compute correlation of per-gene non-A ratios between replicates as in the paper's Fig. 2c, expecting moderate correlation that improves with deeper sequencing. Finally, verify the overall fraction of transcripts with internal non-A residues is consistent with the reported 17% in mouse GV oocytes. The analysis uses the Perl/R pipeline (run.pl, GetCCSpass.pl) from the GV_oocyte_PAIsoSeqAnalysis repository with minimap2 v2.15-r905 splice alignments to GRCm38/mm10.

## Related tools

- **minimap2** (Splice-aware alignment of trimmed CCS reads to the mouse GRCm38 genome to assign reads to genes before per-gene non-A ratio aggregation) — https://github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis
- **ccs** (Generates circular consensus sequences from PacBio Sequel subreads (v3.4.1, --richQVs recommended) whose pass counts gate which reads enter the non-A residue analysis) — https://github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis
- **run.pl (PAIso-Seq pipeline)** (Perl pipeline that calls poly(A) tails and non-A residues per transcript using the minimap2 index and per-read pass counts) — https://github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis
- **GetCCSpass.pl** (Collects per-CCS-read pass counts from the movie CCS BAM into ccs.pass.txt, used to enforce the ≥10-pass accuracy filter) — https://github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis

## Examples

```
perl run.pl --ccs sample.out.fasta --sample sampleName --species mm10 --minimap2_index Mus_musculus.mmi --minimap2_thread 10 --pass ccs.pass.txt &>run.log
```

## Evaluation signals

- Fraction of transcripts with internal non-A residues in the output table is consistent with the reported 17% in mouse GV oocytes
- Per-gene non-A ratios correlate across sequencing replicates as shown in the paper's Fig. 2c
- Every retained read satisfies the filters: ≥10 passes, clipped tail ≥15 nt with ≥5 continuous adenosines, non-A residues <50% of tail and <20 in count
- Per-gene non-A ratio aggregation only includes CCSs uniquely assigned to a single gene (multi-gene CCSs and supplementary alignments excluded)
- Observed internal non-A residues are U, G, and C; tails longer than 231 nt should be detectable in replicates where coverage permits

## Limitations

- Replicate correlation (Fig. 2c) is limited by current PacBio coverage and is expected to improve with deeper sequencing
- PAIso-seq cannot accurately quantify the frequency of non-A modifications at the very last 3′-end base, so terminal residues are excluded from the internal-residue analysis
- Chemically modified bases within poly(A) tails might introduce U, C, or G substitutions during RT, although the authors consider this very unlikely
- Very low-abundance features (e.g. a 13-kb tail detected in only one replicate) are not independently verifiable across replicates

## Evidence

- [abstract] 17% of mRNAs in mouse GV oocytes harbor non-A residues within the body of their poly(A) tails: "we find that 17% of the mRNAs harbor non-A residues within the body of poly(A) tails in mouse GV oocytes"
- [discussion] Pass-number accuracy filter: "We require a pass number of 10 or more to ensure the accuracy of base calling."
- [discussion] Tail-calling filter thresholds: "the length of the clipped sequence was no <15 nt; (2) the clipped sequence contained at least five continuous adenosines"
- [discussion] Replicate correlation improves with depth: "The correlation between different replicates as we shown in Fig. 2c can be expected to be further improved if we sequence more reads."
- [readme] Pipeline invocation from README: "perl run.pl --ccs sample.out.fasta --sample sampleName --species mm10 --minimap2_index Mus_musculus.mmi --minimap2_thread  10 --pass ccs.pass.txt"
