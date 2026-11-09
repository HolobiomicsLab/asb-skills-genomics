---
name: snp-based-read-allele-assignment
description: Use when long-read (nanoEM/nanopore) alignments span a heterozygous SNP
  within or near a CpG-dense imprinted locus and per-allele methylation frequencies
  are needed to test parent-of-origin methylation patterns.
when_to_use_negative:
- The sample has no heterozygous SNP within or near the target imprinted locus, so
  reads cannot be phased to an allele
- The goal is bulk CpG methylation frequency across the genome rather than allele-resolved
  methylation (call_methylation.py output alone suffices)
- You need to distinguish 5-methylcytosine from 5-hydroxymethylcytosine — nanoEM base
  conversion cannot separate them on the same molecule
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_3295
tools:
- name: pysam
  role: Parse output_CT/output_GA sorted BAMs and assign each long read spanning the
    heterozygous C/G SNP to the C or G allele
  repo: https://github.com/yos-sk/nanoEM
- name: sambamba
  role: mpileup extraction of read-level observations at locus CpG sites from CT and
    GA sorted BAMs (-L cpg_sites.bed, -t 8, --samtools -f ref)
  repo: https://github.com/yos-sk/nanoEM
- name: samtools
  role: Sort/index the CT and GA BAMs feeding the allele assignment and pileup steps
  repo: https://github.com/yos-sk/nanoEM
- name: Python3
  role: Host language for the allele-assignment and per-allele methylation aggregation
    script
  repo: https://github.com/yos-sk/nanoEM
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkab397
    title: Long-read whole-genome methylation patterning using enzymatic base conversion
      and nanopore sequencing
  source_content_hash: sha256:dd3a5b644dce44be20b66d7b847665fffa7eebc79b12bd5904ed712de2df76b5
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/yos-sk/nanoEM
  repo_url: https://github.com/yos-sk/nanoEM
  tools:
  - Python3
  - pysam
  - sambamba
  - samtools
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# snp-based-read-allele-assignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assigns individual long reads to parental alleles using heterozygous SNPs (e.g., a heterozygous C/G SNP at MEST in MB231) so that CpG methylation frequencies from nanoEM CT/GA pileups can be computed separately per allele, resolving allele-specific methylation at imprinted loci such as GRB10 and MEST that short-read methylation analysis cannot resolve.

## When to use

Use when long-read (nanoEM/nanopore) alignments span a heterozygous SNP within or near a CpG-dense imprinted locus and per-allele methylation frequencies are needed to test parent-of-origin methylation patterns.

## When NOT to use

- The sample has no heterozygous SNP within or near the target imprinted locus, so reads cannot be phased to an allele
- The goal is bulk CpG methylation frequency across the genome rather than allele-resolved methylation (call_methylation.py output alone suffices)
- You need to distinguish 5-methylcytosine from 5-hydroxymethylcytosine — nanoEM base conversion cannot separate them on the same molecule

## Inputs

- frequency_methylation.tsv (per-CpG methylation frequencies from call_methylation.py)
- output_CT.sorted.bam and output_GA.sorted.bam (best-alignment CT/GA converted reads from best_align.py)
- cpg_sites.bed (CpG site intervals for mpileup -L)
- Converted reference genome output.fa and per-read FASTQ with heterozygous SNP genotype information

## Outputs

- allele_specific_methylation_GRB10_MEST.tsv with one row per locus/CpG/allele and per-allele methylation frequencies

## How to apply

Load per-CpG methylation frequencies from frequency_methylation.tsv (produced by call_methylation.py from the CT and GA pileups) and subset rows overlapping GRB10 and MEST. If read-level resolution is required, extract pileups at the locus CpG sites with sambamba mpileup -L cpg_sites.bed on output_CT.sorted.bam and output_GA.sorted.bam (8 threads, --samtools -f ref). Using pysam, assign each long read spanning the MEST heterozygous C/G SNP to the C or G allele, and group GRB10 reads by their allelic CpG methylation pattern. Aggregate converted-read observations per assigned allele to compute per-allele methylation frequency at each CpG, and write allele_specific_methylation_GRB10_MEST.tsv with one row per locus/CpG/allele.

## Related tools

- **pysam** (Parse output_CT/output_GA sorted BAMs and assign each long read spanning the heterozygous C/G SNP to the C or G allele) — https://github.com/yos-sk/nanoEM
- **sambamba** (mpileup extraction of read-level observations at locus CpG sites from CT and GA sorted BAMs (-L cpg_sites.bed, -t 8, --samtools -f ref)) — https://github.com/yos-sk/nanoEM
- **samtools** (Sort/index the CT and GA BAMs feeding the allele assignment and pileup steps) — https://github.com/yos-sk/nanoEM
- **Python3** (Host language for the allele-assignment and per-allele methylation aggregation script) — https://github.com/yos-sk/nanoEM

## Examples

```
sambamba mpileup output_CT.sorted.bam -L cpg_sites.bed -o pileup_CT.tsv -t 8 --samtools -f ref; sambamba mpileup output_GA.sorted.bam -L cpg_sites.bed -o pileup_GA.tsv -t 8 --samtools -f ref
```

## Evaluation signals

- The output table reproduces the Figure 4C-D pattern: the MEST allele carrying C at the heterozygous SNP is methylated and the G allele is unmethylated, with clearly separated per-allele methylation frequencies
- Every read assigned to an allele actually spans the heterozygous SNP in both CT and GA converted alignments (consistent best alignments from best_align.py)
- allele_specific_methylation_GRB10_MEST.tsv has exactly one row per locus/CpG/allele combination and frequencies in [0, 1]
- Per-allele frequencies are consistent with the bulk frequency_methylation.tsv values when weighted by read counts per allele

## Limitations

- nanoEM cannot distinguish 5-methylcytosine from 5-hydroxymethylcytosine on the same DNA molecule, so allele-specific mC vs hmC assignment is out of scope
- Allele assignment depends on a heterozygous SNP being present and covered; homozygous loci cannot be phased
- Allele-specific methylation inference is anchored to SNP genotype, which must be validated (e.g., MEST is a paternally expressed gene, so only the C allele was expected to be methylated)

## Evidence

- [other] Assign each long read spanning the MEST heterozygous SNP to the C or G allele, and group reads at GRB10 by their allelic CpG methylation pattern.: [evidence span withheld — anchor status: not_found]
- [other] Allele-specific methylation finding: [evidence span withheld — anchor status: not_found]
