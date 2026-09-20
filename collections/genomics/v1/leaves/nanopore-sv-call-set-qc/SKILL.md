---
name: nanopore-sv-call-set-qc
description: Use when a raw NanoSV VCF from MinION long-read data (tens of thousands
  of SV calls) must be reduced to a high-confidence SV set and intermediate/final
  call counts must match published checkpoints.
when_to_use_negative:
- Input is already a filtered/high-confidence SV set — re-applying the cascade would
  remove genuine calls and break the 3271/3345 count expectations
- The goal is calling SVs de novo from reads; this skill only post-filters existing
  NanoSV output and does not replace the 'SV calling using NanoSV' step on the coordinate-sorted
  BAM
- The data are short-read Illumina variant calls rather than long-read nanopore NanoSV
  calls — the repeat filters and NA12878-trained classifier are calibrated for NanoSV
  output
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: NanoSV
  role: Produces the raw genome-wide long-read SV call sets (split- and gapped-read
    breakpoint clustering) and supplies the config.ini parameters, repeat-filter configuration,
    and NA12878-trained random forest ancillary file used in the filtering cascade
  repo: https://github.com/mroosmalen/nanosv
- name: sambamba
  role: Converts and coordinate-sorts the SAM-to-BAM alignments that feed NanoSV,
    ensuring the BAM input to calling (and thus the VCFs being filtered) is sorted
    and indexed
  repo: https://github.com/biod/sambamba
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-017-01343-4
    title: Mapping and phasing of structural variation in patient genomes using nanopore
      sequencing
  source_content_hash: sha256:be782384f5bd6a140bc8c085a2c6c6b702f93af098d9e5efacc29e7814d0ca40
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mroosmalen/nanosv
  tools:
  - NanoSV
  - sambamba
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# nanopore-sv-call-set-qc

## Summary

Reduces raw genome-wide NanoSV call sets from nanopore patient genomes to high-confidence structural variant sets via a two-stage cascade: homopolymer/simple-repeat region filtering followed by a random forest classifier trained on the NA12878 nanopore genome, with validated call-count checkpoints at each stage.

## When to use

Use when a raw NanoSV VCF from MinION long-read data (tens of thousands of SV calls) must be reduced to a high-confidence SV set and intermediate/final call counts must match published checkpoints.

## When NOT to use

- Input is already a filtered/high-confidence SV set — re-applying the cascade would remove genuine calls and break the 3271/3345 count expectations
- The goal is calling SVs de novo from reads; this skill only post-filters existing NanoSV output and does not replace the 'SV calling using NanoSV' step on the coordinate-sorted BAM
- The data are short-read Illumina variant calls rather than long-read nanopore NanoSV calls — the repeat filters and NA12878-trained classifier are calibrated for NanoSV output

## Inputs

- Initial genome-wide NanoSV VCF call set per patient (Patient 1: 36,959 SVs; Patient 2: 36,321 SVs)
- Repeat-filter configuration file distributed with NanoSV (homopolymer and simple-repeat regions)
- Random forest classifier trained on the NA12878 nanopore genome (NanoSV ancillary file)
- NanoSV config.ini with default classifier/filter parameters

## Outputs

- Intermediate repeat-filtered VCF per patient (8578 SVs for Patient 1; 6791 SVs for Patient 2)
- high_confidence_sv_calls.vcf: final high-confidence SV call set (3271 SVs for Patient 1; 3345 SVs for Patient 2)

## How to apply

Load the initial NanoSV VCF call sets (36,959 SVs for Patient 1; 36,321 SVs for Patient 2) with pysam or cyvcf2. Apply the first-stage post-calling filters that remove SVs overlapping homopolymer and simple-repeat regions, using the repeat-filter configuration distributed with NanoSV, and verify the intermediate counts equal 8578 (Patient 1) and 6791 (Patient 2). Score each remaining SV with the ancillary random forest classifier trained on the NA12878 nanopore genome, using NanoSV's config.ini parameters to set the probability threshold. Retain only SVs passing the classifier cutoff — 3271 SVs for Patient 1 and 3345 SVs for Patient 2 — and write them in VCF format. This cascade is intentionally 'extremely stringent', so the counts are the primary correctness check: any deviation indicates a misapplied filter config or wrong classifier threshold.

## Related tools

- **NanoSV** (Produces the raw genome-wide long-read SV call sets (split- and gapped-read breakpoint clustering) and supplies the config.ini parameters, repeat-filter configuration, and NA12878-trained random forest ancillary file used in the filtering cascade) — https://github.com/mroosmalen/nanosv
- **sambamba** (Converts and coordinate-sorts the SAM-to-BAM alignments that feed NanoSV, ensuring the BAM input to calling (and thus the VCFs being filtered) is sorted and indexed) — https://github.com/biod/sambamba

## Examples

```
NanoSV -t 8 -s sambamba -c config.ini -b human_hg19.bed -o sv_calls.vcf reads.sorted.bam
```

## Evaluation signals

- After homopolymer/simple-repeat filtering, the per-patient VCF row counts must equal the reported intermediates: 8578 SVs for Patient 1 and 6791 SVs for Patient 2
- Final high-confidence call set sizes must equal 3271 SVs (Patient 1) and 3345 SVs (Patient 2)
- Every output record must be a valid VCF SV entry (parses cleanly with pysam/cyvcf2)
- The pipeline as a whole targets >95% precision benchmark; the filtering cascade is deliberately stringent, so over-retention versus expected counts signals a misconfigured classifier threshold
- Estimated sensitivity of the stringent cascade should be approximately 75%; far fewer retained SVs than the reported final counts suggest over-filtering

## Limitations

- The extremely stringent filtering reduces estimated sensitivity to ~75%, so genuine SVs are expected to be discarded
- Medium MinION coverage (11–16×) and intrinsic nanopore sequencing biases limit which SVs survive calling and filtering
- The conservative SV calling and post-calling filtering steps lower the fraction of novel variants relative to other long-read datasets
- The random forest is trained on the NA12878 nanopore genome; applying it to other samples assumes their call characteristics transfer

## Evidence

- [discussion] we employ extremely stringent filtering that results in a good estimated sensitivity (~75%): "we employ extremely stringent filtering that results in a good estimated sensitivity (~75%)"
- [discussion] SV calls matching any state of the art precision benchmark (>95% precision): "SV calls matching any state of the art precision benchmark (>95% precision)"
- [readme] NanoSV is a software package that can be used to identify structural genomic variations in long-read sequencing data: "NanoSV is a software package that can be used to identify structural genomic variations in long-read sequencing data"
- [readme] NanoSV uses a config.ini file which contains default settings for all running parameters.: "NanoSV uses a config.ini file which contains default settings for all running parameters."
- [readme] The core algorithm of NanoSV identifies split- and gapped-aligned reads and clusters the reads according to the orientations and genomic positions: "The core algorithm of NanoSV identifies split- and gapped-aligned reads and clusters the reads according to the orientations and genomic positions of the read segments to define breakpoint-junctions"
