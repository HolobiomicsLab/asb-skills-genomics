---
name: nanopore-current-rescaling-and-event-assignment
description: Use when raw ONT fastq/fast5 reads (basecalled with Albacore or another
  event-preserving basecaller) must be aligned and event-level, model-scaled current
  values extracted before per-read methylation classification with mCaller.
when_to_use_negative:
- Input is R10.4 flowcell data, which nanopolish does not currently support for signal-level
  analysis
- Downstream tooling (e.g. Tombo with its alternative model) consumes reads and BAMs
  directly and does not need the eventalign TSV
- Basecaller did not save event data, since eventalign requires event-level information
  from the fast5 files
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: nanopolish
  role: Indexes fastq against fast5 (nanopolish index) and performs model-scaled event
    alignment (eventalign --scale-events -n) to produce the event TSV
  repo: https://github.com/jts/nanopolish
- name: bwa mem
  role: Aligns ONT fastq reads to the reference assembly with the ont2d model to create
    the BAM consumed by eventalign
- name: samtools
  role: Sorts and indexes the bwa mem output BAM prior to eventalign
- name: graphmap
  role: Alternative aligner (graphmap align -r -d -o) that has been tested with mCaller
    for producing alignments
- name: mCaller
  role: 'Downstream consumer: classifies m6A from deviations between measured and
    expected currents in the eventalign TSV using r95_twobase_model_NN_6_m6A.pkl'
  repo: github.com/al-mcintyre/mCaller
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-08289-9
    title: Single-molecule sequencing detection of N6-methyladenine in microbial reference
      materials
  source_content_hash: sha256:2e84831b4309155124c85922ede3be3ceade8c2f7b92e2d1a1c8fe2bbc583eb3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jts/nanopolish
  tools:
  - Tombo
  - bwa mem
  - graphmap
  - mCaller
  - nanopolish
  - samtools
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 16
---

# nanopore-current-rescaling-and-event-assignment

## Summary

Rescale ONT nanopore measured current levels toward the pore model and assign sequenced k-mers to signal events with nanopolish eventalign, producing a TSV of scaled event values that mCaller consumes to detect m6A from measured-vs-expected current deviations.

## When to use

Use when raw ONT fastq/fast5 reads (basecalled with Albacore or another event-preserving basecaller) must be aligned and event-level, model-scaled current values extracted before per-read methylation classification with mCaller.

## When NOT to use

- Input is R10.4 flowcell data, which nanopolish does not currently support for signal-level analysis
- Downstream tooling (e.g. Tombo with its alternative model) consumes reads and BAMs directly and does not need the eventalign TSV
- Basecaller did not save event data, since eventalign requires event-level information from the fast5 files

## Inputs

- ONT nanopore fastq reads (basecalled with Albacore or another basecaller that saves event data)
- fast5 files containing raw signal/event data
- sequencing summary TSV
- reference genome FASTA
- sorted, indexed BAM alignment produced with bwa mem (-x ont2d) or graphmap

## Outputs

- nanopolish eventalign TSV with model-scaled event current values per read and reference position
- indexed fastq/fast5 read database from nanopolish index

## How to apply

First index the ONT fastq against the fast5 directory with `nanopolish index -d <fast5 dir> -s sequencing_summary.txt <fastq>` to link reads to signal data. Align reads to the reference fasta with `bwa mem -x ont2d` (graphmap also works), then sort and index the BAM with samtools. Run `nanopolish eventalign --scale-events -n -r <fastq> -b <sorted.bam> -g <reference.fasta>` so event current values are scaled toward the basecaller model; the rationale is that mCaller's classifiers score deviations between measured and expected currents, so raw unscaled events are unusable. The output is a TSV of per-read, per-position event values that mCaller.py consumes via its -e argument alongside the r95_twobase_model_NN_6_m6A.pkl model. Verify eventalign completed for all reads and that event TSV rows map cleanly onto reference positions before downstream classification.

## Related tools

- **nanopolish** (Indexes fastq against fast5 (nanopolish index) and performs model-scaled event alignment (eventalign --scale-events -n) to produce the event TSV) — https://github.com/jts/nanopolish
- **bwa mem** (Aligns ONT fastq reads to the reference assembly with the ont2d model to create the BAM consumed by eventalign)
- **samtools** (Sorts and indexes the bwa mem output BAM prior to eventalign)
- **graphmap** (Alternative aligner (graphmap align -r -d -o) that has been tested with mCaller for producing alignments)
- **mCaller** (Downstream consumer: classifies m6A from deviations between measured and expected currents in the eventalign TSV using r95_twobase_model_NN_6_m6A.pkl) — github.com/al-mcintyre/mCaller

## Examples

```
nanopolish eventalign -t 8 --scale-events -n -r reads.fastq -b reads.sorted.bam -g reference.fasta > reads.eventalign.tsv
```

## Evaluation signals

- nanopolish index reports a summary file (sequencing_summary.txt) successfully linking fastq read IDs to fast5 files
- BAM is sorted and indexed (samtools sort / index) and alignment rate to the E. coli MG1655 K12 reference is consistent with ONT data quality
- eventalign TSV contains --scale-events-adjusted current values for reads covering the target motif positions (e.g. GATC sites)
- mCaller run with -e <eventalign.tsv> and -d r95_twobase_model_NN_6_m6A.pkl produces per-read methylation probabilities without dropped reads from missing events

## Limitations

- Scaling and event assignment depend on the basecaller having saved event data; basecallers that discard events are incompatible
- Event-level accuracy degrades at sequence contexts dissimilar to training examples, since training biases of the downstream model decrease accuracy at low-similarity contexts
- Presently nanopolish does not support R10.4 flowcells for signal-level analysis

## Evidence

- [methods] nanopolish index -d <fast5 directory> -s sequencing_summary.txt <filename>.fastq: "nanopolish index -d <fast5 directory> -s sequencing_summary.txt <filename>.fastq"
- [methods] eventalign scaled events: "nanopolish eventalign -t <num_threads> --scale-events -n -r <filename>.fastq -b <filename>.sorted.bam -g <reference>.fasta"
- [methods] bwa mem ont2d alignment: "bwa mem -x ont2d -t <num_threads> <reference>.fasta <filename>.fastq | samtools view -Sb - | samtools sort"
- [readme] readme basecaller requirement: "nanopore sequencing data (fastq format + fast5 to run nanopolish, basecalled using Albacore or another basecaller that saves event data)"
- [abstract] current deviation rationale: "using deviations between measured and expected current values as each adenine travels through a pore"
- [readme] R10 limitation: [evidence span withheld — anchor status: external_doc]
