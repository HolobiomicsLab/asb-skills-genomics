---
name: k-mer-size-sweep-execution
description: Use when you need to determine which k-mer size(s) yield the scaffolding
  graph linking information from a read set, or to benchmark aligners/scaffolding-graph
  construction across multiple k values on the same reads and contigs.
when_to_use_negative:
- When you need full base-level read alignments rather than alignment-free linking
  information — Bowtie2 or Bwa-Mem give higher precision (e.g., Bwa-Mem 99.97% vs
  Fast-SG minimum 99.21%) but are 7X-14X slower than Fast-SG.
- When input reads cannot be described as FASTA/FASTQ libraries with a known pairing
  configuration (single vs paired) expected by the '#type libID Path(fwd) Path(rev)
  SAM' format.
- When the k-mer size you want exceeds 256 or is below 12 — Fast-SG's -k argument
  is restricted to [12-256].
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3174
tools:
- name: Fast-SG
  role: Alignment-free scaffolding graph construction executed once per k-mer size
    via FAST-SG.pl
  repo: https://github.com/adigenova/fast-sg
- name: KMC
  role: k-mer counter whose binaries (kmc, kmc_dump) must be placed inside Fast-SG/KMC/bin
    before compiling Fast-SG
  repo: https://github.com/refresh-bio/KMC
- name: PERL
  role: Driver language for FAST-SG.pl wrapper that runs the k-mer sweep
- name: g++
  role: Compiler used to build Fast-SG from source (tested with g++ 5.3 on Linux)
- name: Bowtie2
  role: Reference short-read aligner run on the same read sets to compare precision/recall/runtime
    per k
- name: Bwa-Mem
  role: Reference aligner run on identical simulated read sets for the benchmark comparison
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giy048
    title: 'Fast-SG: an alignment-free algorithm for hybrid assembly'
  source_content_hash: sha256:8ddc067ce02a03dc5e523cfc224a548869fc7b1522e24a94e5e8316eb02dc0fd
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
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# k-mer-size-sweep-execution

## Summary

Runs Fast-SG repeatedly over a range of k-mer sizes (k in [12-256], supporting range syntax such as 15-25:5) to construct scaffolding graphs from short or long reads, so that precision, recall, and runtime can be compared across k values.

## When to use

Use when you need to determine which k-mer size(s) yield the best scaffolding graph linking information from a read set, or to benchmark aligners/scaffolding-graph construction across multiple k values on the same reads and contigs.

## When NOT to use

- When you need full base-level read alignments rather than alignment-free linking information — Bowtie2 or Bwa-Mem give higher precision (e.g., Bwa-Mem 99.97% vs Fast-SG minimum 99.21%) but are 7X-14X slower than Fast-SG.
- When input reads cannot be described as FASTA/FASTQ libraries with a known pairing configuration (single vs paired) expected by the '#type libID Path(fwd) Path(rev) SAM' format.
- When the k-mer size you want exceeds 256 or is below 12 — Fast-SG's -k argument is restricted to [12-256].

## Inputs

- Read configuration file in the '#type libID Path(fwd) Path(rev) SAM(1:single 2:paired)' format (e.g., examples/ecoli-reads.txt)
- Contig/reference sequences in FASTA format (may be gzipped, e.g., example/ecoli-illumina.fa.gz)
- k-mer size or range specification (integers in [12-256], range syntax such as 15-25:5)
- Output prefix path
- KMC3 binaries (kmc, kmc_dump) placed inside Fast-SG/KMC/bin

## Outputs

- Scaffolding graph / read-to-contig linking information per k-mer size (prefixed with -p)
- Synthetic libraries extracted from reads for scaffolding
- Per-k precision, recall, and runtime values for downstream aggregation (e.g., aligner_benchmark_table.csv)

## How to apply

Describe the read libraries (e.g., simulated paired-end Illumina reads) in a Fast-SG read configuration file using the '#type libID Path(fwd) Path(rev) SAM(1:single 2:paired)' format. Compile Fast-SG from source with g++ (tested with g++ 5.3 on Linux) after placing the KMC3 binaries (kmc, kmc_dump) inside the Fast-SG/KMC/bin directory. Then run FAST-SG.pl with the four mandatory arguments (-k for k-mer size(s) restricted to [12-256] and supporting range syntax such as 15-25:5, -r for contigs in FASTA format, -l for the read configuration file, -p for an output prefix) once per k-mer size, or supply a k-mer range to sweep multiple values in a single command. For each k, the tool builds the scaffolding graph alignment-free using KMC k-mer counting; evaluating precision and recall of the recovered linking information against simulated-read truth per k-mer size (e.g., Fast-SG recall 71.67% at k=25-30) tells you which k to keep.

## Related tools

- **Fast-SG** (Alignment-free scaffolding graph construction executed once per k-mer size via FAST-SG.pl) — https://github.com/adigenova/fast-sg
- **KMC** (k-mer counter whose binaries (kmc, kmc_dump) must be placed inside Fast-SG/KMC/bin before compiling Fast-SG) — https://github.com/refresh-bio/KMC
- **PERL** (Driver language for FAST-SG.pl wrapper that runs the k-mer sweep)
- **g++** (Compiler used to build Fast-SG from source (tested with g++ 5.3 on Linux))
- **Bowtie2** (Reference short-read aligner run on the same read sets to compare precision/recall/runtime per k)
- **Bwa-Mem** (Reference aligner run on identical simulated read sets for the benchmark comparison)

## Examples

```
./FAST-SG.pl -k 15-25:5 -l example/ecoli-reads.txt -r example/ecoli-illumina.fa.gz -p test
```

## Evaluation signals

- FAST-SG.pl runs complete for every k-mer size in the specified range (e.g., 15-25:5) without errors after 'make test' passes
- Each tool x k-mer combination in the benchmark table is present with precision, recall, and runtime values (table completeness check against aligner_benchmark_table.csv)
- Recovered linking-information precision is high for any k (minimum 99.21% on simulated human Illumina data), comparable to Bowtie2-global (99.74%)
- Fast-SG runtime is 7X-14X faster than the next fastest aligner in the comparison
- Recall at the chosen k (e.g., 71.67% at k=25-30) is comparable to Bowtie's 71.52% on the same data

## Limitations

- k-mer size is restricted to the range [12-256] by Fast-SG's -k argument
- Precision of Fast-SG linking information (min 99.21%) does not reach Bwa-Mem's 99.97% or Bowtie2-global's 99.74%; the trade-off is speed (7X-14X faster)
- Compilation was tested only with g++ 5.3 (Linux) and clang 4.2 (Mac OSX); other toolchains may fail
- No version number is stated for the Fast-SG software itself (only RRID SCR_015934) and no changelog is available for the repository

## Evidence

- [results] Fast-SG precision is high for any k-mer size (minimum 99.21%): [evidence span withheld — anchor status: not_found]
- [methods] FAST-SG.pl mandatory arguments: "./FAST-SG.pl  -k 15 -l example/ecoli-reads.txt -r example/ecoli-illumina.fa.gz -p test"
- [methods] k-mer range syntax: "./FAST-SG.pl  -k 15-25:5 -l example/ecoli-reads.txt -r example/ecoli-illumina.fa.gz -p test"
- [discussion] Fast-SG is faster than state-of-the-art aligners: "Fast-SG is faster than the current state-of-the-art short-read aligners"
- [readme] Compilation instructions: "After getting KMC3, put the binaries inside Fast-SG directory"
- [abstract] Alignment-free scaffolding graph construction: "uses a new ultrafast alignment-free algorithm specifically designed for constructing a scaffolding graph using light-weight data structures"
