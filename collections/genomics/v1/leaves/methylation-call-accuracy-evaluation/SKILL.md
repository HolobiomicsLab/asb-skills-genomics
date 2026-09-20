---
name: methylation-call-accuracy-evaluation
description: Use when evaluating m6A per-read predictions from ONT nanopore eventalign
  data and you need to restrict classification to reads above an average base-quality
  cutoff before scoring per-position accuracy against reference labels.
when_to_use_negative:
- Input reads are basecalled by a basecaller that does not save event data in the
  fast5 files, since nanopolish eventalign (the event source for mCaller) cannot run
- Data are from flowcells other than R9.4/R9.5 E. coli-trained models and no retrained
  model or re-derived quality cutoff is available
- The goal is motif-level per-position methylation summary without per-read filtering;
  use make_bed.py summary output instead
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: mCaller
  role: Runs the neural-network m6A classification on eventalign data; the -q/qual_thresh
    flag restricts classification to reads whose average base quality exceeds the
    cutoff
  repo: github.com/al-mcintyre/mCaller
- name: biopython
  role: Parses the fastq file to compute each read's average base quality used for
    thresholding
- name: nanopolish
  role: Produces the eventalign TSV (scaled event values) that mCaller consumes as
    per-read input features
  repo: https://github.com/jts/nanopolish
- name: scikit-learn
  role: Implements the neural network classifier behind the mCaller model pickle evaluated
    here
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-08289-9
    title: Single-molecule sequencing detection of N6-methyladenine in microbial reference
      materials
  source_content_hash: sha256:65fdf9a5aa199b4d38049296038235d02e6623921dbdfe6142e846c18f3e6a88
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - biopython
  - mCaller
  - nanopolish
  - numpy
  - pandas
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# methylation-call-accuracy-evaluation

## Summary

Evaluate m6A methylation calls from mCaller by re-running classification with a per-read average base-quality cutoff (-q/qual_thresh) so only high-quality reads are classified, then scoring per-position accuracy against orthogonal (PacBio-derived) methylation labels; in the source paper, restricting to reads with average base quality > 9 for R9 E. coli data yielded ~84.2% per-position accuracy.

## When to use

Use when evaluating m6A per-read predictions from ONT nanopore eventalign data and you need to restrict classification to reads above an average base-quality cutoff before scoring per-position accuracy against reference labels.

## When NOT to use

- Input reads are basecalled by a basecaller that does not save event data in the fast5 files, since nanopolish eventalign (the event source for mCaller) cannot run
- Data are from flowcells other than R9.4/R9.5 E. coli-trained models and no retrained model or re-derived quality cutoff is available
- The goal is motif-level per-position methylation summary without per-read filtering; use make_bed.py summary output instead

## Inputs

- Nanopore FASTQ file with per-base quality scores (e.g. testdata/masonread1.fastq)
- nanopolish eventalign TSV with scaled event values (e.g. testdata/masonread1.eventalign.tsv)
- Reference assembly FASTA (e.g. testdata/pb_ecoli_polished_assembly.fasta)
- Positions file with chromosome, position, strand, and PacBio-derived methylation labels (e.g. testdata/test_positions_m6A.txt)
- Trained mCaller model pickle (e.g. r95_twobase_model_NN_6_m6A.pkl)

## Outputs

- Quality-filtered per-read m6A predictions (testdata/masonread1.eventalign.diffs.6) with chromosome, read name, genomic position, 11-mer k-mer context, features, strand, label, and methylation probability
- Per-position accuracy score against PacBio-derived labels (expected ~84.2% for R9 data with quality > 9 filter)

## How to apply

Parse the fastq (e.g. testdata/masonread1.fastq) with Biopython to compute each read's average base quality. Run mCaller.py baseline without -q (e.g. -p positions file -r reference fasta -e nanopolish eventalign.tsv -d r95_twobase_model_NN_6_m6A.pkl -f fastq -b A) to obtain unfiltered per-read NN predictions over the 11-mer context (6 variables). Re-run mCaller.py with identical arguments plus -q set to the R9 read-quality cutoff (reads with average base quality > 9 for R9 data), so only reads exceeding the threshold are classified. Score per-position accuracy of the filtered calls against PacBio-derived labels in the positions file, checking that high-quality-read accuracy reproduces the reported ~84.2% for R9 data, and write filtered per-read predictions (chromosome, read name, genomic position, k-mer context, features, strand, label, methylation probability) to testdata/masonread1.eventalign.diffs.6.

## Related tools

- **mCaller** (Runs the neural-network m6A classification on eventalign data; the -q/qual_thresh flag restricts classification to reads whose average base quality exceeds the cutoff) — github.com/al-mcintyre/mCaller
- **biopython** (Parses the fastq file to compute each read's average base quality used for thresholding)
- **nanopolish** (Produces the eventalign TSV (scaled event values) that mCaller consumes as per-read input features) — https://github.com/jts/nanopolish
- **scikit-learn** (Implements the neural network classifier behind the mCaller model pickle evaluated here)

## Examples

```
mCaller.py -p testdata/test_positions_m6A.txt -r testdata/pb_ecoli_polished_assembly.fasta -e testdata/masonread1.eventalign.tsv -d r95_twobase_model_NN_6_m6A.pkl -f testdata/masonread1.fastq -b A -q 9
```

## Evaluation signals

- With -q 9 on R9 E. coli data, per-position accuracy against PacBio-derived labels should reproduce the reported ~84.2%
- Number of classified reads should drop between the unfiltered baseline run and the -q run, confirming the filter engaged
- The output diffs file must contain the expected columns: chromosome, read name, genomic position, k-mer context, features, strand, label, methylation probability
- Every retained read in the filtered output should have an average base quality strictly greater than the qual_thresh value
- Baseline run without -q should succeed, confirming the same eventalign/model inputs are otherwise valid

## Limitations

- Training biases can decrease mCaller accuracy at sequence contexts with lower similarity to the training examples
- mCaller showed variable performance across motifs (60–98% per-position accuracy) with notable failure at one S. aureus motif, indicating incomplete motif coverage
- The qual_thresh default is none, so without an explicit -q the model uses reads at all quality levels, which can degrade per-position accuracy

## Evidence

- [methods] qual_thresh filter flag: "-q, --qual_thresh
 quality threshold for reads (default none)"
- [methods] mCaller invocation: "mCaller.py <-m GATC or -p positions.txt> -r <reference>.fasta -d r95_twobase_model_NN_6_m6A.pkl -e <filename>.eventalign.tsv -f <filename>.fastq -b A"
- [discussion] mCaller per-position accuracy range: "mCaller showed 60–98% per-position accuracy at most motifs, with notable failure at one S. aureus motif"
- [readme] mCaller purpose: "This program is designed to call m6A from nanopore data using the differences between measured and expected currents."
- [discussion] training-bias limitation: "For mCaller, training biases could decrease accuracy at contexts with lower similarity to the training examples."
