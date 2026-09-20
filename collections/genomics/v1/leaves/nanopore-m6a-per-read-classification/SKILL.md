---
name: nanopore-m6a-per-read-classification
description: Use when ONT fastq/fast5 reads (Albacore-basecalled, R9/R9.4/R9.5) need
  per-read m6A probability calls at known motif or positions-file sites, using nanopolish
  eventalign TSVs with --scale-events and a trained model like r95_twobase_model_NN_6_m6A.pkl.
when_to_use_negative:
- 'Data from R10.4 flowcells: nanopolish does not support R10.4 variant/methylation
  calling, so the eventalign-based input pipeline is unavailable'
- 'Motifs where both m6A and m4C are possible (e.g. the GATCGVNY motif in S. aureus):
  single-molecule methods fail to distinguish methylation types'
- 'Sequence contexts very dissimilar to the E. coli-trained model contexts: training
  biases can decrease mCaller accuracy at untrained motifs (e.g. it lost to Tombo
  at the untrained E. coli AAGANNNNNCTC motif, 0.71 vs 0.86 PR-AUC)'
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3295
tools:
- name: mCaller
  role: Neural-network classifier producing per-read m6A probabilities from current
    deviations; also provides make_bed.py for per-position BED summaries
  repo: https://github.com/al-mcintyre/mCaller
- name: nanopolish
  role: Indexes fastq/fast5 (nanopolish index) and generates the scaled event-level
    TSV (nanopolish eventalign --scale-events -n) that mCaller consumes
  repo: https://github.com/jts/nanopolish
- name: bwa mem
  role: Aligns ONT fastq reads to the reference fasta with -x ont2d before event alignment;
    output sorted/indexed with samtools
- name: Tombo
  role: Alternative m6A alternative-model scorer run on the same reads/alignments
    for benchmarking per-position performance
- name: scikit-learn
  role: Runtime dependency of mCaller for the pickled classifier model and for computing
    precision-recall curves/AUC
provenance:
  source_task_ids:
  - task_002
  - task_006
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-08289-9
    title: Single-molecule sequencing detection of N6-methyladenine in microbial reference
      materials
  source_content_hash: sha256:66c4cd67c20012702e9758929b1ccfdc7f48c596067fbe08403fa3b661b0fd4e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/al-mcintyre/mCaller
  tools:
  - Tombo
  - biopython
  - bwa mem
  - mCaller
  - nanopolish
  - numpy
  - pandas
  - samtools
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# nanopore-m6a-per-read-classification

## Summary

Per-read m6A classification on Oxford Nanopore data using mCaller, a scikit-learn neural network that scores deviations between measured and expected currents at adenine positions. It complements Tombo and is designed for microbial reference materials with motif-confirmed methylated/unmethylated sites.

## When to use

Use when ONT fastq/fast5 reads (Albacore-basecalled, R9/R9.4/R9.5) need per-read m6A probability calls at known motif or positions-file sites, using nanopolish eventalign TSVs with --scale-events and a trained model like r95_twobase_model_NN_6_m6A.pkl.

## When NOT to use

- Data from R10.4 flowcells: nanopolish does not support R10.4 variant/methylation calling, so the eventalign-based input pipeline is unavailable
- Motifs where both m6A and m4C are possible (e.g. the GATCGVNY motif in S. aureus): single-molecule methods fail to distinguish methylation types
- Sequence contexts very dissimilar to the E. coli-trained model contexts: training biases can decrease mCaller accuracy at untrained motifs (e.g. it lost to Tombo at the untrained E. coli AAGANNNNNCTC motif, 0.71 vs 0.86 PR-AUC)

## Inputs

- ONT fastq plus fast5 files basecalled with Albacore (or another basecaller saving event data)
- nanopolish sequencing_summary.txt
- Reference genome fasta (e.g. E. coli MG1655 K12 assembly)
- nanopolish eventalign TSV produced with --scale-events and -n
- Sorted/indexed BAM from bwa mem -x ont2d (input to eventalign)
- Positions file (space/tab-separated: chromosome, position, strand, label if training) or motif string via -m (e.g. GATC)
- Pickled sklearn model file, e.g. r95_twobase_model_NN_6_m6A.pkl (R9.4/R9.5 models trained on E. coli data)

## Outputs

- Per-read prediction file (e.g. testdata/masonread1.eventalign.diffs.6) with columns: chromosome, read name, genomic position, k-mer context, features, strand, label, methylation probability
- Per-position BED file from make_bed.py with chrom, chromStart, chromEnd, context, % methylated, strand, and depth of coverage
- Optionally per-motif precision-recall AUC comparisons (e.g. pr_auc_comparison.csv) computed with scikit-learn

## How to apply

Index the ONT fastq with nanopolish (fast5 + sequencing_summary.txt), align reads with bwa mem -x ont2d (or graphmap), and sort/index the BAM with samtools. Run nanopolish eventalign with --scale-events and -n to produce a TSV of event values scaled toward the model. Run mCaller.py with -r reference fasta, -e the eventalign TSV, -f the fastq, -d r95_twobase_model_NN_6_m6A.pkl, -b A, and either -m MOTIF (e.g. GATC) or -p positions.txt (chromosome, position, strand, label if training); the default 6 variables corresponds to an 11-mer sequence context. Optionally apply the -s skip_thresh (default 0), -q qual_thresh (e.g. average base quality > 9 for the NN_hq variant), -d minimum read depth (e.g. 15), and -t mod_threshold (e.g. 0.5) filters. Finally run make_bed.py to summarize per-read calls into per-position BED records (chrom, start, end, context, % methylated, strand, depth).

## Related tools

- **mCaller** (Neural-network classifier producing per-read m6A probabilities from current deviations; also provides make_bed.py for per-position BED summaries) — https://github.com/al-mcintyre/mCaller
- **nanopolish** (Indexes fastq/fast5 (nanopolish index) and generates the scaled event-level TSV (nanopolish eventalign --scale-events -n) that mCaller consumes) — https://github.com/jts/nanopolish
- **bwa mem** (Aligns ONT fastq reads to the reference fasta with -x ont2d before event alignment; output sorted/indexed with samtools)
- **Tombo** (Alternative m6A alternative-model scorer run on the same reads/alignments for benchmarking per-position performance)
- **scikit-learn** (Runtime dependency of mCaller for the pickled classifier model and for computing precision-recall curves/AUC)

## Examples

```
mCaller.py -p testdata/test_positions_m6A.txt -r testdata/pb_ecoli_polished_assembly.fasta -e testdata/masonread1.eventalign.tsv -d r95_twobase_model_NN_6_m6A.pkl -f testdata/masonread1.fastq -b A
```

## Evaluation signals

- At >= 15x read depth (-d 15) with mod_threshold 0.5, per-position accuracy reaches 95.4% and AUC 0.99 on the test positions
- With the -q quality filter (average base quality > 9, NN_hq variant) on R9 E. coli data, per-position accuracy against PacBio-derived labels is ~84.2%
- Per-motif PR-AUC versus Tombo matches the published comparison: GATC 0.91 vs 0.83, AAGANNNNNCTC 0.71 vs 0.86, GAGNNNNNTCTT 0.90 vs 0.64, using P. aeruginosa unmethylated motif sites as true negatives
- The .diffs.6 output contains all documented columns (chromosome, read name, genomic position, k-mer context, features, strand, label, methylation probability) with probabilities in [0, 1]
- Every retained row in the make_bed.py BED has depth-of-coverage >= the -d threshold and 7 documented columns

## Limitations

- Trained contexts limit generalization: mCaller showed 60-98% per-position accuracy at most motifs but notable failure at one S. aureus motif, and training biases could decrease accuracy at contexts with lower similarity to training examples
- Both mCaller and Tombo can fail to distinguish methylation types at motifs like GATCGVNY in S. aureus, and predicted lower methylation than combined PacBio + MeDIP-seq predictions
- Defaults are permissive: -s skip_thresh defaults to 0 and -q qual_thresh defaults to none, so without explicit thresholds low-quality or skip-rich observations are included; nanopore m6A accuracy is inherently lower (~70% per-read range) than m5C

## Evidence

- [methods] mCaller.py <-m GATC or -p positions.txt> -r <reference>.fasta -d r95_twobase_model_NN_6_m6A.pkl -e <filename>.eventalign.tsv -f <filename>.fastq -b A: "mCaller.py <-m GATC or -p positions.txt> -r <reference>.fasta -d r95_twobase_model_NN_6_m6A.pkl -e <filename>.eventalign.tsv -f <filename>.fastq -b A"
- [methods] nanopolish eventalign scaled TSV command: "nanopolish eventalign -t <num_threads> --scale-events -n -r <filename>.fastq -b <filename>.sorted.bam -g <reference>.fasta"
- [methods] minimum read depth filter: "`-d` indicates the minimum read depth for inclusion"
- [readme] README description of mCaller purpose: "This program is designed to call m6A from nanopore data using the differences between measured and expected currents."
- [discussion] nanopore m6A accuracy limitation: "nanopore sequencing can detect m5C with per-read accuracy of upwards of 80% but m6A with lower accuracy (~70%)"
- [discussion] training bias limitation: "For mCaller, training biases could decrease accuracy at contexts with lower similarity to the training examples."
