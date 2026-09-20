---
name: transposable-element-insertion-calling
description: Use when detecting non-reference TE insertions (Alu, L1, SVA, HERV) from
  BAM/CRAM alignments of PacBio or ONT long reads, especially when runtime and CPU
  efficiency matter (e.g. trio-scale or 60x genomes).
when_to_use_negative:
- The input is short-read data — sTELLeR is designed and tested for long-read (PacBio
  or ONT) data, so split-read evidence will not be available
- Only reference-present TE calls are needed — sTELLeR targets non-reference insertions,
  so a reference TE catalog tool would be more appropriate
- A reference FASTA or TE FASTA cannot be supplied — the caller requires both the
  alignment reference and a FASTA of insertion sequences to search for
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0199
tools:
- name: sTELLeR
  role: 'Primary TE insertion caller: consumes BAM/CRAM plus reference and TE FASTA,
    produces haplotype-aware VCF output'
  repo: https://github.com/kristinebilgrav/sTELLeR
- name: minimap2
  role: Dependency used to align long reads to the reference genome before TE calling
  repo: https://github.com/kristinebilgrav/sTELLeR
- name: samtools
  role: Dependency for BAM/CRAM handling during the calling workflow
  repo: https://github.com/kristinebilgrav/sTELLeR
- name: pysam
  role: Python dependency sTELLeR uses to read alignment files
  repo: https://github.com/kristinebilgrav/sTELLeR
- name: sTELLeR_supplementary
  role: Provides test BAM, masked chr22 reference, TE FASTAs (Alu/L1 and SVA/HERV
    versions), truth file, and benchmarking truth-set scripts
  repo: https://github.com/kristinebilgrav/sTELLeR_supplementary/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae686
    title: Detecting transposable elements in long-read genomes using sTELLeR
  source_content_hash: sha256:d9f58657e40d0e42257451e22b29872b1a0113b8846a429a88d2ae39724e14e3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kristinebilgrav/sTELLeR
  tools:
  - TLDR
  - minimap2
  - pysam
  - python3
  - sTELLeR
  - sTELLeR_supplementary
  - samtools
  - xTEA
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# transposable-element-insertion-calling

## Summary

Detect non-reference transposable element (TE) insertions in long-read genomes (PacBio or ONT) using sTELLeR, a fast, CPU-light, haplotype-aware caller that outputs VCF (INS:ME) records compatible with other variant callers.

## When to use

Use when detecting non-reference TE insertions (Alu, L1, SVA, HERV) from BAM/CRAM alignments of PacBio or ONT long reads, especially when runtime and CPU efficiency matter (e.g. trio-scale or 60x genomes).

## When NOT to use

- The input is short-read data — sTELLeR is designed and tested for long-read (PacBio or ONT) data, so split-read evidence will not be available
- Only reference-present TE calls are needed — sTELLeR targets non-reference insertions, so a reference TE catalog tool would be more appropriate
- A reference FASTA or TE FASTA cannot be supplied — the caller requires both the alignment reference and a FASTA of insertion sequences to search for

## Inputs

- BAM or CRAM file of long reads (PacBio CLR/HiFi or ONT) aligned to the reference genome
- Reference genome FASTA used for alignment (e.g. HG38_chr22.fa)
- FASTA file of insertion/TE sequences to detect (e.g. TEfastasequences.fa with Alu and L1 subfamilies, or TEsequences_SVA_HERV_ALU_L1.fasta including SVA and HERV)

## Outputs

- VCF file (<output prefix>_repeats.vcf) with variants named INS:ME, INFO fields with TE type and start/end positions, and FORMAT fields TE:TS:HT:GT
- Optional intermediate files when --keep_intermediates is set: per-chromosome candidate-insertion FASTA (ID_chr_reads.fasta) and candidate-to-TE SAM alignments (ID_chr_repeats.sam)

## How to apply

Align long reads to GRCh38 with minimap2 and run steller.py with a BAM/CRAM, the alignment reference (--ref), and a FASTA of TE insertion sequences to detect (--TE_fasta), setting --style to 'pb' for PacBio or 'ont' for ONT reads. Tune the minimum supporting split reads/insertions (--sr, default 3; examples use 4), the mapping quality threshold to the TE sequence (--mq, default 20), and the maximum supporting reads to avoid calls in regions exceeding a coverage cap (--maxreads, default 100; examples use 80). Use --keep_intermediates to retain per-chromosome candidate-insertion FASTA files (ID_chr_reads.fasta) and candidate-to-TE alignments (ID_chr_repeats.sam); otherwise these are deleted. If the input BAM is phased, the output VCF reports the haplotype (HT) carrying the insertion and genotype (1|0, 0|1, or 1|1); unphased input yields haplotag 0 with genotypes 1/0 or 1/1. Verify the installation by running the supplied test BAM and confirming 7 variants against testbamTRUTH.txt.

## Related tools

- **sTELLeR** (Primary TE insertion caller: consumes BAM/CRAM plus reference and TE FASTA, produces haplotype-aware VCF output) — https://github.com/kristinebilgrav/sTELLeR
- **minimap2** (Dependency used to align long reads to the reference genome before TE calling) — https://github.com/kristinebilgrav/sTELLeR
- **samtools** (Dependency for BAM/CRAM handling during the calling workflow) — https://github.com/kristinebilgrav/sTELLeR
- **pysam** (Python dependency sTELLeR uses to read alignment files) — https://github.com/kristinebilgrav/sTELLeR
- **sTELLeR_supplementary** (Provides test BAM, masked chr22 reference, TE FASTAs (Alu/L1 and SVA/HERV versions), truth file, and benchmarking truth-set scripts) — https://github.com/kristinebilgrav/sTELLeR_supplementary/

## Examples

```
python steller/steller.py --ref HG38_chr22.fa --TE_fasta fasta/TEfastasequences.fa --bam testdata.bam --sr 4 --style pb -o testdata_res -mr 80
```

## Evaluation signals

- Running the provided test BAM should identify a total of 7 variants matching testbamTRUTH.txt
- Output VCF variants are named INS:ME with TE type, transposon start (TS), haplotag (HT), and genotype (GT) populated; HT is 0 when input is unphased
- Calls must meet the configured thresholds: minimum supporting split reads (--sr, default 3), mapping quality to TE sequence (--mq, default 20), and at most --maxreads supporting reads (default 100)
- If phased input is supplied, genotypes appear as 1|0, 0|1, or 1|1; unphased input yields 1/0 or 1/1
- Intermediate files ID_chr_reads.fasta and ID_chr_repeats.sam should be absent unless --keep_intermediates was set

## Limitations

- The caller was designed and tested for TE insertions in long-read data; detection of other insertion types is supported but was not the design target
- Reference base correctness matters: the test reference is a masked chr22 with removed masked parts (zip file) that must be extracted (tar -xvzf) for correct reference bases
- Supporting reads threshold should be adjusted to sample coverage: default --sr 3 balances sensitivity, and --maxreads caps calls in regions exceeding a coverage limit
- If input is not phased, the output VCF cannot report which haplotype carries the TE (haplotype reported as 0)

## Evidence

- [intro] Designed for non-reference TE detection on long reads: "a tool for detection of non-reference transposable element using long-read data such as PacBio or ONT"
- [readme] Required inputs are BAM/CRAM, reference, and TE FASTA: "sTELLeR needs a bam or cram file, the reference used to align it, and a fasta file with insertions sequences to look for (--TE_fasta)"
- [readme] Parameter defaults and roles: "-r , --sr         Minimum number of supporting split reads/insertions to call a variant (default 3)"
- [readme] Concrete test invocation and validation: "python steller/steller.py --ref HG38_chr22.fa --TE_fasta fasta/TEfastasequences.fa --bam testdata.bam --sr 4 --style pb -o testdata_res -mr 80"
- [methods] Test data should yield 7 variants: "a total of 7 variants should be identified"
- [abstract] Speed and CPU efficiency relative to competing callers: "The caller is 5-48× as fast and uses <2% of the CPU hours compared to competitive callers."
