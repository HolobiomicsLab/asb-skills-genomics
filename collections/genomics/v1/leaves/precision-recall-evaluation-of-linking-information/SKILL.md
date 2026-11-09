---
name: precision-recall-evaluation-of-linking-information
description: Use when comparing aligners or alignment-free linking methods on simulated
  reads where the true read-to-contig links are known, and you must report precision,
  recall, and runtime per tool.
when_to_use_negative:
- Real reads with unknown truth — precision/recall of linking information cannot be
  computed without simulated ground truth
- Evaluation goals unrelated to scaffolding-graph linking information (e.g. base-level
  alignment accuracy of alignments themselves)
- Long-read aligner benchmarking — this protocol targets short-read aligners (Bowtie,
  Bowtie2, Bwa, Bwa-Mem) and Fast-SG
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3168
tools:
- name: Fast-SG
  role: alignment-free method under test that constructs the scaffolding graph and
    whose linking information is evaluated
  repo: https://github.com/adigenova/fast-sg
- name: Bowtie
  role: short-read aligner baseline whose links are compared for precision/recall/runtime
- name: Bowtie2
  role: short-read aligner baseline run in global and local mode
- name: Bwa
  role: short-read aligner baseline
- name: Bwa-Mem
  role: short-read aligner baseline and precision leader (99.97%)
- name: KMC
  role: k-mer counter whose binaries (kmc, kmc_dump) are placed in Fast-SG/KMC/bin
    to support Fast-SG runs
  repo: https://github.com/refresh-bio/KMC
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giy048
    title: 'Fast-SG: an alignment-free algorithm for hybrid assembly'
  source_content_hash: sha256:185159b08290e2f9f4e6cb20a43ec134c9403c0b21b43614ff35f6ae4a7b0323
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/adigenova/fast-sg
  tools:
  - Bowtie
  - Bowtie2
  - Bwa
  - Bwa-Mem
  - Fast-SG
  - KMC
  - PERL
  - g++
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# precision-recall-evaluation-of-linking-information

## Summary

Benchmark protocol that quantifies alignment precision and recall of read-to-contig linking information (as used for scaffolding graphs) against truth from simulated reads, alongside wall-clock runtime, across competing aligners and k-mer sizes.

## When to use

Use when comparing aligners or alignment-free linking methods on simulated reads where the true read-to-contig links are known, and you must report precision, recall, and runtime per tool.

## When NOT to use

- Real reads with unknown truth — precision/recall of linking information cannot be computed without simulated ground truth
- Evaluation goals unrelated to scaffolding-graph linking information (e.g. base-level alignment accuracy of alignments themselves)
- Long-read aligner benchmarking — this protocol targets short-read aligners (Bowtie, Bowtie2, Bwa, Bwa-Mem) and Fast-SG

## Inputs

- Simulated paired-end Illumina read libraries from a reference genome (e.g. GRCh38.p10) with known true linking information
- Fast-SG read configuration file ('#type libID Path(fwd) Path(rev) SAM(1:single 2:paired)' format)
- Contig/reference sequences in FASTA format (e.g. example/ecoli-illumina.fa.gz)

## Outputs

- aligner_benchmark_table.csv with precision, recall, and runtime for every tool × k-mer combination
- Per-tool, per-k read-to-contig alignments feeding the scaffolding graph

## How to apply

Simulate paired-end Illumina reads from a reference genome (e.g. GRCh38.p10) so that the true read-to-contig links are known, and describe the libraries in a Fast-SG read configuration file using the '#type libID Path(fwd) Path(rev) SAM(1:single 2:paired)' format. Run each competitor (Fast-SG via FAST-SG.pl once per k-mer size in [12-256], Bowtie, Bowtie2 in global and local mode, Bwa, Bwa-Mem) on the identical simulated read sets and contigs to construct equivalent read-to-contig alignments for the scaffolding graph. For each tool and k-mer size, evaluate precision and recall of the recovered linking information against the simulation truth and record wall-clock runtime. Aggregate the three metrics per tool × k-mer into aligner_benchmark_table.csv; in the Fast-SG paper this produced precision ≥99.21% for Fast-SG (Bwa-Mem 99.97%, Bowtie2-local 98.17%) and recall of 71.67% at k=25-30 versus 71.52% for Bowtie, with Fast-SG 7X-14X faster than the next fastest aligner.

## Related tools

- **Fast-SG** (alignment-free method under test that constructs the scaffolding graph and whose linking information is evaluated) — https://github.com/adigenova/fast-sg
- **Bowtie** (short-read aligner baseline whose links are compared for precision/recall/runtime)
- **Bowtie2** (short-read aligner baseline run in global and local mode)
- **Bwa** (short-read aligner baseline)
- **Bwa-Mem** (short-read aligner baseline and precision leader (99.97%))
- **KMC** (k-mer counter whose binaries (kmc, kmc_dump) are placed in Fast-SG/KMC/bin to support Fast-SG runs) — https://github.com/refresh-bio/KMC

## Examples

```
./FAST-SG.pl -k 25-30 -l reads.txt -r contigs.fa.gz -p sim_benchmark
```

## Evaluation signals

- aligner_benchmark_table.csv is complete: every tool × k-mer combination present with precision, recall, and runtime values
- Results are consistent with the paper's qualitative finding that Fast-SG outperforms state-of-the-art short-read aligners when building the scaffolding graph
- Fast-SG precision stays above 99% for any k-mer size (minimum 99.21%) and recall reaches ~71.67% at k=25-30, comparable to Bowtie (71.52%)
- Runtime records show Fast-SG 7X-14X faster than the next fastest aligner on identical simulated read sets
- All aligners were run on the identical simulated read sets and contigs, so metric differences are attributable to the method, not the input

## Limitations

- Evaluation depends on simulated reads; conclusions rest on simulation fidelity (Illumina reads from GRCh38.p10)
- Fast-SG recall at evaluated k-mer sizes (~71.67%) leaves a substantial fraction of true links unrecovered, comparable to Bowtie but below the precision leaders
- Fast-SG k-mer size is restricted to [12-256]; runtime must be measured once per k-mer size since FAST-SG.pl is invoked separately per k

## Evidence

- [results] precision-recall protocol on simulated human Illumina data: [evidence span withheld — anchor status: not_found]
- [abstract] Fast-SG outperforms aligners for scaffolding graph: "Fast-SG outperforms the state-of-the-art short-read aligners when building the scaffoldinggraph"
- [readme] identical inputs across aligners: "./FAST-SG.pl  -k 15 -l example/ecoli-reads.txt -r example/ecoli-illumina.fa.gz -p test"
- [methods] range of k-mers support: "Using a range of k-mers
	./FAST-SG.pl  -k 15-25:5 -l example/ecoli-reads.txt -r example/ecoli-illumina.fa.gz -p test"
