---
name: cross-species-motif-methylation-comparison
description: Use when you have per-read ONT methylation calls for several strains
  and an orthogonal PacBio motif list, and need a species-by-motif percentage table
  to confirm methyltransferase target motifs across a reference community.
when_to_use_negative:
- Model has not been trained on a sequence context similar to the target motif - the
  article notes training biases can decrease accuracy at contexts with lower similarity
  to the E. coli training examples, as seen for one S. aureus motif
- You are trying to distinguish methylation types at ambiguous motifs such as GATCGVNY
  in S. aureus, where both single-molecule methods fail to distinguish between methylation
  types
- Working with the two Dikarya fungi (C. neoformans and S. cerevisiae), where the
  study could not confidently assign motifs or identify consistently detected sites
edam_operation: http://edamontology.org/operation_3186
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_0654
tools:
- name: mCaller
  role: Per-position, multi-read m6A prediction using deviations between measured
    and expected nanopore current; produces the per-read methylation calls aggregated
    in this skill
  repo: https://github.com/al-mcintyre/mCaller
- name: nanopolish
  role: eventalign --scale-events produces the scaled event-alignment TSV that mCaller
    consumes; index prepares fast5/fastq inputs
  repo: https://github.com/jts/nanopolish
- name: pandas
  role: Aggregates per-position methylation bed data into the species-by-motif percentage
    table
- name: pysam
  role: Reads bed/BAM-derived per-position records during aggregation with pandas
- name: bwa mem
  role: Aligns fastq reads to each strain reference assembly (ont2d preset) before
    event alignment
- name: samtools
  role: Sorts and indexes BAM alignments after bwa mem
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-08289-9
    title: Single-molecule sequencing detection of N6-methyladenine in microbial reference
      materials
  source_content_hash: sha256:61bd8bb8ab524a77962d14dc2d4ecade7497f3f893de29de22afed78a63b7015
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/al-mcintyre/mCaller
  tools:
  - bwa mem
  - mCaller
  - nanopolish
  - pandas
  - pysam
  - samtools
  edam_operation: http://edamontology.org/operation_3186
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# cross-species-motif-methylation-comparison

## Summary

Aggregates mCaller per-position methylation predictions across multiple microbial strains and motifs to compute the fraction of PacBio-predicted methyltransferase target motif sites classified as methylated per species and per motif, enabling cross-species confirmation of motif calls in the ZymoBIOMICS reference community.

## When to use

Use when you have per-read ONT methylation calls for several strains and an orthogonal PacBio motif list, and need a species-by-motif percentage table to confirm methyltransferase target motifs across a reference community.

## When NOT to use

- Model has not been trained on a sequence context similar to the target motif - the article notes training biases can decrease accuracy at contexts with lower similarity to the E. coli training examples, as seen for one S. aureus motif
- You are trying to distinguish methylation types at ambiguous motifs such as GATCGVNY in S. aureus, where both single-molecule methods fail to distinguish between methylation types
- Working with the two Dikarya fungi (C. neoformans and S. cerevisiae), where the study could not confidently assign motifs or identify consistently detected sites

## Inputs

- mCaller per-read methylation probability outputs for each ZymoBIOMICS strain-motif combination (from r95_twobase_model_NN_6_m6A.pkl)
- make_bed.py per-position methylation summary bed files (filtered by minimum read depth and mod_threshold)
- PacBio-predicted methyltransferase target motif site lists per strain (e.g. GATC motifs)
- Strain reference assembly FASTA files and sorted/indexed BAM alignments from bwa mem -x ont2d
- nanopolish eventalign --scale-events TSV files per strain

## Outputs

- fig3a_species_motif_percent_methylated.csv: species-by-motif table of the percentage of PacBio-predicted motif sites classified as methylated
- Per-motif, per-species methylation percentages suitable for confirming or refuting PacBio motif identifications

## How to apply

Run mCaller.py per strain-motif combination with a pre-trained r95_twobase_model_NN_6_m6A.pkl model on the nanopolish eventalign --scale-events TSV, then run make_bed.py with -d (minimum read depth for inclusion) and -t 0.5 (--mod_threshold, the minimum fraction of per-read observations labeled methylated) to produce per-position methylation summary bed files. Using pandas and pysam, aggregate the fraction of PacBio-predicted motif sites classified as methylated for each species and each motif. Write the aggregated species-by-motif percentage table to fig3a_species_motif_percent_methylated.csv. Enrichment of most PacBio-predicted target motifs confirms their identification; expect 60-98% per-position accuracy at most motifs but watch for motifs where the E. coli-trained model fails.

## Related tools

- **mCaller** (Per-position, multi-read m6A prediction using deviations between measured and expected nanopore current; produces the per-read methylation calls aggregated in this skill) — https://github.com/al-mcintyre/mCaller
- **nanopolish** (eventalign --scale-events produces the scaled event-alignment TSV that mCaller consumes; index prepares fast5/fastq inputs) — https://github.com/jts/nanopolish
- **pandas** (Aggregates per-position methylation bed data into the species-by-motif percentage table)
- **pysam** (Reads bed/BAM-derived per-position records during aggregation with pandas)
- **bwa mem** (Aligns fastq reads to each strain reference assembly (ont2d preset) before event alignment)
- **samtools** (Sorts and indexes BAM alignments after bwa mem)

## Examples

```
mCaller.py -m GATC -r <reference>.fasta -d r95_twobase_model_NN_6_m6A.pkl -e <filename>.eventalign.tsv -f <filename>.fastq -b A
```

## Evaluation signals

- Most PacBio-predicted target motifs should show clear enrichment (higher fraction methylated) relative to non-motif background positions
- Per-position accuracy should fall in the reported 60-98% range at most motifs; a notable failure at one S. aureus motif indicates incomplete motif coverage
- The output CSV must contain one percentage value per species-per-motif combination, computed as fraction of PacBio-predicted motif sites labeled methylated by make_bed.py
- Per-position summaries must have sufficient read depth to pass the -d filter before aggregation
- Non-motif site calls should be treated as method-specific false positives reflecting different noise sources between nanopore current deviations and PacBio IPD ratios

## Limitations

- mCaller showed 60-98% per-position accuracy at most motifs, with notable failure at one S. aureus motif
- Training biases could decrease accuracy at contexts with lower similarity to the E. coli training examples
- Both single-molecule sequencing methods can fail to distinguish between types of methylation, as seen for the GATCGVNY motif in S. aureus
- Non-motif site calls are method-specific false positives reflecting different sources of noise for nanopore current deviations and PacBio IPD ratios
- Nanopore sequencing detects m6A with lower per-read accuracy (~70%) than m5C (upwards of 80%), so per-read calls are noisy

## Evidence

- [results] per-position multi-read predictions: [evidence span withheld — anchor status: not_found]
- [methods] make_bed parameters: "`-d` indicates the minimum read depth for inclusion"
- [methods] mod_threshold: "while `-t` indicates the `--mod_threshold`, or the minimum fraction of observations of a position (ie. currents deviations in individual reads) labelled as methylated"
- [discussion] motif failure: "mCaller showed 60–98% per-position accuracy at most motifs, with notable failure at one S. aureus motif"
- [discussion] methylation type ambiguity: "both single-molecule sequencing methods can fail to distinguish between types of methylation, as we saw for the GATCGVNY motif in S. aureus"
- [discussion] false positives: "suggesting that these are false positives that reflect different sources of noise for nanopore current deviations and PacBio IPD ratios"
- [readme] README dependencies: "This program is designed to call m6A from nanopore data using the differences between measured and expected currents."
