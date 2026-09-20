---
name: short-read-long-read-sv-concordance-analysis
description: Use when you have matched long-read (MinION/nanopore) and short-read
  (Illumina) SV call sets from the same genomes and need to determine which SVs are
  long-read-specific and characterize their size, GC, and repeat content.
when_to_use_negative:
- When there is no matched short-read call set from the same genome — the intersection
  defining 'nanopore-specific' requires the Illumina comparator data
- When the goal is de novo complex SV breakpoint detection or phasing of chromothripsis
  rather than genome-wide inherited SV concordance — use the NanoSV calling/phasing
  pipeline directly instead
- When reads have not been basecalled and mapped (BAM must be coordinate-sorted and
  indexed before NanoSV calling)
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: NanoSV
  role: Produces the nanopore SV VCF call sets that are classified as shared or long-read-specific
  repo: https://github.com/mroosmalen/nanosv
- name: Pindel
  role: Illumina short-read SV caller providing one of six comparator call sets for
    the intersection
- name: Manta
  role: Illumina short-read SV caller providing one of six comparator call sets for
    the intersection
- name: Delly
  role: Illumina short-read SV caller providing one of six comparator call sets for
    the intersection
- name: FREEC
  role: Illumina short-read variant caller providing one of six comparator call sets
    for the intersection
- name: Mobster
  role: Illumina short-read mobile-element caller providing one of six comparator
    call sets for the intersection
- name: GATK HaplotypeCaller
  role: Illumina variant caller providing one of six comparator call sets for the
    intersection
- name: sambamba
  role: Converts and sorts LAST/BWA MEM SAM output into sorted BAM input for NanoSV
    SV calling
  repo: https://github.com/biod/sambamba
- name: LAST
  role: Long-read mapper whose alignments give the most accurate results for SV calling
    with NanoSV
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-017-01343-4
    title: Mapping and phasing of structural variation in patient genomes using nanopore
      sequencing
  source_content_hash: sha256:7fe721def9f7eae30701ea64f7f0cc37f20348cf425699b35c0122f2f9b62a10
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mroosmalen/nanosv
  tools:
  - Delly
  - FREEC
  - GATK HaplotypeCaller
  - LAST
  - Manta
  - Mobster
  - NanoSV
  - Pindel
  - sambamba
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# short-read-long-read-sv-concordance-analysis

## Summary

A comparison workflow that intersects nanopore NanoSV call sets from MinION-sequenced patient genomes against short-read Illumina SV calls from six variant callers and 1KG variant sites, to quantify and characterize nanopore-specific structural variants missed by short reads.

## When to use

Use when you have matched long-read (MinION/nanopore) and short-read (Illumina) SV call sets from the same genomes and need to determine which SVs are long-read-specific and characterize their size, GC, and repeat content.

## When NOT to use

- When there is no matched short-read call set from the same genome — the intersection defining 'nanopore-specific' requires the Illumina comparator data
- When the goal is de novo complex SV breakpoint detection or phasing of chromothripsis rather than genome-wide inherited SV concordance — use the NanoSV calling/phasing pipeline directly instead
- When reads have not been basecalled and mapped (BAM must be coordinate-sorted and indexed before NanoSV calling)

## Inputs

- NanoSV VCF call sets from MinION nanopore reads (coordinate-sorted, indexed BAM from LAST/BWA MEM/minimap2/NGMLR mapping)
- SV call outputs from six Illumina callers (Pindel, Manta, Delly, FREEC, Mobster, GATK HaplotypeCaller) on matched Illumina data
- 1KG (1000 Genomes Project) variant sites for novelty filtering
- UCSC RepeatMasker and DFAM repeat annotations

## Outputs

- Classification of each nanopore SV as shared vs nanopore-specific
- nanopore_sv_comparison_summary.csv with per-class counts, size bins, repeat/LINE fractions, and mean GC content per category
- Fraction of nanopore-specific SVs (novel variants)

## How to apply

Load the NanoSV VCF call sets (e.g., Patient 1 and Patient 2 from MinION sequencing at 11–16x coverage) and the SV call outputs from six Illumina callers (Pindel, Manta, Delly, FREEC, Mobster, GATK HaplotypeCaller) run on the same genomes. Classify each nanopore SV as shared or nanopore-specific by intersecting against all six Illumina call sets using breakpoint-proximity/reciprocal-overlap criteria, then remove variants matching 1KG variant sites to define the novel set. Annotate repeat content of the SVs using UCSC RepeatMasker and DFAM annotations to flag retrotransposon/LINE insertions, then compute per-SV-class distributions, deletion size distributions (especially calls <200 bp), and GC content at nanopore-specific versus shared sites. Aggregate contrasts into a summary table with per-class counts, size bins, repeat/LINE fractions, and mean GC content per category.

## Related tools

- **NanoSV** (Produces the nanopore SV VCF call sets that are classified as shared or long-read-specific) — https://github.com/mroosmalen/nanosv
- **Pindel** (Illumina short-read SV caller providing one of six comparator call sets for the intersection)
- **Manta** (Illumina short-read SV caller providing one of six comparator call sets for the intersection)
- **Delly** (Illumina short-read SV caller providing one of six comparator call sets for the intersection)
- **FREEC** (Illumina short-read variant caller providing one of six comparator call sets for the intersection)
- **Mobster** (Illumina short-read mobile-element caller providing one of six comparator call sets for the intersection)
- **GATK HaplotypeCaller** (Illumina variant caller providing one of six comparator call sets for the intersection)
- **sambamba** (Converts and sorts LAST/BWA MEM SAM output into sorted BAM input for NanoSV SV calling) — https://github.com/biod/sambamba
- **LAST** (Long-read mapper whose alignments give the most accurate results for SV calling with NanoSV)

## Examples

```
lastal -Q1 -p my-params refdb reads.fastq | last-split | maf-convert -f reference.dict sam /dev/stdin | sambamba view -h -S --format=bam /dev/stdin | sambamba sort /dev/stdin -o reads.sorted.bam && NanoSV -s sambamba -o nanosv_calls.vcf reads.sorted.bam
```

## Evaluation signals

- The computed fraction of nanopore-specific SVs should fall near the reported ~14% novel-variant fraction (944 SVs), well below the 89% reported for previously reported PB data
- The SV class distribution of nanopore-specific calls should be deletion-dominated
- Nanopore-specific deletions should be enriched below 200 bp in the size distribution
- Nanopore-specific sites should show elevated LINE/retrotransposon content and higher mean GC content compared to shared SV sites
- SV calling precision of the underlying NanoSV pipeline should match the >95% precision benchmark, with stringent post-calling filtering at ~75% estimated sensitivity

## Limitations

- The ~14% novel-variant fraction is lowered by conservative SV calling and extremely stringent post-calling filtering that reduces estimated sensitivity to ~75%
- Medium coverage (11–16x) and intrinsic nanopore sequencing biases limit sensitivity of the underlying calls
- A large proportion of novel variants are retrotransposon insertions, which reside in complex repeat regions and segmental duplications — an area the authors flag as a particular challenge for building larger SV catalogs

## Evidence

- [abstract] Genome-wide surveillance of inherited SVs reveals novel variants missed in short-read data sets, a large proportion of which are retrotransposon insertions: "genome-wide surveillance of inherited SVs reveals novel variants, missed in short-read data sets, a large proportion of which are retrotransposon insertions"
- [discussion] Novel fraction is 14% vs 89% for PB data due to conservative filtering: "this percentage of novel variants is lower than for previously reported PB data (89%)"
- [discussion] SV pipeline precision and sensitivity benchmarks: [evidence span withheld — anchor status: not_found]
- [discussion] Six Illumina callers used for matched comparison: "could not be found in matching Illumina sequencing data (despite extensive variant calling using six different variant calling methods"
- [readme] LAST alignments are most accurate input for NanoSV SV calling: "We found that LAST alignments give the most accurate results for SV calling with NanoSV. However, mapping with LAST requires more compute resources."
