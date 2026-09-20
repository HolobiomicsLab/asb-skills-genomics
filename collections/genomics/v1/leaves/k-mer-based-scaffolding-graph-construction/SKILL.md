---
name: k-mer-based-scaffolding-graph-construction
description: Use when you have Illumina assembly contigs plus raw or hybrid-corrected
  long reads (ONT R9.2 or PacBio Sequel) and need alignment-free linking information
  / synthetic mate-pair libraries for hybrid scaffolding.
when_to_use_negative:
- You already have high-quality long-read alignments to contigs and a standard alignment-based
  long-read scaffolder (e.g. Links) is appropriate — Fast-SG is designed as an alignment-free
  alternative, not to replace direct alignment when alignments are cheap and reliable
- Synthetic mate pairs with insert sizes larger than the actual long-read length are
  required — the insert size is restricted to the actual long-read size
- Only short reads are available with no long-read data, since the synthetic mate-pair
  extraction relies on linking information from long reads
edam_operation: http://edamontology.org/operation_0558
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: Fast-SG
  role: Alignment-free algorithm that builds the scaffolding graph and extracts synthetic
    mate-pair libraries; invoked via the FAST-SG.pl PERL wrapper
  repo: https://github.com/adigenova/fast-sg
- name: KMC
  role: Disk-based k-mer counter whose binaries must be placed inside the Fast-SG
    directory to support k-mer counting for graph construction
  repo: https://github.com/refresh-bio/KMC
- name: ntHash
  role: Rolling hash function library used internally by Fast-SG for k-mer hashing
  repo: https://github.com/bcgsc/ntHash
- name: BBHash
  role: Minimal perfect hash library used inside Fast-SG to index k-mer sets with
    light-weight data structures
  repo: https://github.com/rizkg/BBHash
- name: quasi_dictionary
  role: Probabilistic key-value index (fingerprint-based, false positives below 1/2^fingerprint_size)
    used by Fast-SG for k-mer lookup
  repo: https://github.com/pierrepeterlongo/quasi_dictionary
- name: PERL
  role: Language of the FAST-SG.pl wrapper that orchestrates k-mer counting, graph
    construction, and library extraction
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giy048
    title: 'Fast-SG: an alignment-free algorithm for hybrid assembly'
  source_content_hash: sha256:a3fb0f27f5450eddf61923580365479d82d29cba38bb5b6c594d80723a7289d6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/adigenova/fast-sg
  tools:
  - BBHash
  - Fast-SG
  - KMC
  - PERL
  - ntHash
  - quasi_dictionary
  edam_operation: http://edamontology.org/operation_0558
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# k-mer-based-scaffolding-graph-construction

## Summary

Build an ultrafast alignment-free scaffolding graph from short or long reads using k-mer counting and light-weight data structures (Fast-SG with KMC3), enabling extraction of synthetic mate-pair libraries from raw or error-corrected long reads to scaffold an Illumina assembly.

## When to use

Use when you have Illumina assembly contigs plus raw or hybrid-corrected long reads (ONT R9.2 or PacBio Sequel) and need alignment-free linking information / synthetic mate-pair libraries for hybrid scaffolding.

## When NOT to use

- You already have high-quality long-read alignments to contigs and a standard alignment-based long-read scaffolder (e.g. Links) is appropriate — Fast-SG is designed as an alignment-free alternative, not to replace direct alignment when alignments are cheap and reliable
- Synthetic mate pairs with insert sizes larger than the actual long-read length are required — the insert size is restricted to the actual long-read size
- Only short reads are available with no long-read data, since the synthetic mate-pair extraction relies on linking information from long reads

## Inputs

- Read configuration file (space-separated format: #type libID Path(long read) Insert-sizes SAM) listing raw or hybrid-corrected ONT R9.2 / PacBio Sequel libraries
- Long-read FASTQ/FASTA files (raw or Lordec error-corrected)
- Illumina assembly contigs in FASTA (optionally gzipped), e.g. ecoli-illumina.fa.gz
- k-mer size or range (e.g. k=15 or k=15-25:5)

## Outputs

- Scaffolding graph built from short or long reads
- Synthetic mate-pair library files extracted from long reads
- Metrics such as k-mer recall, synthetic mate-pair read recall, and outlier pair percentage per library

## How to apply

Author a space-separated read configuration file in the Fast-SG long-read format (#type libID Path(long read) Insert-sizes SAM), listing the long-read libraries with insert sizes spanning e.g. 0.5-8 kb. Clone Fast-SG, place KMC3 binaries inside the Fast-SG directory, compile with 'make all', and validate with 'make test'. Run the FAST-SG.pl wrapper with the alignment-free k-mer size (-k 15, or a range such as -k 15-25:5), the read configuration file (-l), the Illumina contigs in FASTA (-r), and an output prefix (-p); Fast-SG uses KMC k-mer counting with ntHash rolling hashes, BBHash minimal perfect hashing, and a quasi_dictionary to build the scaffolding graph without aligning reads. Extract synthetic mate-pair libraries from the graph and assess k-mer recall and mate-pair read recall against the Illumina assembly. Note that the synthetic library insert size is restricted to the actual long-read size, so recall depends on read accuracy — hybrid error correction (e.g. Lordec) substantially raises recall for raw ONT/PacBio reads.

## Related tools

- **Fast-SG** (Alignment-free algorithm that builds the scaffolding graph and extracts synthetic mate-pair libraries; invoked via the FAST-SG.pl PERL wrapper) — https://github.com/adigenova/fast-sg
- **KMC** (Disk-based k-mer counter whose binaries must be placed inside the Fast-SG directory to support k-mer counting for graph construction) — https://github.com/refresh-bio/KMC
- **ntHash** (Rolling hash function library used internally by Fast-SG for k-mer hashing) — https://github.com/bcgsc/ntHash
- **BBHash** (Minimal perfect hash library used inside Fast-SG to index k-mer sets with light-weight data structures) — https://github.com/rizkg/BBHash
- **quasi_dictionary** (Probabilistic key-value index (fingerprint-based, false positives below 1/2^fingerprint_size) used by Fast-SG for k-mer lookup) — https://github.com/pierrepeterlongo/quasi_dictionary
- **PERL** (Language of the FAST-SG.pl wrapper that orchestrates k-mer counting, graph construction, and library extraction)

## Examples

```
./FAST-SG.pl -k 15 -l example/ecoli-reads.txt -r example/ecoli-illumina.fa.gz -p test
```

## Evaluation signals

- Output metrics table contains a row for each technology/correction-state combination (raw vs corrected ONT R9.2 and PacBio Sequel), with recall and outlier percentages parseable in [0, 100]
- k-mer recall and synthetic mate-pair read recall against the Illumina assembly are computed and reported per library (e.g. mate-pair read recall of 49.42% raw ONT rising to 75.12% after Lordec correction)
- 'make test' small test passes after 'make all' compilation
- Percentage of outlier synthetic pairs per library is calculated and within expected bounds for the insert-size range used (e.g. 0.5-8 kb)
- Number of synthetic read pairs aligned to the reference assembly (e.g. DiscovarDeNovo assembly of E. coli K12) is tabulated per library

## Limitations

- The synthetic library insert size is restricted to the actual long-read size
- Raw long reads substantially reduce recall: synthetic mate-pair read recall drops from 75.12% to 49.42% (ONT) and 65.02% to 31.65% (PacBio) without hybrid error correction
- No version number stated for the Fast-SG software itself (only RRID SCR_015934)
- Gap size estimation with long-range mate-pair technologies like 10X Genomics and DOVETAIL genomics (unknown orientation, ranged insert sizes) is more challenging than with Fast-SG synthetic libraries
- Fast-SG's k-mer-based graph is validated against specific organisms (E. coli K12, S. cerevisiae W303, A. thaliana, NA12878); extreme k values beyond tested ranges may behave differently

## Evidence

- [abstract] Fast-SG is an ultrafast alignment-free method for constructing a scaffolding graph from either short or long reads using light-weight data structures: "uses a new ultrafast alignment-free algorithm specifically designed for constructing a scaffolding graph using light-weight data structures."
- [methods] Run FAST-SG.pl with k=15, read config, contigs, and output prefix: "./FAST-SG.pl  -k 15 -l example/ecoli-reads.txt -r example/ecoli-illumina.fa.gz -p test"
- [abstract] Fast-SG extracts linking information from raw or error-corrected long reads: "can be used to extract linking information from either raw or error-corrected long reads"
- [readme] KMC binaries must be placed inside the Fast-SG directory: "After getting KMC3, put the binaries inside Fast-SG directory"
- [discussion] Synthetic library insert size restricted to actual long-read size: "The insert size is restricted to the actual long-read size."
- [intro] Fast-SG builds scaffolding graph from short or long reads: "Fast-SG is an alignment-free algorithm for ultrafast scaffolding graph construction from short or long reads."
