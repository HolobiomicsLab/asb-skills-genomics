---
name: assembly-based-insertion-calling
description: Use when building or validating a truth set of TE insertions (Alu, L1,
  HERV, SVA) from de novo long-read assemblies, or when benchmarking read-based TE
  callers against an assembly-derived callset for samples like HG002 or HG01071.
when_to_use_negative:
- Input is short-read (Illumina) data — this method depends on long-read assemblies
  or reads for resolving full-length TE insertions
- You only need to re-call insertions from an existing BAM/CRAM rather than generate
  an assembly-based truth set — a read-based caller like sTELLeR directly on the BAM
  is appropriate
- The analysis targets old/divergent insertions — the <15% divergence filter is intentionally
  biased toward recent intact insertions
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3673
tools:
- name: SVIM-asm
  role: Assembly-based extraction of candidate TE insertion sequences from HPRC de
    novo assemblies
- name: RepeatMasker
  role: Aligns candidate insertions to TE consensus sequences and assigns TE family
    labels and divergence values
- name: minimap2
  role: Alignment of insertion sequences/reads during candidate extraction and downstream
    processing
- name: samtools
  role: Alignment file handling in the calling workflow
- name: sTELLeR
  role: Long-read TE caller benchmarked against the assembly-based truth callset;
    also capable of running on genome assemblies
  repo: https://github.com/kristinebilgrav/sTELLeR
- name: python3
  role: Glue scripting for filtering and compiling truth callset tables (with pysam)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae686
    title: Detecting transposable elements in long-read genomes using sTELLeR
  source_content_hash: sha256:88aebe0ca0ad28ec401277673497718e147994efab861c8e04f22be5182ba65e
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
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# assembly-based-insertion-calling

## Summary

Generate a per-sample truth callset of non-reference transposable element insertions by calling candidate insertions from de novo genome assemblies with SVIM-asm, labeling them by TE family with RepeatMasker, and applying divergence and length filters. This assembly-based callset (HG002, HG01071) serves as the ground truth for benchmarking long-read TE callers such as sTELLeR.

## When to use

Use when building or validating a truth set of TE insertions (Alu, L1, HERV, SVA) from de novo long-read assemblies, or when benchmarking read-based TE callers against an assembly-derived callset for samples like HG002 or HG01071.

## When NOT to use

- Input is short-read (Illumina) data — this method depends on long-read assemblies or reads for resolving full-length TE insertions
- You only need to re-call insertions from an existing BAM/CRAM rather than generate an assembly-based truth set — a read-based caller like sTELLeR directly on the BAM is appropriate
- The analysis targets old/divergent insertions — the <15% divergence filter is intentionally biased toward recent intact insertions

## Inputs

- HPRC de novo long-read genome assemblies (FASTA) for HG002 and HG01071
- TE consensus sequences (FASTA), e.g. TEfastasequences.fa containing Alu and L1 consensus variants or TEsequences_SVA_HERV_ALU_L1.fasta
- Reference genome (e.g. HG38) for coordinates

## Outputs

- Per-sample truth callset tables of TE insertions (HG002: 2193 Alu, 492 L1, 14 HERV, 205 SVA; HG01071: 2262 Alu, 439 L1, 11 HERV, 202 SVA)
- RepeatMasker-annotated insertion alignments (SAM) with TE family labels and divergence values
- VCF-format TE insertion calls (INS:ME) when used for caller benchmarking

## How to apply

Run assembly-based insertion calling with SVIM-asm on the HPRC de novo assemblies of HG002 and HG01071 to extract candidate insertion sequences; minimap2, samtools, and python3 handle alignment and parsing. Align the extracted insertion sequences against TE consensus sequences and annotate with RepeatMasker to assign TE family labels. Enrich for recent intact insertions by filtering to calls with sequence divergence below 15% relative to the TE consensus, then apply TE-family-specific minimum consecutive length filters: 300 bp for SVA, 1500 bp for HERV, 200 bp for L1, and 100 bp for Alu. Compile the surviving calls into per-sample truth callset tables. When using this callset to benchmark a caller such as sTELLeR, note that sTELLeR itself is haplotype-aware, emits VCF (INS:ME), and can also run on genome assemblies as well as on any species.

## Related tools

- **SVIM-asm** (Assembly-based extraction of candidate TE insertion sequences from HPRC de novo assemblies)
- **RepeatMasker** (Aligns candidate insertions to TE consensus sequences and assigns TE family labels and divergence values)
- **minimap2** (Alignment of insertion sequences/reads during candidate extraction and downstream processing)
- **samtools** (Alignment file handling in the calling workflow)
- **sTELLeR** (Long-read TE caller benchmarked against the assembly-based truth callset; also capable of running on genome assemblies) — https://github.com/kristinebilgrav/sTELLeR
- **python3** (Glue scripting for filtering and compiling truth callset tables (with pysam))

## Examples

```
python steller/steller.py --ref HG38_chr22.fa --TE_fasta fasta/TEfastasequences.fa --bam testdata.bam --sr 4 --style pb -o testdata_res -mr 80
```

## Evaluation signals

- Per-family call counts match the reported truth set sizes (HG002: 2193 Alu, 492 L1, 14 HERV, 205 SVA; HG01071: 2262 Alu, 439 L1, 11 HERV, 202 SVA)
- All retained calls have sequence divergence <15% relative to the RepeatMasker TE consensus
- Retained calls meet family-specific minimum consecutive lengths (SVA ≥300 bp, HERV ≥1500 bp, L1 ≥200 bp, Alu ≥100 bp)
- When used to validate a caller, calls are output in VCF format with INS:ME naming and TE type/start/end in the INFO field
- On the sTELLeR test data a total of 7 variants should be identified, verifiable against testbamTRUTH.txt

## Limitations

- The >15% divergence filter enriches for recent, intact insertions and will exclude older, more divergent TE insertions from the truth set
- Family-specific length cutoffs differ by an order of magnitude (Alu 100 bp vs HERV 1500 bp), so counts are not directly comparable across families
- sTELLeR's --sr (default 3), --mq (default 20) and --maxreads (default 100) thresholds must be tuned to coverage and desired sensitivity; maxreads avoids calls in regions exceeding given coverage

## Evidence

- [other] SVIM-asm assembly extraction of candidates: [evidence span withheld — anchor status: not_found]
- [other] Divergence and length filters yield truth set sizes: [evidence span withheld — anchor status: not_found]
- [other] Truth-set workflow validation: [evidence span withheld — anchor status: not_found]
- [readme] Assembly-based calling is in scope for sTELLeR: "sTELLeR gives output in VCF, is haplotype-aware and can run on genome assemblies as well as on any species"
- [other] Family-specific filtering rationale: [evidence span withheld — anchor status: not_found]
