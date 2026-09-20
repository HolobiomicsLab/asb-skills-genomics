---
name: te-insertion-candidate-clustering
description: Use when you have a BAM/CRAM of PacBio or ONT long reads aligned to a
  reference plus a fasta of insertion sequences to detect, and you need to call non-reference
  TE insertions as VCF INS:ME variants.
when_to_use_negative:
- Input is short-read data — sTELLeR is designed and tested for detection of transposable
  element insertions in long-read data (PacBio or ONT)
- Reference or TE fasta is absent — sTELLeR needs a bam or cram file, the reference
  used to align it, and a fasta file with insertion sequences to look for
- You only need reference-TE annotation rather than non-reference insertion calling
  — this skill targets non-reference TE detection and variant-style output
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: sTELLeR
  role: End-to-end caller implementing candidate extraction, clustering, TE alignment,
    consensus refinement, filtering, and VCF writing
  repo: https://github.com/kristinebilgrav/sTELLeR
- name: pysam
  role: Parse the BAM to extract split reads and insertion sequences spanning candidate
    TE insertions
- name: minimap2
  role: Align clustered candidate insertion sequences to the TE fasta (ID_chr_repeats.sam
    intermediates)
- name: samtools
  role: Dependency supporting BAM handling during candidate extraction and filtering
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btae686
    title: Detecting transposable elements in long-read genomes using sTELLeR
  source_content_hash: sha256:04c3e5ae99aa1b6c09a949590cebb7642a9b9df3b6e5e0737a869ca3d84278ea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kristinebilgrav/sTELLeR
  tools:
  - minimap2
  - pysam
  - python3
  - sTELLeR
  - samtools
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# te-insertion-candidate-clustering

## Summary

Cluster candidate TE insertion signatures from long-read data and align candidate insertion sequences to a TE fasta (e.g. fasta/TEfastasequences.fa) with minimap2, then refine consensus positions and filter calls by support depth, mapping quality, and coverage before writing haplotype-aware VCF output.

## When to use

Use when you have a BAM/CRAM of PacBio or ONT long reads aligned to a reference plus a fasta of insertion sequences to detect, and you need to call non-reference TE insertions as VCF INS:ME variants.

## When NOT to use

- Input is short-read data — sTELLeR is designed and tested for detection of transposable element insertions in long-read data (PacBio or ONT)
- Reference or TE fasta is absent — sTELLeR needs a bam or cram file, the reference used to align it, and a fasta file with insertion sequences to look for
- You only need reference-TE annotation rather than non-reference insertion calling — this skill targets non-reference TE detection and variant-style output

## Inputs

- BAM or CRAM file of long reads (PacBio pb/clr/hifi or ont) aligned to a reference genome
- Reference genome fasta used for the alignment (e.g. HG38_chr22.fa)
- Fasta file with sequences of elements to be detected (--TE_fasta, e.g. fasta/TEfastasequences.fa)
- Optional phased BAM input for haplotype-aware output

## Outputs

- VCF file (<output prefix>_repeats.vcf) with variants named INS:ME and format fields TE:TS:HT:GT
- Optional intermediate per-chromosome fasta files (ID_chr_reads.fasta) with candidate insertions
- Optional intermediate SAM files (ID_chr_repeats.sam) with candidate insertions aligned to TEs

## How to apply

Parse the input BAM with pysam to extract split reads and insertion sequences spanning candidate TE insertions, keeping per-chromosome candidate fasta intermediates (ID_chr_reads.fasta). Cluster candidate insertion signatures and align the candidate insertion sequences to the TE fasta with minimap2, writing SAM intermediates (ID_chr_repeats.sam). Refine aligned sequences to a consensus nucleotide position and keep only clusters where at least --sr (default 3) supporting split reads/insertions match a TE, with mapping quality >= --mq (default 20) and at most --maxreads/--mr (default 100) supporting reads to avoid calls in regions exceeding given coverage. Write final calls to a VCF with variant name INS:ME and format fields TE:TS:HT:GT (TE type, transposon start, haplotag, genotype).

## Related tools

- **sTELLeR** (End-to-end caller implementing candidate extraction, clustering, TE alignment, consensus refinement, filtering, and VCF writing) — https://github.com/kristinebilgrav/sTELLeR
- **pysam** (Parse the BAM to extract split reads and insertion sequences spanning candidate TE insertions)
- **minimap2** (Align clustered candidate insertion sequences to the TE fasta (ID_chr_repeats.sam intermediates))
- **samtools** (Dependency supporting BAM handling during candidate extraction and filtering)

## Examples

```
python steller/steller.py --ref HG38_chr22.fa --TE_fasta fasta/TEfastasequences.fa --bam testdata.bam --sr 4 --style pb -o testdata_res -mr 80
```

## Evaluation signals

- Run on the sTELLeR_supplementary test data (HG38_chr22.fa, fasta/TEfastasequences.fa, testdata.bam, --style pb): a total of 7 variants should be identified
- Compare the produced testdata_res_repeats.vcf to the provided testbamTRUTH.txt; correct results match the truth file's TE positions
- Each VCF record is named INS:ME and its format section carries TE:TS:HT:GT fields (TE type, transposon start, haplotag, genotype)
- Filter invariants: every call has >= --sr supporting split reads/insertions (default 3), TE mapping quality >= --mq (default 20), and <= --mr/--maxreads supporting reads (default 100)
- Phased input yields haplotype-aware GT values (1|0, 0|1 or 1|1); unphased input yields HT 0 with GT 1/0 or 1/1

## Limitations

- Supporting reads (--sr) depends on the coverage of the sample and the desired sensitivity (default 3), so the threshold must be tuned per dataset
- Maximum number of reads (--maxreads, default 100) must be adjusted to avoid calls in regions exceeding given coverage
- Supplementary test data reference is a masked chr22 with removed masked parts in a zip file that must be unpacked (tar -xvzf) for correct reference bases
- Intermediate files (ID_chr_reads.fasta, ID_chr_repeats.sam) are removed unless --keep_intermediates is set, which can hinder debugging

## Evidence

- [other] After alignment, sTELLeR refines the aligned sequences to a consensus nucleotide position and filters to keep only clusters where a minimum number of reads (user-defined, default 3) match a TE.: [evidence span withheld — anchor status: not_found]
- [readme] Filter defaults and thresholds from CLI documentation.: "-r , --sr         Minimum number of supporting split reads/insertions to call a variant (default 3)"
- [readme] Maxreads rationale.: "Maximum number of reads (--maxreads) can be adjusted to avoid calls in regions exceeding given coverage (default=100)."
- [methods] Test-data validation expectation.: "a total of 7 variants should be identified"
- [readme] VCF naming and format fields.: "The variant is named INS:ME and in the info section the TE type along with start and end positions are given."
