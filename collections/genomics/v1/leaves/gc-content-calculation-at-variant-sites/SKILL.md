---
name: gc-content-calculation-at-variant-sites
description: Use when comparing SV call sets (e.g., NanoSV nanopore calls vs six Illumina
  caller outputs) and you need sequence-composition features such as GC content and
  repeat/LINE fractions to characterize novel variant sites.
when_to_use_negative:
- When the input is already an annotated feature/composition table with GC content
  per variant — recomputation is redundant.
- When the goal is SV calling or genotyping itself (use NanoSV on long-read BAMs)
  rather than post-hoc characterization of existing call sets.
edam_operation: http://edamontology.org/operation_0236
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: NanoSV
  role: Source of the nanopore SV VCF call sets whose sites are characterized for
    GC content and novelty
  repo: https://github.com/mroosmalen/nanosv
- name: Illumina variant callers (Pindel, Manta, Delly, FREEC, Mobster, GATK HaplotypeCaller)
  role: Short-read call sets used to classify nanopore SVs as shared vs nanopore-specific
- name: UCSC RepeatMasker / DFAM
  role: Repeat annotations used alongside GC content to flag retrotransposon/LINE
    insertions at SV sites
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-017-01343-4
    title: Mapping and phasing of structural variation in patient genomes using nanopore
      sequencing
  source_content_hash: sha256:452d12e839733343624b5852ec3a90860e877b07a6ef64f7c049caf653918ba1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mroosmalen/nanosv
  tools:
  - Delly
  - FREEC
  - GATK HaplotypeCaller
  - Illumina variant callers (Pindel, Manta, Delly, FREEC, Mobster, GATK HaplotypeCaller)
  - Manta
  - Mobster
  - NanoSV
  - Pindel
  - UCSC RepeatMasker / DFAM
  edam_operation: http://edamontology.org/operation_0236
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# gc-content-calculation-at-variant-sites

## Summary

Computes GC content at SV breakpoint/local-sequence windows to contrast nanopore-specific versus shared SV sites, as done in the Cretu Stancu et al. (2017) nanopore vs Illumina SV comparison where nanopore-specific sites showed higher GC content.

## When to use

Use when comparing SV call sets (e.g., NanoSV nanopore calls vs six Illumina caller outputs) and you need sequence-composition features such as GC content and repeat/LINE fractions to characterize novel variant sites.

## When NOT to use

- When the input is already an annotated feature/composition table with GC content per variant — recomputation is redundant.
- When the goal is SV calling or genotyping itself (use NanoSV on long-read BAMs) rather than post-hoc characterization of existing call sets.

## Inputs

- NanoSV VCF call sets for Patient 1 and Patient 2 (nanopore, MinION 11-16x)
- SV/VCF call outputs from six Illumina callers (Pindel, Manta, Delly, FREEC, Mobster, GATK HaplotypeCaller) on the same genomes
- 1KG (1000 Genomes Project) variant sites for novelty filtering
- UCSC RepeatMasker and DFAM repeat annotations
- Reference genome FASTA (e.g., hg19) for extracting GC-content windows

## Outputs

- nanopore_sv_comparison_summary.csv with per-class counts, size bins, repeat/LINE fractions, and mean GC content per category
- Novel SV set (14%, 944 SVs) flagged as nanopore-specific (absent from Illumina calls and 1KG)

## How to apply

First classify each SV from the NanoSV Patient 1 and Patient 2 call sets as shared or nanopore-specific by intersecting against the six Illumina call sets (Pindel, Manta, Delly, FREEC, Mobster, GATK HaplotypeCaller) with breakpoint-proximity/reciprocal-overlap criteria, and remove variants matching 1KG variant sites. For each SV site, extract the local reference sequence window around the breakpoint and compute GC fraction, and annotate repeat content with UCSC RepeatMasker and DFAM to flag retrotransposon/LINE insertions. Aggregate mean GC content per category (nanopore-specific vs shared) alongside SV class counts, size bins (with focus on deletions <200 bp), and repeat fractions into nanopore_sv_comparison_summary.csv. The rationale is that sequence composition biases (GC, repeats) explain why certain SVs, especially small LINE-rich deletions, are invisible to short-read Illumina calling but recoverable from MinION long reads.

## Related tools

- **NanoSV** (Source of the nanopore SV VCF call sets whose sites are characterized for GC content and novelty) — https://github.com/mroosmalen/nanosv
- **Illumina variant callers (Pindel, Manta, Delly, FREEC, Mobster, GATK HaplotypeCaller)** (Short-read call sets used to classify nanopore SVs as shared vs nanopore-specific)
- **UCSC RepeatMasker / DFAM** (Repeat annotations used alongside GC content to flag retrotransposon/LINE insertions at SV sites)

## Examples

```
python scripts/nanopore_sv_comparison.py --nanopore patient1_nanosv.vcf patient2_nanosv.vcf --illumina pindel.vcf manta.vcf delly.vcf freec.vcf mobster.vcf haplotypecaller.vcf --repeats repeatmasker.bed --out nanopore_sv_comparison_summary.csv
```

## Evaluation signals

- The computed fraction of nanopore-specific SVs falls near the reported ~14% novel-variant fraction (944 SVs), well below the 89% reported for PB data.
- SV class distribution is deletion-dominated.
- Nanopore-specific deletions are enriched below 200 bp in size-bin analysis.
- Mean GC content at nanopore-specific sites is higher than at shared SV sites.
- LINE/retrotransposon fraction is elevated among nanopore-specific SVs, consistent with the paper's finding that a large proportion of novel variants are retrotransposon insertions.

## Limitations

- Conservative SV calling and extremely stringent post-calling filtering (sensitivity ~75%) lower the fraction of novel variants, biasing GC/repeat contrast estimates.
- Medium coverage (11-16x) and intrinsic nanopore sequencing biases limit SV detection, so nanopore-specific sets are incomplete.
- Creating larger SV catalogs in complex repeat regions and segmental duplications remains an unresolved challenge.

## Evidence

- [discussion] Novel SV GC contrast: "this percentage of novel variants is lower than for previously reported PB data (89%)"
- [abstract] Novel variants missed in short reads are retrotransposon-rich: "genome-wide surveillance of inherited SVs reveals novel variants, missed in short-read data sets, a large proportion of which are retrotransposon insertions"
- [discussion] Comparison against Illumina data with six callers and 1KG: "could not be found in matching Illumina sequencing data (despite extensive variant calling using six different variant calling methods"
- [discussion] Filtering stringency caveat: "we employ extremely stringent filtering that results in a good estimated sensitivity (~75%)"
- [discussion] Repeat-region challenge: "Creating larger catalogs of SVs, in complex repeat regions and segmental duplications, is a particular challenge in the coming years"
