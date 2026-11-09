---
name: nanopore-sv-calling
description: Use when calling structural variants (deletions, inversions, translocations)
  from long nanopore reads at medium coverage (11–16×), or when reproducing the NanoSV
  SV discovery workflow from Cretu Stancu et al. 2017.
when_to_use_negative:
- Input is short-read Illumina data — NanoSV is designed for long reads (MinION/GridION/PromethION,
  PacBio RSII/Sequel); many SVs are missed in short-read data
- Coverage is too low for de novo assembly — the authors explicitly did not attempt
  a full genome assembly from MinION reads due to insufficient coverage
- The goal is comprehensive SV discovery in complex repeat regions and segmental duplications,
  which the authors flag as an unresolved challenge with conservative calling/filtering
edam_operation: http://edamontology.org/operation_3228
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: NanoSV
  role: 'Core SV caller: clusters split- and gapped-aligned long reads to define SV
    breakpoint-junctions and emit a VCF'
  repo: https://github.com/mroosmalen/nanosv
- name: LAST
  role: Preferred read mapper; last-train/lastal/last-split/maf-convert produce the
    SAM/BAM input that gives the most accurate NanoSV SV calls
- name: sambamba
  role: Converts SAM to BAM (sambamba view -h -S --format=bam), sorts BAMs, and is
    invoked internally by NanoSV
  repo: https://github.com/biod/sambamba
- name: Albacore
  role: Basecaller producing FASTQ from raw MinION/GridION/PromethION signal prior
    to mapping
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-017-01343-4
    title: Mapping and phasing of structural variation in patient genomes using nanopore
      sequencing
  source_content_hash: sha256:dec3044c6b557de18a5e8b78447cc45d2be50212942d05e66f03b02a6cc8173d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mroosmalen/nanosv
  tools:
  - Albacore
  - LAST
  - NanoSV
  - sambamba
  edam_operation: http://edamontology.org/operation_3228
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# nanopore-sv-calling

## Summary

Detect structural variants from Oxford Nanopore MinION long reads using the NanoSV pipeline (LAST mapping + split/gapped-read breakpoint clustering), achieving >95% precision with ~75% estimated sensitivity at 11–16× coverage.

## When to use

Use when calling structural variants (deletions, inversions, translocations) from long nanopore reads at medium coverage (11–16×), or when reproducing the NanoSV SV discovery workflow from Cretu Stancu et al. 2017.

## When NOT to use

- Input is short-read Illumina data — NanoSV is designed for long reads (MinION/GridION/PromethION, PacBio RSII/Sequel); many SVs are missed in short-read data
- Coverage is too low for de novo assembly — the authors explicitly did not attempt a full genome assembly from MinION reads due to insufficient coverage
- The goal is comprehensive SV discovery in complex repeat regions and segmental duplications, which the authors flag as an unresolved challenge with conservative calling/filtering

## Inputs

- Raw MinION nanopore signal or basecalled FASTQ reads (11–16× human genome coverage)
- Reference genome FASTA (e.g. hg19) plus picard sequence dictionary (.dict)
- NanoSV config.ini (default parameters) and coverage BED file (default human_hg19.bed)

## Outputs

- LAST-trained scoring parameters file ([my-params])
- Coordinate-sorted, indexed BAM of mapped long reads
- VCF of SV breakpoint-junction calls from NanoSV

## How to apply

Basecall raw MinION signal (e.g. with Albacore) to FASTQ, then map reads to the reference with LAST: build a lastdb index, train parameters with last-train on >10,000 sampled reads (last-train -Q1 [referencedb] [reads_sample.fastq]), align with lastal -Q1 -p [my-params] piped through last-split, convert MAF to SAM with maf-convert (using a picard CreateSequenceDictionary .dict for read-group tags), and convert/sort to BAM with sambamba. LAST alignments were found to give the most accurate NanoSV results, though BWA MEM (-x ont2d -M), minimap2 (-a), and NGMLR (-x ont) are supported. Run NanoSV on the coordinate-sorted, indexed BAM (default config: max_split=10, min_pid=0.7, min_mapq=20, cluster_distance=10, cluster_count=2, mate_distance=300); NanoSV clusters split- and gapped-aligned reads by segment orientation and position to define SV breakpoint-junctions and outputs a VCF. For human genomes where only intrachromosomal SVs matter, split the BAM per chromosome and run NanoSV per chromosome for speed. Apply extremely stringent post-calling filtering (e.g. the paper's random-forest filtering on VAF, PID, CIPOS/CIEND, plength, MAPQ and local coverage) to reach the >95% precision / ~75% sensitivity operating point.

## Related tools

- **NanoSV** (Core SV caller: clusters split- and gapped-aligned long reads to define SV breakpoint-junctions and emit a VCF) — https://github.com/mroosmalen/nanosv
- **LAST** (Preferred read mapper; last-train/lastal/last-split/maf-convert produce the SAM/BAM input that gives the most accurate NanoSV SV calls)
- **sambamba** (Converts SAM to BAM (sambamba view -h -S --format=bam), sorts BAMs, and is invoked internally by NanoSV) — https://github.com/biod/sambamba
- **Albacore** (Basecaller producing FASTQ from raw MinION/GridION/PromethION signal prior to mapping)

## Examples

```
lastal -Q1 -p my-params referencedb reads.fastq | last-split | maf-convert -f reference.dict sam /dev/stdin | sambamba view -h -S --format=bam /dev/stdin | sambamba sort /dev/stdin -o reads.sorted.bam; NanoSV -o svs.vcf reads.sorted.bam
```

## Evaluation signals

- SV calls matching any state-of-the-art precision benchmark (>95% precision) with a good estimated sensitivity (~75%) after stringent post-calling filtering
- All known de novo breakpoint junctions for Patient 1 are recovered, even at the relatively low coverage of 16×
- NanoSV input BAM is coordinate-sorted and indexed; LAST mapping is parameter-trained on >10,000 reads (last-train) rather than using default scores
- Output VCF contains reciprocal breakpoint-junction mates within mate_distance=300 (MATEID check) and calls pass default filters (min_pid=0.7, min_mapq=20, cluster_count=2)
- Called SVs overlap 1KG/PacBio benchmark SV sites at the expected rate; fraction of novel variants stays low due to conservative calling and post-calling filtering

## Limitations

- Sensitivity is limited by medium coverage and intrinsic nanopore sequencing biases
- Conservative SV calling and post-calling filtering lower the fraction of novel variants (89% novel in PB data vs 14% here)
- LAST mapping requires more compute resources than alternative mappers
- Creating larger SV catalogs in complex repeat regions and segmental duplications remains a particular challenge
- NanoSV is developed/tested primarily on human (hg19 BED default); other genomes require a custom coverage BED

## Evidence

- [readme] LAST alignments give the most accurate results for SV calling with NanoSV: "We found that LAST alignments give the most accurate results for SV calling with NanoSV. However, mapping with LAST requires more compute resources."
- [readme] NanoSV clusters split/gapped reads: "The core algorithm of NanoSV identifies split- and gapped-aligned reads and clusters the reads according to the orientations and genomic positions of the read segments"
- [discussion] Pipeline precision/sensitivity benchmark: [evidence span withheld — anchor status: not_found]
- [discussion] Stringent filtering step: "we employ extremely stringent filtering that results in a good estimated sensitivity (~75%)"
- [intro] Coverage range demonstrated: "we demonstrate sequencing of the whole diploid human genomes of two patients on the MinION sequencer at 11–16× depth of coverage."
