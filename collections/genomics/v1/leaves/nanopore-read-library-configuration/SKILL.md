---
name: nanopore-read-library-configuration
description: Use when preparing Fast-SG hybrid scaffolding or assembly runs on raw
  Nanopore/PacBio long-read FASTA libraries, e.g. shallow ~5X NA12878 ultra-long reads
  feeding a scaffolding graph built with k-mer size 22.
when_to_use_negative:
- The input libraries are Illumina paired-end short reads described as separate fwd/rev
  paths — use the short-read config format '#type libID Path(fwd) Path(rev) SAM(1:single
  2:paired)' instead of the long-read 'long' line format
- Requested insert sizes exceed the actual long-read length, since Fast-SG restricts
  synthetic insert size to the real read size
- The scaffolding graph will be built by a read-to-contig aligner (Bowtie/Bowtie2/Bwa-Mem)
  rather than Fast-SG, which does not consume this configuration file
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: Fast-SG
  role: Consumes the long-read configuration file through the FAST-SG.pl wrapper (-l
    argument) to extract alignment-free synthetic mate-pair libraries from Nanopore
    reads
  repo: https://github.com/adigenova/fast-sg
- name: KMC
  role: KMC3 k-mer counter binaries (kmc, kmc_dump) must be placed in Fast-SG/KMC/bin/
    so Fast-SG can process the configured long-read libraries
  repo: https://github.com/refresh-bio/KMC
- name: PERL
  role: Language of the FAST-SG.pl wrapper that parses the read configuration file
provenance:
  source_task_ids:
  - task_005
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giy048
    title: 'Fast-SG: an alignment-free algorithm for hybrid assembly'
  source_content_hash: sha256:e6224eef1ee782a876dfe5d2ff237e756c2bae97282c5e96f3a793abc387e9c1
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
  - ScaffMatch
  - g++
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# nanopore-read-library-configuration

## Summary

Writes the Fast-SG read configuration file that describes Nanopore (or other long-read) libraries — one space-separated line per library with type 'long', library ID, FASTA path, comma-separated insert sizes, and the single-end flag — so FAST-SG.pl can extract alignment-free synthetic mate-pair libraries from uncorrected long reads.

## When to use

Use when preparing Fast-SG hybrid scaffolding or assembly runs on raw Nanopore/PacBio long-read FASTA libraries, e.g. shallow ~5X NA12878 ultra-long reads feeding a scaffolding graph built with k-mer size 22.

## When NOT to use

- The input libraries are Illumina paired-end short reads described as separate fwd/rev paths — use the short-read config format '#type libID Path(fwd) Path(rev) SAM(1:single 2:paired)' instead of the long-read 'long' line format
- Requested insert sizes exceed the actual long-read length, since Fast-SG restricts synthetic insert size to the real read size
- The scaffolding graph will be built by a read-to-contig aligner (Bowtie/Bowtie2/Bwa-Mem) rather than Fast-SG, which does not consume this configuration file

## Inputs

- Uncorrected Nanopore long-read FASTA library (e.g. ultra-long NA12878 Nanopore reads, ~5X coverage)
- Library metadata: library ID, read type ('long'), single-end flag, and desired insert-size list (comma-separated)
- Fast-SG examples/ecoli-reads.txt as the format template

## Outputs

- Read configuration text file consumable by FAST-SG.pl via -l
- Downstream: synthetic mate-pair libraries per insert size with library_stats.tsv (pair counts, linking percentage, outlier percentage per library)

## How to apply

Create a plain-text configuration file listing each long-read library on its own space-separated line: the type token 'long', a library ID, the path to the long-read FASTA, a comma-separated list of candidate insert sizes, and the single-end flag — following the examples/ecoli-reads.txt format shipped with Fast-SG. Choose insert sizes within the range up to BAC-like sizes (150–180 kb) for ultra-long Nanopore reads, remembering that the effective insert size is bounded by the actual long-read length, so a 5X-coverage Nanopore library supports multiple synthetic libraries but never insert sizes longer than its reads. Pass the file to the FAST-SG.pl PERL wrapper via the mandatory -l argument together with -k (k-mer size, e.g. 22), -r (contig FASTA such as the NA12878 DiscovarDeNovo assembly) and -p (output prefix); Fast-SG then uses KMC k-mer counting to extract synthetic mate-pair libraries alignment-free. Re-running extraction across the insert-size range yields one synthetic library per insert size, each summarised in library_stats.tsv.

## Related tools

- **Fast-SG** (Consumes the long-read configuration file through the FAST-SG.pl wrapper (-l argument) to extract alignment-free synthetic mate-pair libraries from Nanopore reads) — https://github.com/adigenova/fast-sg
- **KMC** (KMC3 k-mer counter binaries (kmc, kmc_dump) must be placed in Fast-SG/KMC/bin/ so Fast-SG can process the configured long-read libraries) — https://github.com/refresh-bio/KMC
- **PERL** (Language of the FAST-SG.pl wrapper that parses the read configuration file)

## Examples

```
./FAST-SG.pl -k 22 -l NA12878-nanopore-reads.txt -r NA12878-DiscovarDeNovo-contigs.fa -p NA12878_dfs
```

## Evaluation signals

- Each non-comment line of the configuration file has the expected space-separated fields: type 'long', library ID, FASTA path, comma-separated insert sizes, and the single-end flag
- All FASTA paths in the configuration resolve to existing, valid FASTA long-read files
- Every listed insert size does not exceed the actual long-read length (Fast-SG bounds insert size to read length)
- Running FAST-SG.pl with the -l file produces one synthetic library per insert size (e.g. 20 libraries up to 180 kb for NA12878) and a library_stats.tsv with pair counts, linking percentage, and outlier percentage for every library

## Limitations

- The insert size of synthetic libraries is restricted to the actual long-read size, so shallow or short Nanopore libraries cannot support BAC-like (150–180 kb) insert sizes
- The configuration format is Fast-SG-specific and was only exercised in the paper on uncorrected/corrected PacBio and Nanopore long reads and simulated Illumina reads
- Long-read libraries are declared with the single-end flag; the SAM paired/single numeric flags (1:single 2:paired) described for the short-read format do not apply to 'long' lines

## Evidence

- [methods] Example of a read configuration file can be found in examples/ecoli-reads.txt: "Example of a read configuration file can be found in examples/ecoli-reads.txt"
- [discussion] Fast-SG synthetic libraries from long reads: "The insert size is restricted to the actual long-read size."
- [intro] Fast-SG graph construction from short or long reads: "Fast-SG is an alignment-free algorithm for ultrafast scaffolding graph construction from short or long reads."
