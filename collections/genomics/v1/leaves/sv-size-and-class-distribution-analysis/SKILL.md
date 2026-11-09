---
name: sv-size-and-class-distribution-analysis
description: Use when you have long-read (e.g., nanopore NanoSV VCF) SV call sets
  plus matched short-read caller outputs and need to classify which calls are novel,
  then profile their size bins, GC content, and retrotransposon (LINE) enrichment.
when_to_use_negative:
- Input consists only of short-read Illumina SV calls with no long-read call set to
  compare against — there is no nanopore-specific fraction to define.
- The goal is de novo complex rearrangement resolution or phasing (e.g., chromothripsis
  breakpoint phasing) rather than genome-wide novel-SV characterization.
- Coverage is far below the 11–16× MinION depth used here; size/GC bias contrasts
  will not be comparable to the published distributions.
edam_operation: http://edamontology.org/operation_3226
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_0622
tools:
- name: NanoSV
  role: Produces the long-read SV call sets (split- and gapped-aligned read clustering)
    whose class/size/GC/repeat distributions are analyzed
  repo: https://github.com/mroosmalen/nanosv
- name: sambamba
  role: Converts and sorts LAST-generated SAM alignments into coordinate-sorted BAMs
    that are the input to NanoSV SV calling
  repo: https://github.com/biod/sambamba
- name: Pindel / Manta / Delly / FREEC / Mobster / GATK HaplotypeCaller
  role: Six Illumina short-read callers whose call sets define the 'shared' SV baseline
    against which nanopore-specific calls are identified
- name: UCSC RepeatMasker / DFAM
  role: Annotate repeat content of SVs to flag retrotransposon/LINE insertions among
    nanopore-specific calls
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-017-01343-4
    title: Mapping and phasing of structural variation in patient genomes using nanopore
      sequencing
  source_content_hash: sha256:abc6c516e35f99a6b366e1bcad190d3f925d435dcbedbc9a2888cadb4739a8a4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mroosmalen/nanosv
  tools:
  - Delly
  - FREEC
  - GATK HaplotypeCaller
  - Manta
  - Mobster
  - NanoSV
  - Pindel
  - Pindel / Manta / Delly / FREEC / Mobster / GATK HaplotypeCaller
  - UCSC RepeatMasker / DFAM
  - sambamba
  edam_operation: http://edamontology.org/operation_3226
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# sv-size-and-class-distribution-analysis

## Summary

Quantifies the fraction of nanopore-specific SVs (not seen in short-read Illumina calls or 1KG sites) and characterizes them by SV class, size, GC content, and repeat/LINE composition, as done with NanoSV calls from MinION-sequenced patient genomes at 11–16× coverage.

## When to use

Use when you have long-read (e.g., nanopore NanoSV VCF) SV call sets plus matched short-read caller outputs and need to classify which calls are novel, then profile their size bins, GC content, and retrotransposon (LINE) enrichment.

## When NOT to use

- Input consists only of short-read Illumina SV calls with no long-read call set to compare against — there is no nanopore-specific fraction to define.
- The goal is de novo complex rearrangement resolution or phasing (e.g., chromothripsis breakpoint phasing) rather than genome-wide novel-SV characterization.
- Coverage is far below the 11–16× MinION depth used here; size/GC bias contrasts will not be comparable to the published distributions.

## Inputs

- NanoSV VCF call sets for the nanopore (MinION) patient genomes (Patient 1 and Patient 2)
- SV call outputs from six Illumina short-read callers: Pindel, Manta, Delly, FREEC, Mobster, GATK HaplotypeCaller
- 1KG (1000 Genomes Project) variant sites for filtering known variants
- UCSC RepeatMasker and DFAM repeat annotations
- Reference genome sequence and LAST/Sambamba-processed nanopore BAM alignments

## Outputs

- nanopore_sv_comparison_summary.csv with per-SV-class counts, size bins, repeat/LINE fractions, and mean GC content per category
- Nanopore-specific (novel) SV set with class, size, GC, and repeat annotations

## How to apply

Load the NanoSV VCF call sets (here, Patient 1 and Patient 2 MinION data at 11–16× coverage) and the SV outputs of six Illumina callers (Pindel, Manta, Delly, FREEC, Mobster, GATK HaplotypeCaller) on the same genomes. Classify each nanopore SV as shared vs nanopore-specific by intersecting against all six Illumina call sets using breakpoint-proximity/reciprocal-overlap criteria, and remove variants matching 1KG variant sites to define the novel set. Annotate repeat content with UCSC RepeatMasker and DFAM to flag retrotransposon/LINE insertions, then compute SV class distribution, size distribution of nanopore-specific deletions (focusing on calls <200 bp), and GC content at nanopore-specific vs shared sites. Aggregate per-class counts, size bins, repeat/LINE fractions, and mean GC per category into a summary table (nanopore_sv_comparison_summary.csv). Note the article's caveat that conservative NanoSV calling and extremely stringent post-calling filtering deliberately lower the novel-variant fraction, so the baseline expectation is ~14% novel, far below the 89% reported for PB data.

## Related tools

- **NanoSV** (Produces the long-read SV call sets (split- and gapped-aligned read clustering) whose class/size/GC/repeat distributions are analyzed) — https://github.com/mroosmalen/nanosv
- **sambamba** (Converts and sorts LAST-generated SAM alignments into coordinate-sorted BAMs that are the input to NanoSV SV calling) — https://github.com/biod/sambamba
- **Pindel / Manta / Delly / FREEC / Mobster / GATK HaplotypeCaller** (Six Illumina short-read callers whose call sets define the 'shared' SV baseline against which nanopore-specific calls are identified)
- **UCSC RepeatMasker / DFAM** (Annotate repeat content of SVs to flag retrotransposon/LINE insertions among nanopore-specific calls)

## Examples

```
lastal -Q1 -p my-params refdb reads.fastq | last-split | maf-convert -f ref.dict sam | sambamba view -h -S --format=bam /dev/stdin | sambamba sort /dev/stdin -o reads.sorted.bam
```

## Evaluation signals

- Nanopore-specific (novel) SV fraction should fall near ~14%, well below the 89% reported for PB data
- Class distribution should be deletion-dominated
- Nanopore-specific deletions should be enriched below 200 bp in the size distribution
- Nanopore-specific sites should show elevated LINE/retrotransposon content per RepeatMasker/DFAM annotation
- Mean GC content should be higher at nanopore-specific SV sites than at shared sites

## Limitations

- Conservative SV calling and extremely stringent post-calling filtering lower the estimated sensitivity (~75%) and deliberately depress the novel-variant fraction, so absolute counts depend on these filter choices
- Medium coverage (11–16×) and intrinsic nanopore sequencing biases limit detection of some SV classes
- Creating larger SV catalogs in complex repeat regions and segmental duplications remains an unresolved challenge, biasing repeat-enrichment estimates

## Evidence

- [abstract] Genome-wide surveillance of inherited SVs reveals novel variants missed in short-read data sets, a large proportion of which are retrotransposon insertions: "genome-wide surveillance of inherited SVs reveals novel variants, missed in short-read data sets, a large proportion of which are retrotransposon insertions"
- [discussion] Novel fraction lower than PB data due to conservative calling: "this percentage of novel variants is lower than for previously reported PB data (89%)"
- [discussion] Conservative calling and filtering lower the novel fraction: "this is partly due to our conservative SV calling and post-calling filtering steps"
- [discussion] 1KG comparison used to define novel SVs: "as well as comparison to 1KG variant sites"
- [readme] NanoSV algorithm definition: "The core algorithm of NanoSV identifies split- and gapped-aligned reads and clusters the reads according to the orientations and genomic positions of the read segments to define breakpoint-junctions"
