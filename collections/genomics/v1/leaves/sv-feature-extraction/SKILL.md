---
name: sv-feature-extraction
description: Use when you have a NanoSV VCF of SV calls from long nanopore reads and
  need per-call numeric features to train or apply a machine-learning filter (e.g.
  random forest in scikit-learn) that separates true SVs from false positives beyond
  baseline precision (~54%).
when_to_use_negative:
- You need an SV caller itself — use NanoSV (or minimap2/ngmlr + NanoSV) to produce
  the call set first; feature extraction assumes calls already exist in a VCF.
- Input alignments are short-read Illumina (e.g. from six short-read variant calling
  methods); this feature set is derived from long-read NanoSV split/gapped-read breakpoint-junction
  output.
- You only need raw call sets without ML-based filtering — NanoSV's own config thresholds
  (min_mapq 20, min_pid 0.7, cluster_count 2) may suffice without a random forest.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: NanoSV
  role: Produces the SV call VCF whose per-call fields supply the 12 features (VAF,
    PID, CIPOS/CIEND, plength, MAPQ, coverage); its config.ini defines the detection
    parameters upstream of feature extraction
  repo: https://github.com/mroosmalen/nanosv
- name: LAST
  role: Long-read mapper whose split alignments feed NanoSV; the paper notes LAST
    alignments give the most accurate results for SV calling with NanoSV
- name: sambamba
  role: Converts LAST MAF output to SAM→BAM and sorts it, producing the coordinate-sorted,
    indexed BAM that NanoSV requires for coverage and breakpoint-junction feature
    computation
  repo: https://github.com/biod/sambamba
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-017-01343-4
    title: Mapping and phasing of structural variation in patient genomes using nanopore
      sequencing
  source_content_hash: sha256:7ebf3cfd8efc9a217a9ac765fd1b1908060cc04322a5344db9f4f6c7405b3db3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mroosmalen/nanosv
  tools:
  - LAST
  - NanoSV
  - sambamba
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# sv-feature-extraction

## Summary

Extract per-call features from NanoSV structural-variant call output — including VAF, PID, CIPOS/CIEND confidence intervals, plength, MAPQ, and local coverage — to build a feature matrix for training a random forest SV filtering classifier.

## When to use

Use when you have a NanoSV VCF of SV calls from long nanopore reads and need per-call numeric features to train or apply a machine-learning filter (e.g. random forest in scikit-learn) that separates true SVs from false positives beyond baseline precision (~54%).

## When NOT to use

- You need an SV caller itself — use NanoSV (or minimap2/ngmlr + NanoSV) to produce the call set first; feature extraction assumes calls already exist in a VCF.
- Input alignments are short-read Illumina (e.g. from six short-read variant calling methods); this feature set is derived from long-read NanoSV split/gapped-read breakpoint-junction output.
- You only need raw call sets without ML-based filtering — NanoSV's own config thresholds (min_mapq 20, min_pid 0.7, cluster_count 2) may suffice without a random forest.

## Inputs

- NanoSV output VCF of SV calls from coordinate-sorted, indexed BAM of MinION nanopore reads
- Benchmark SV site sets (1KG / Pacific Biosciences SV sites) for TP/FP labeling
- NanoSV configuration (config.ini) specifying detection thresholds (max_split, min_pid, min_mapq)

## Outputs

- Labeled feature matrix (12 features per SV call: VAF, PID, CIPOS/CIEND, plength, MAPQ, local coverage, etc.)
- Per-call TP/FP labels based on overlap with 1KG/PB benchmark SV sites

## How to apply

Run NanoSV on LAST-aligned MinION reads (the paper found LAST alignments give the most accurate results for NanoSV SV calling) to produce the SV call set. Label each call as a true positive if it overlaps a 1KG or Pacific Biosciences benchmark SV site, otherwise a false positive. From the NanoSV output VCF, extract 12 features per call, including variant allele fraction (VAF), percentage of identity (PID), confidence intervals CIPOS/CIEND, pattern length (plength), mapping quality (MAPQ), and local coverage. Assemble the labeled feature matrix and feed it to a random forest classifier (scikit-learn) for filtering-model training; this skill covers feature construction, not classifier training or threshold selection.

## Related tools

- **NanoSV** (Produces the SV call VCF whose per-call fields supply the 12 features (VAF, PID, CIPOS/CIEND, plength, MAPQ, coverage); its config.ini defines the detection parameters upstream of feature extraction) — https://github.com/mroosmalen/nanosv
- **LAST** (Long-read mapper whose split alignments feed NanoSV; the paper notes LAST alignments give the most accurate results for SV calling with NanoSV)
- **sambamba** (Converts LAST MAF output to SAM→BAM and sorts it, producing the coordinate-sorted, indexed BAM that NanoSV requires for coverage and breakpoint-junction feature computation) — https://github.com/biod/sambamba

## Examples

```
NanoSV -s sambamba -c config.ini -b human_hg19.bed -o nanosv_calls.vcf reads.sorted.bam
```

## Evaluation signals

- Feature matrix contains exactly 12 features per call, including the named ones: VAF, PID, CIPOS/CIEND, plength, MAPQ, and local coverage.
- TP/FP labels agree with benchmark overlap: a call is TP only if it overlaps a 1KG or Pacific Biosciences benchmark SV site.
- Upstream NanoSV parameters match the paper's defaults (min_pid = 0.7, min_mapq = 20, cluster_count = 2) so features are comparable to the published model.
- Trained on these features, the random forest reaches the paper's operating point of 82% precision and 75% sensitivity on training data (Supplementary Fig. 19), versus a ~54% baseline precision.
- Features derive from a coordinate-sorted, indexed BAM generated via LAST → maf-convert → sambamba pipeline as described in the NanoSV user guide.

## Limitations

- Feature and label quality depend on medium coverage (11–16×) and intrinsic nanopore sequencing biases, which limit sensitivity.
- The 82% precision / 75% sensitivity figures are on training data; extremely stringent filtering is used to reach the ~75% estimated sensitivity benchmark, lowering the fraction of novel variants.
- Labeling depends on benchmark SV site sets (1KG, PB); SVs absent from these benchmarks are conservatively labeled FP, so the feature matrix under-represents genuinely novel variants.
- NanoSV output is tested for human genomes (default bed file human_hg19.bed); coverage-depth features require an appropriate BED file for other genomes.

## Evidence

- [supplementary] Extract 12 features per call from the NanoSV output, including variant allele fraction (VAF), percentage of identity (PID), confidence intervals CIPOS/CIEND, pattern length (plength), mapping quality (MAPQ), and local coverage.: [evidence span withheld — anchor status: not_found]
- [readme] We found that LAST alignments give the most accurate results for SV calling with NanoSV.: "We found that LAST alignments give the most accurate results for SV calling with NanoSV. However, mapping with LAST requires more compute resources."
- [discussion] extremely stringent filtering that results in a good estimated sensitivity (~75%): "we employ extremely stringent filtering that results in a good estimated sensitivity (~75%)"
- [discussion] SV discovery/genotyping pipeline achieves >95% precision with ~75% estimated sensitivity: [evidence span withheld — anchor status: not_found]
