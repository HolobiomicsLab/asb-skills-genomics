---
name: imprinted-locus-methylation-analysis
description: Use when nanoEM (converted long-read) alignments cover an imprinted locus
  with a heterozygous SNP, and per-allele CpG methylation frequencies must be computed
  to determine which parental allele is methylated.
when_to_use_negative:
- The locus contains no heterozygous SNP to anchor reads to a parental allele, so
  reads cannot be assigned to C or G alleles.
- The biological question is distinguishing 5-methylcytosine from 5-hydroxymethylcytosine
  on the same molecule — nanoEM cannot separate these.
- Input data are short-read bisulfite/EMseq alignments rather than nanoEM CT/GA converted
  long-read BAMs, since short reads cannot span SNP-to-CpG distances needed for allele
  phasing.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: pysam
  role: Assign each long read spanning the MEST heterozygous C/G SNP to an allele
    and group reads by allelic CpG methylation pattern
- name: sambamba
  role: Extract per-read pileups at locus CpG sites from the CT and GA sorted BAMs
- name: samtools
  role: Sort and index BAM outputs consumed and produced during locus pileup extraction
- name: minimap2
  role: Align C-to-T and G-to-A converted reads to the converted reference to produce
    the underlying alignments
- name: call_methylation.py
  role: Merge CT and GA pileups into per-CpG methylation frequencies (frequency_methylation.tsv)
    used as the starting table
  repo: https://github.com/yos-sk/nanoEM
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkab397
    title: Long-read whole-genome methylation patterning using enzymatic base conversion
      and nanopore sequencing
  source_content_hash: sha256:30135e0f800398076a63e5c651650386d59d86eeb72e2ff2992824a61e77c10e
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
  - call_methylation.py
  - minimap2
  - pysam
  - sambamba
  - samtools
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# imprinted-locus-methylation-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Resolves allele-specific methylation at imprinted loci (e.g., GRB10 and MEST) by combining nanoEM C-to-T/G-to-A converted-read pileups with read-level assignment to alleles via a heterozygous SNP, which short-read methylation analysis cannot resolve.

## When to use

Use when nanoEM (converted long-read) alignments cover an imprinted locus with a heterozygous SNP, and per-allele CpG methylation frequencies must be computed to determine which parental allele is methylated.

## When NOT to use

- The locus contains no heterozygous SNP to anchor reads to a parental allele, so reads cannot be assigned to C or G alleles.
- The biological question is distinguishing 5-methylcytosine from 5-hydroxymethylcytosine on the same molecule — nanoEM cannot separate these.
- Input data are short-read bisulfite/EMseq alignments rather than nanoEM CT/GA converted long-read BAMs, since short reads cannot span SNP-to-CpG distances needed for allele phasing.

## Inputs

- output_CT.sorted.bam and output_GA.sorted.bam (best-align selected nanoEM CT/GA alignments from the nanoEM pipeline)
- frequency_methylation.tsv (per-CpG methylation frequencies from call_methylation.py)
- cpg_sites.bed (locus CpG site coordinates)
- Reference genome FASTA (used via --samtools -f ref)

## Outputs

- allele_specific_methylation_GRB10_MEST.tsv (one row per locus/CpG/allele with per-allele methylation frequency)
- pileup_CT.tsv and pileup_GA.tsv (sambamba mpileup output restricted to locus CpG sites)

## How to apply

Load per-CpG methylation frequencies from frequency_methylation.tsv (produced by call_methylation.py from the CT and GA pileups) and subset rows overlapping GRB10 and MEST. For read-level resolution, extract pileups at the locus CpG sites with sambamba mpileup -L cpg_sites.bed from output_CT.sorted.bam and output_GA.sorted.bam (8 threads, --samtools -f ref). Using pysam, assign each long read spanning the MEST heterozygous C/G SNP to the C or G allele, and aggregate converted-read observations per assigned allele at each CpG. The rationale is that long reads span the SNP and many CpGs simultaneously, so allelic origin of each read is unambiguous where short reads cannot link the SNP to distant CpGs. Write the per-allele methylation table to allele_specific_methylation_GRB10_MEST.tsv with one row per locus/CpG/allele, and validate that one MEST allele (C at the SNP) is methylated while the other (G) is unmethylated, reproducing the Figure 4C-D pattern.

## Related tools

- **pysam** (Assign each long read spanning the MEST heterozygous C/G SNP to an allele and group reads by allelic CpG methylation pattern)
- **sambamba** (Extract per-read pileups at locus CpG sites from the CT and GA sorted BAMs)
- **samtools** (Sort and index BAM outputs consumed and produced during locus pileup extraction)
- **minimap2** (Align C-to-T and G-to-A converted reads to the converted reference to produce the underlying alignments)
- **call_methylation.py** (Merge CT and GA pileups into per-CpG methylation frequencies (frequency_methylation.tsv) used as the starting table) — https://github.com/yos-sk/nanoEM

## Examples

```
sambamba mpileup output_CT.sorted.bam -L cpg_sites.bed -o pileup_CT.tsv -t 8 --samtools -f ref
```

## Evaluation signals

- The output table reproduces the Figure 4C-D pattern: the MEST allele carrying C at the heterozygous SNP is methylated while the G allele is unmethylated, i.e., the two alleles show clearly separated methylation frequencies.
- Every locus/CpG/allele combination appears as exactly one row in allele_specific_methylation_GRB10_MEST.tsv.
- Per-allele methylation frequencies are consistent with frequency_methylation.tsv when the two allele-specific CpG tracks are combined.
- Reads at GRB10 cluster into two allelic CpG methylation patterns consistent with the MEST-validated C/G SNP assignment.
- MEST, a known paternally expressed gene, shows methylation on the C allele, matching expected imprinting directionality.

## Limitations

- nanoEM cannot distinguish between 5-methylcytosine and 5-hydroxymethylcytosine on the same DNA molecule, unlike direct nanopore WGS calling.
- Allele assignment depends on a heterozygous SNP being detectable within the imprinted locus; loci without informative SNPs cannot be phased.
- No version numbers are stated for any of the software used (pysam, sambamba, samtools, minimap2), so exact reproducibility of the environment is uncertain.

## Evidence

- [results] nanoEM revealed a clear allele-specific methylation pattern at the imprinted GRB10 and MEST loci: [evidence span withheld — anchor status: not_found]
