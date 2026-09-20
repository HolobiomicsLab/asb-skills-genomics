---
name: random-forest-variant-classification
description: Use when a raw genome-wide NanoSV VCF call set from MinION nanopore reads
  must be reduced to a high-confidence SV set, and a repeat filter has already (or
  will) produced the intermediate filtered call set the classifier is trained to score.
when_to_use_negative:
- The input is a raw NanoSV call set that has not yet passed the homopolymer/simple-repeat
  filter — the random forest is the second stage of the cascade and expects repeat-filtered
  records
- The data are short-read (Illumina) variant calls — the classifier was trained on
  NA12878 nanopore SV features and NanoSV's split-read/gapped-alignment statistics,
  not short-read evidence
- Maximum sensitivity is the priority rather than precision — this filter is explicitly
  stringent and drops an estimated ~25% of true SVs
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0622
tools:
- name: NanoSV
  role: SV caller whose ancillary random forest classifier (trained on NA12878) and
    config.ini threshold parameters implement the second-stage filtering of patient
    NanoSV call sets
  repo: https://github.com/mroosmalen/nanosv
- name: sambamba
  role: Used to convert, sort and coordinate-index the LAST-mapped BAM files that
    feed NanoSV, the upstream prerequisite for the filtered call set being classified
  repo: https://github.com/biod/sambamba
- name: pysam/cyvcf2
  role: Python libraries used to load and parse the patient NanoSV VCF call sets before
    scoring with the random forest model
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-017-01343-4
    title: Mapping and phasing of structural variation in patient genomes using nanopore
      sequencing
  source_content_hash: sha256:435757cb60f0499b5cb6b30ff67eb1e692a48304a627845e159ad75260099855
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mroosmalen/nanosv
  tools:
  - NanoSV
  - pysam/cyvcf2
  - sambamba
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# random-forest-variant-classification

## Summary

Second-stage post-calling filter for NanoSV structural variant calls: after homopolymer/simple-repeat filtering, each remaining SV is scored by an ancillary random forest classifier trained on the NA12878 nanopore genome, and only SVs passing the classifier probability threshold (set via NanoSV's config.ini) are retained as high-confidence calls.

## When to use

Use when a raw genome-wide NanoSV VCF call set from MinION nanopore reads must be reduced to a high-confidence SV set, and a repeat filter has already (or will) produced the intermediate filtered call set the classifier is trained to score.

## When NOT to use

- The input is a raw NanoSV call set that has not yet passed the homopolymer/simple-repeat filter — the random forest is the second stage of the cascade and expects repeat-filtered records
- The data are short-read (Illumina) variant calls — the classifier was trained on NA12878 nanopore SV features and NanoSV's split-read/gapped-alignment statistics, not short-read evidence
- Maximum sensitivity is the priority rather than precision — this filter is explicitly stringent and drops an estimated ~25% of true SVs

## Inputs

- Initial genome-wide NanoSV VCF call set from MinION nanopore reads (e.g. 36,959 SVs for Patient 1; 36,321 SVs for Patient 2)
- Repeat-filtered intermediate VCF call set (8578 SVs Patient 1; 6791 SVs Patient 2) after homopolymer/simple-repeat removal
- Ancillary random forest classifier trained on the NA12878 nanopore genome, distributed with NanoSV
- NanoSV config.ini with classifier probability-threshold parameters

## Outputs

- Final high-confidence SV call set in VCF format (high_confidence_sv_calls.vcf; 3271 SVs for Patient 1, 3345 SVs for Patient 2)
- Per-patient intermediate filtered counts (8578 and 6791 SVs) for validation

## How to apply

Load the initial NanoSV VCF call sets (e.g. 36,959 SVs for Patient 1 and 36,321 SVs for Patient 2) with pysam/cyvcf2. Apply the first-stage homopolymer and simple-repeat filters using the repeat-filter configuration distributed with NanoSV, and check the intermediate counts match the reported 8578 SVs (Patient 1) and 6791 SVs (Patient 2). Then score each remaining SV with the random forest classifier trained on the NA12878 nanopore genome, taking the probability cutoff from NanoSV's configuration parameters (the [Filter options] in config.ini, e.g. min_pid = 0.7, min_mapq = 20 govern the segment features feeding the model). Retain only SVs passing the classifier cutoff as the final high-confidence call set — 3271 SVs for Patient 1 and 3345 SVs for Patient 2 in the published run — and write them as valid VCF SV records. The rationale is the paper's deliberately conservative strategy: extremely stringent post-calling filtering trades sensitivity (~75%) for a precision benchmark above 95%.

## Related tools

- **NanoSV** (SV caller whose ancillary random forest classifier (trained on NA12878) and config.ini threshold parameters implement the second-stage filtering of patient NanoSV call sets) — https://github.com/mroosmalen/nanosv
- **sambamba** (Used to convert, sort and coordinate-index the LAST-mapped BAM files that feed NanoSV, the upstream prerequisite for the filtered call set being classified) — https://github.com/biod/sambamba
- **pysam/cyvcf2** (Python libraries used to load and parse the patient NanoSV VCF call sets before scoring with the random forest model)

## Examples

```
NanoSV -c config.ini -s sambamba -t 8 -o nanosv_patient1.vcf reads.sorted.bam
```

## Evaluation signals

- After repeat filtering, per-patient record counts equal 8578 SVs (Patient 1) and 6791 SVs (Patient 2)
- Final high-confidence call set sizes equal 3271 SVs (Patient 1) and 3345 SVs (Patient 2)
- Every retained record is a valid VCF structural-variant entry with proper breakend fields
- No SV is scored without having first passed the homopolymer/simple-repeat filter
- Filter cascade stringency is consistent with the paper's benchmark: >95% precision at an estimated ~75% sensitivity

## Limitations

- The filtering cascade is deliberately extremely stringent, reducing estimated sensitivity to ~75% at medium (11–16x) nanopore coverage
- The classifier is trained on NA12878 nanopore data, so transfer to other samples/genomes assumes similar feature distributions
- SVs in complex repeat regions and segmental duplications remain difficult, limiting catalog completeness

## Evidence

- [discussion] we employ extremely stringent filtering that results in a good estimated sensitivity (~75%): "we employ extremely stringent filtering that results in a good estimated sensitivity (~75%)"
- [discussion] SV calls matching any state of the art precision benchmark (>95% precision): [evidence span withheld — anchor status: not_found]
- [readme] NanoSV uses a config.ini file which contains default settings for all running parameters.: "NanoSV uses a config.ini file which contains default settings for all running parameters. Users can change the parameters by creating their own config.ini file and provide this as a command line"
- [readme] The core algorithm of NanoSV identifies split- and gapped-aligned reads: "The core algorithm of NanoSV identifies split- and gapped-aligned reads and clusters the reads according to the orientations and genomic positions of the read segments"
