---
name: vcf-manipulation-and-filtering
description: Use when a raw genome-wide NanoSV VCF call set (tens of thousands of
  SV records) must be reduced to a high-confidence set via the repeat-region filter
  plus the random forest classifier trained on NA12878 nanopore data.
when_to_use_negative:
- Input is raw BAM/FASTQ long-read data — NanoSV SV calling must run first to produce
  the VCF call set that this skill filters
- The call set has already passed the homopolymer/simple-repeat filter and the NA12878
  random forest classifier — re-applying would be redundant
- The goal is SNV calling or phasing rather than structural variant confidence filtering
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: NanoSV
  role: Produced the initial SV VCF call sets and supplies the repeat-filter configuration,
    ancillary random forest files, and config.ini parameters used to threshold the
    classifier
  repo: https://github.com/mroosmalen/nanosv
- name: sambamba
  role: BAM sorting/viewing prerequisite in the NanoSV pipeline that generates the
    coordinate-sorted, indexed BAM consumed by SV calling
  repo: https://github.com/biod/sambamba
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-017-01343-4
    title: Mapping and phasing of structural variation in patient genomes using nanopore
      sequencing
  source_content_hash: sha256:ec89776f27548d663f7f1d3e4e52a3ace6ffd3a45827b515b4ff0bcebda33c3b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mroosmalen/nanosv
  tools:
  - NanoSV
  - sambamba
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# vcf-manipulation-and-filtering

## Summary

Multi-stage post-calling filtering of NanoSV VCF call sets: remove SVs overlapping homopolymer/simple-repeat regions using NanoSV's repeat-filter configuration, then score survivors with the NA12878-trained random forest classifier to retain only high-confidence structural variants.

## When to use

Use when a raw genome-wide NanoSV VCF call set (tens of thousands of SV records) must be reduced to a high-confidence set via the repeat-region filter plus the random forest classifier trained on NA12878 nanopore data.

## When NOT to use

- Input is raw BAM/FASTQ long-read data — NanoSV SV calling must run first to produce the VCF call set that this skill filters
- The call set has already passed the homopolymer/simple-repeat filter and the NA12878 random forest classifier — re-applying would be redundant
- The goal is SNV calling or phasing rather than structural variant confidence filtering

## Inputs

- Initial genome-wide NanoSV VCF call sets (Patient 1: 36,959 SVs; Patient 2: 36,321 SVs)
- Repeat-filter configuration distributed with NanoSV (homopolymer / simple-repeat regions)
- Random forest classifier ancillary file trained on the NA12878 nanopore genome
- NanoSV config.ini parameters defining the classifier probability threshold

## Outputs

- Intermediate repeat-filtered VCF call sets (8578 SVs for Patient 1; 6791 SVs for Patient 2)
- Final high-confidence VCF call sets (3271 SVs for Patient 1; 3345 SVs for Patient 2), e.g. high_confidence_sv_calls.vcf

## How to apply

Load the initial NanoSV VCF call sets (Patient 1: 36,959 SVs; Patient 2: 36,321 SVs) with pysam/cyvcf2. Apply the first-stage filter that removes SVs overlapping homopolymer and simple-repeat regions, using the repeat-filter configuration distributed with NanoSV, and checkpoint the intermediate counts (8578 for Patient 1, 6791 for Patient 2). Score each remaining SV with the ancillary random forest classifier trained on the NA12878 nanopore genome, using NanoSV's config.ini parameters to set the probability cutoff. Retain only SVs passing the classifier threshold to form the final high-confidence call sets (3271 SVs for Patient 1, 3345 for Patient 2), written in VCF format. The article notes this pipeline uses extremely stringent filtering, trading sensitivity (~75%) for >95% precision.

## Related tools

- **NanoSV** (Produced the initial SV VCF call sets and supplies the repeat-filter configuration, ancillary random forest files, and config.ini parameters used to threshold the classifier) — https://github.com/mroosmalen/nanosv
- **sambamba** (BAM sorting/viewing prerequisite in the NanoSV pipeline that generates the coordinate-sorted, indexed BAM consumed by SV calling) — https://github.com/biod/sambamba

## Examples

```
from cyvcf2 import VCF; kept = [r for r in VCF('patient1_nanosv.vcf') if passes_repeat_filter(r) and rf_score(r) >= threshold]; write_vcf(kept, 'high_confidence_sv_calls.vcf')
```

## Evaluation signals

- After repeat filtering, per-patient row counts must equal the reported intermediate values: 8578 SVs (Patient 1) and 6791 SVs (Patient 2)
- Final high-confidence call sets must contain exactly 3271 SVs (Patient 1) and 3345 SVs (Patient 2)
- Every output record must be a valid VCF SV entry
- Each retained SV must have passed the random forest probability cutoff defined by NanoSV's configuration parameters
- No retained SV should overlap homopolymer or simple-repeat regions defined by the NanoSV repeat-filter configuration

## Limitations

- The filtering cascade is extremely stringent: estimated sensitivity drops to ~75% in exchange for >95% precision
- Conservative SV calling and post-calling filtering steps lower the fraction of novel variants relative to other long-read datasets
- SVs in complex repeat regions and segmental duplications remain a challenge; repeat filtering will remove calls overlapping homopolymers and simple repeats even if genuine

## Evidence

- [discussion] we employ extremely stringent filtering that results in a good estimated sensitivity (~75%): "we employ extremely stringent filtering that results in a good estimated sensitivity (~75%)"
- [discussion] SV calls matching any state of the art precision benchmark: "SV calls matching any state of the art precision benchmark (>95% precision)"
- [readme] NanoSV uses a config.ini file which contains default settings for all running parameters: "NanoSV uses a config.ini file which contains default settings for all running parameters. Users can change the parameters by creating their own config.ini"
- [readme] NanoSV is a software package that can be used to identify structural genomic variations in long-read sequencing data: "NanoSV is a software package that can be used to identify structural genomic variations in long-read sequencing data, such as data produced by Oxford Nanopore Technologies’ MinION"
- [discussion] this is partly due to our conservative SV calling and post-calling filtering steps: "this is partly due to our conservative SV calling and post-calling filtering steps"
