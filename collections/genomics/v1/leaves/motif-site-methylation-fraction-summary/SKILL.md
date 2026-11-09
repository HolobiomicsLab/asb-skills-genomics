---
name: motif-site-methylation-fraction-summary
description: Use when you must quantify the fraction of PacBio-predicted motif sites
  (e.g. GATC) called methylated by mCaller per-position ONT predictions across multiple
  strains, to reproduce a species-by-motif enrichment table like the paper's fig3a_species_motif_percent_methylated.csv.
when_to_use_negative:
- You need the per-read classification itself (mCaller.py on the eventalign TSV) rather
  than a downstream per-site fraction summary — this skill operates on mCaller output,
  not raw fast5/fastq.
- The target sequence context was not part of the E. coli training set — the paper
  warns training biases can decrease accuracy at contexts dissimilar to the training
  examples, so fraction summaries there are unreliable.
- You need de novo motif discovery or methylation-type assignment (e.g. the GATCGVNY
  motif in S. aureus) — this skill only quantifies methylation at pre-defined PacBio-predicted
  motif sites and cannot distinguish methylation types.
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0654
tools:
- name: mCaller
  role: Generates per-read m6A probability calls at motif positions from the scaled
    eventalign TSV (mCaller.py) and the per-position bed summary (make_bed.py) from
    which the motif-site fractions are computed
  repo: https://github.com/al-mcintyre/mCaller
- name: nanopolish
  role: eventalign with --scale-events produces the scaled event-alignment TSV per
    strain that mCaller consumes before per-position summarization
  repo: https://github.com/jts/nanopolish
- name: pandas
  role: Aggregates per-position methylation calls into the species-by-motif percent-methylated
    table and writes the CSV
- name: pysam
  role: Used alongside pandas to query positions and compute the fraction of PacBio-predicted
    motif sites classified as methylated
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-08289-9
    title: Single-molecule sequencing detection of N6-methyladenine in microbial reference
      materials
  source_content_hash: sha256:1dfbcdf842fcada62405bb798206bc5525676e35f4dd29006975286e705f6ad8
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
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# motif-site-methylation-fraction-summary

## Summary

Summarizes mCaller per-read m6A predictions into per-position methylation calls (make_bed.py with a -d read-depth floor and -t 0.5 mod_threshold), then computes the fraction of PacBio-predicted methyltransferase target motif sites called methylated per Zymo strain and motif, yielding a species-by-motif percentage table that cross-confirms motif identification.

## When to use

Use when you must quantify the fraction of PacBio-predicted motif sites (e.g. GATC) called methylated by mCaller per-position ONT predictions across multiple strains, to reproduce a species-by-motif enrichment table like the paper's fig3a_species_motif_percent_methylated.csv.

## When NOT to use

- You need the per-read classification itself (mCaller.py on the eventalign TSV) rather than a downstream per-site fraction summary — this skill operates on mCaller output, not raw fast5/fastq.
- The target sequence context was not part of the E. coli training set — the paper warns training biases can decrease accuracy at contexts dissimilar to the training examples, so fraction summaries there are unreliable.
- You need de novo motif discovery or methylation-type assignment (e.g. the GATCGVNY motif in S. aureus) — this skill only quantifies methylation at pre-defined PacBio-predicted motif sites and cannot distinguish methylation types.

## Inputs

- Per-read mCaller methylation probability output for each strain-motif combination (mCaller.py run on the scaled eventalign TSV)
- Scaled event-alignment TSV per Zymo strain (nanopolish eventalign --scale-events)
- PacBio-predicted methyltransferase target motif site lists per strain
- Strain reference assemblies (FASTA) and indexed nanopore reads (fastq + fast5 + sequencing_summary.txt)
- make_bed.py parameters: -d minimum read depth and -t 0.5 mod_threshold

## Outputs

- Per-position methylation summary bed file per strain-motif (make_bed.py output)
- Species-by-motif percentage table CSV (fig3a_species_motif_percent_methylated.csv) giving the fraction of PacBio-predicted motif sites classified as methylated

## How to apply

For each Zymo strain-motif pair, run mCaller.py on the scaled nanopolish eventalign TSV (e.g. -m GATC -d r95_twobase_model_NN_6_m6A.pkl -b A) to get per-read methylation probabilities, then run the make_bed.py summary script with -d (minimum read depth for inclusion) and -t 0.5 (--mod_threshold: the minimum fraction of per-read observations of a position labelled as methylated) to produce a per-position bed file. Match these per-position calls against each strain's PacBio-predicted motif-site list and, using pandas/pysam, aggregate the fraction of PacBio-predicted motif positions classified as methylated per species and per motif. Write the species-by-motif percentage table to a CSV (e.g. fig3a_species_motif_percent_methylated.csv). Correct application is judged by clear enrichment of most PacBio-predicted target motifs in their cognate strain — consistent with mCaller's reported 60–98% per-position accuracy at most motifs — with the known S. aureus failure motif treated as an expected exception.

## Related tools

- **mCaller** (Generates per-read m6A probability calls at motif positions from the scaled eventalign TSV (mCaller.py) and the per-position bed summary (make_bed.py) from which the motif-site fractions are computed) — https://github.com/al-mcintyre/mCaller
- **nanopolish** (eventalign with --scale-events produces the scaled event-alignment TSV per strain that mCaller consumes before per-position summarization) — https://github.com/jts/nanopolish
- **pandas** (Aggregates per-position methylation calls into the species-by-motif percent-methylated table and writes the CSV)
- **pysam** (Used alongside pandas to query positions and compute the fraction of PacBio-predicted motif sites classified as methylated)

## Examples

```
mCaller.py -m GATC -r <reference>.fasta -d r95_twobase_model_NN_6_m6A.pkl -e <filename>.eventalign.tsv -f <filename>.fastq -b A; python make_bed.py -d <min_depth> -t 0.5
```

## Evaluation signals

- Output CSV is a species-by-motif matrix of percentages with one entry per Zymo strain × motif combination, and PacBio-predicted target motifs show clear enrichment (high percent methylated) in their cognate strain while non-target species sit near background.
- Every position in the make_bed.py bed file satisfies the -d minimum read depth and only positions with at least the -t 0.5 fraction of per-read observations labeled methylated are counted.
- Per-position percentages at trained motifs fall within the reported 60–98% mCaller accuracy band, with the known one S. aureus failure motif flagged as the expected outlier.
- Spot-check several motif sites against the eventalign TSV: overlapping reads carry mCaller methylation probabilities consistent with the summarized per-position call.
- The table's enrichment pattern reproduces the paper's Fig 3a species-by-motif layout, confirming most PacBio-predicted motifs are recovered by mCaller.

## Limitations

- mCaller notably failed at one S. aureus motif; reported accuracy is 60–98% per-position at most motifs, so some target motifs will not enrich.
- Training on E. coli contexts means accuracy can decrease at sequence contexts dissimilar to the training examples, particularly for contexts the model hasn't been trained on.
- Both single-molecule methods can fail to distinguish types of methylation (e.g. the GATCGVNY motif in S. aureus), so a methylated fraction alone does not identify the modification type.
- Motifs could not be confidently assigned for the two Dikarya fungi (C. neoformans and S. cerevisiae), limiting summary-table coverage to well-assigned motifs.
- Non-motif site calls are method-specific false positives reflecting different noise sources between nanopore current deviations and PacBio IPD ratios.

## Evidence

- [methods] mCaller invocation for per-read motif calls: "mCaller.py <-m GATC or -p positions.txt> -r <reference>.fasta -d r95_twobase_model_NN_6_m6A.pkl -e <filename>.eventalign.tsv -f <filename>.fastq -b A"
- [methods] make_bed.py mod_threshold definition: "while `-t` indicates the `--mod_threshold`, or the minimum fraction of observations of a position (ie. currents deviations in individual reads) labelled as methylated"
- [abstract] motif confirmation purpose: "can be extended to detect known or confirm suspected methyltransferase target motifs based on predictions of methylation at untrained contexts"
- [discussion] expected per-position accuracy and known failure: "mCaller showed 60–98% per-position accuracy at most motifs, with notable failure at one S. aureus motif"
- [discussion] methylation-type ambiguity caveat: "both single-molecule sequencing methods can fail to distinguish between types of methylation, as we saw for the GATCGVNY motif in S. aureus"
- [readme] mCaller purpose from README: "This program is designed to call m6A from nanopore data using the differences between measured and expected currents."
