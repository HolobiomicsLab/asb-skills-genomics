---
name: repeatmasker-te-annotation
description: Use when generating or curating a truth callset of non-reference transposable
  element insertions from long-read assemblies or read-level candidate insertions,
  and per-family TE labels with divergence/length filtering are needed for benchmarking.
when_to_use_negative:
- The goal is de novo TE insertion detection from long reads rather than annotating/labeling
  already-extracted candidate insertions — use a caller like sTELLeR directly (it
  takes --bam, --ref, --TE_fasta and outputs a VCF).
- The analysis targets ancient/divergent TE insertions, since the <15% consensus-divergence
  filter is designed specifically to enrich for recent, intact insertions.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3175
tools:
- name: RepeatMasker
  role: Aligns extracted insertion sequences against TE consensus sequences and assigns
    TE family labels (Alu, L1, HERV, SVA) used for truth-set annotation
- name: minimap2
  role: Aligns candidate insertion sequences / assemblies used in the truth-set extraction
    workflow
- name: samtools
  role: BAM/SAM handling in the assembly-based insertion calling and annotation workflow
- name: sTELLeR
  role: Alternative/complementary caller whose intermediate files (ID_chr_reads.fasta,
    ID_chr_repeats.sam) contain candidate insertions and insertions aligned to TEs;
    the truth set produced by this skill is used to benchmark sTELLeR
  repo: https://github.com/kristinebilgrav/sTELLeR
- name: SVIM-asm
  role: Assembly-based insertion caller run on HPRC de novo assemblies of HG002 and
    HG01071 to extract candidate insertion sequences for annotation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae686
    title: Detecting transposable elements in long-read genomes using sTELLeR
  source_content_hash: sha256:583c7df92a77a8007e668700102dbe862f0c7d38749444a6635c6a9a179c2ba0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kristinebilgrav/sTELLeR
  tools:
  - RepeatMasker
  - SVIM-asm
  - minimap2
  - python3
  - sTELLeR
  - samtools
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# repeatmasker-te-annotation

## Summary

Annotate candidate insertion sequences with RepeatMasker by aligning them against TE consensus sequences to assign TE family labels (Alu, L1, HERV, SVA), then filter for recent intact insertions using a divergence and family-specific length thresholds. This is the approach used to build assembly-based truth callsets for benchmarking long-read TE callers like sTELLeR.

## When to use

Use when generating or curating a truth callset of non-reference transposable element insertions from long-read assemblies or read-level candidate insertions, and per-family TE labels with divergence/length filtering are needed for benchmarking.

## When NOT to use

- The goal is de novo TE insertion detection from long reads rather than annotating/labeling already-extracted candidate insertions — use a caller like sTELLeR directly (it takes --bam, --ref, --TE_fasta and outputs a VCF).
- The analysis targets ancient/divergent TE insertions, since the <15% consensus-divergence filter is designed specifically to enrich for recent, intact insertions.

## Inputs

- FASTA file of candidate insertion sequences (e.g., from SVIM-asm on de novo assemblies, or sTELLeR intermediate ID_chr_reads.fasta files)
- FASTA file of TE consensus sequences (e.g., TEfastasequences.fa containing L1 and Alu subfamilies, or TEsequences_SVA_HERV_ALU_L1.fasta also containing SVA and HERV)
- Reference genome assembly or alignment (minimap2/samtools BAM) used for coordinate placement of insertions

## Outputs

- RepeatMasker-annotated insertion records with TE family labels (e.g., ID_chr_repeats.sam alignment of insertions to TEs)
- Filtered per-sample truth callset tables of Alu, L1, HERV, and SVA insertions passing <15% divergence and family-specific length thresholds

## How to apply

First extract candidate insertion sequences (e.g., with SVIM-asm on HPRC de novo assemblies, or from the intermediate ID_chr_reads.fasta files that sTELLeR emits with --keep_intermediates). Align the extracted insertion sequences against TE consensus sequences and annotate them with RepeatMasker to assign TE family labels. Filter annotated insertions to retain only those with sequence divergence below 15% relative to the TE consensus, which enriches for recent intact insertions. Apply TE-family-specific minimum length filters of 300 bp for SVA, 1500 bp for HERV, 200 bp for L1, and 100 bp for Alu, then compile the filtered calls into per-sample truth callset tables. Validate by confirming per-family counts match expected truth set sizes (e.g., HG002: 2193 Alu, 492 L1, 14 HERV, 205 SVA; HG01071: 2262 Alu, 439 L1, 11 HERV, 202 SVA).

## Related tools

- **RepeatMasker** (Aligns extracted insertion sequences against TE consensus sequences and assigns TE family labels (Alu, L1, HERV, SVA) used for truth-set annotation)
- **minimap2** (Aligns candidate insertion sequences / assemblies used in the truth-set extraction workflow)
- **samtools** (BAM/SAM handling in the assembly-based insertion calling and annotation workflow)
- **sTELLeR** (Alternative/complementary caller whose intermediate files (ID_chr_reads.fasta, ID_chr_repeats.sam) contain candidate insertions and insertions aligned to TEs; the truth set produced by this skill is used to benchmark sTELLeR) — https://github.com/kristinebilgrav/sTELLeR
- **SVIM-asm** (Assembly-based insertion caller run on HPRC de novo assemblies of HG002 and HG01071 to extract candidate insertion sequences for annotation)

## Examples

```
python steller/steller.py --ref HG38_chr22.fa --TE_fasta fasta/TEfastasequences.fa --bam testdata.bam --sr 4 --style pb -o testdata_res -mr 80 -k
```

## Evaluation signals

- Per-family truth set counts match the reported sizes: HG002 2193 Alu, 492 L1, 14 HERV, 205 SVA; HG01071 2262 Alu, 439 L1, 11 HERV, 202 SVA
- All retained insertions have sequence divergence below 15% relative to their TE consensus
- Retained insertions satisfy family-specific minimum lengths: SVA >= 300 bp, HERV >= 1500 bp, L1 >= 200 bp, Alu >= 100 bp
- Annotated insertions carry TE family labels consistent with the consensus sequences in the TE FASTA used (e.g., AluJb, AluY, AluYa5, L1HS, L1PA2, SVA, HERV)
- If applied to sTELLeR intermediates, the annotated SAM (ID_chr_repeats.sam) covers the same candidate insertions present in the kept ID_chr_reads.fasta files

## Limitations

- The <15% divergence filter intentionally restricts the truth set to recent, intact insertions; older or highly diverged elements are excluded.
- Family-specific length cutoffs (SVA 300, HERV 1500, L1 200, Alu 100) mean truncated or partial insertions below these thresholds are dropped and cannot appear in the truth set.
- sTELLeR intermediates (candidate insertion FASTAs and insertion-to-TE SAMs) are deleted by default unless --keep_intermediates is set, so annotation inputs must be explicitly retained.

## Evidence

- [methods] Applying the >15% divergence and minimum consecutive length filters (SVA 300, HERV 1500, L1 200, Alu 100) to SVIM-asm/RepeatMasker calls yielded 2193 Alu, 492 L1, 14 HERV, and 205 SVA in HG002: [evidence span withheld — anchor status: not_found]
- [methods] Run assembly-based insertion calling with SVIM-asm on the HPRC de novo assemblies of HG002 and HG01071 to extract candidate insertion sequences.: [evidence span withheld — anchor status: not_found]
- [methods] Align the extracted insertion sequences against TE consensus sequences and annotate them with RepeatMasker to assign TE family labels.: [evidence span withheld — anchor status: not_found]
- [methods] Filter annotated insertions to retain only those with sequence divergence below 15% relative to the TE consensus, enriching for recent intact insertions.: [evidence span withheld — anchor status: not_found]
- [readme] intermediate files containing fasta sequences of candidate insertions (ID_chr_reads.fasta) and candiate insertions aligned to TEs (ID_chr_repeats.sam) will be removed: "intermediate files containing fasta sequences of candidate insertions (ID_chr_reads.fasta) and candiate insertions aligned to TEs (ID_chr_repeats.sam) will be removed"
