---
name: outlier-trimming-statistics
description: Use when estimating mate-pair library insert-size mean/SD and read-pair
  orientation from aligned synthetic read pairs where extreme tail values (very small
  or very large inferred inserts) would distort the mean and standard deviation.
when_to_use_negative:
- The library is a real long-range mate-pair technology (e.g. 10X Genomics or DOVETAIL
  genomics) where linking information comes from a range of insert sizes and read-pair
  orientation may be unknown — the paper notes gap-size estimation there is more challenging
  than with Fast-SG synthetic libraries
- The input is already a per-library statistics table — no re-trimming or re-estimation
  is needed
- The long reads are shorter than the requested insert size, since Fast-SG restricts
  the insert size to the actual long-read length and no valid pairs can be extracted
tools:
- name: Fast-SG
  role: Extracts the synthetic forward/reverse mate-pair libraries from long reads
    whose same-contig alignments feed the trimmed insert-size statistics
  repo: https://github.com/adigenova/fast-sg
- name: PERL
  role: Scripting language used for the Fast-SG pipeline driver (FAST-SG.pl) that
    orchestrates library extraction and statistics
- name: C++
  role: Implementation language of the Fast-SG alignment-free scaffolding graph engine
    underlying the synthetic library extraction
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/gigascience/giy048
    title: 'Fast-SG: an alignment-free algorithm for hybrid assembly'
  source_content_hash: sha256:981568f8745fd26500958bc10728f880c9a97445703ff397b2dea7f8ff45bd23
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/adigenova/fast-sg
  tools:
  - C++
  - Fast-SG
  - PERL
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# outlier-trimming-statistics

## Summary

Estimates per-library mate-pair statistics (mean insert size, standard deviation, FR/RF orientation) by computing them from same-contig aligned synthetic read pairs after symmetrically trimming the lowest and highest 10% of the insert-size distribution as outliers, as done in the Fast-SG hybrid assembly workflow.

## When to use

Use when estimating mate-pair library insert-size mean/SD and read-pair orientation from aligned synthetic read pairs where extreme tail values (very small or very large inferred inserts) would distort the mean and standard deviation.

## When NOT to use

- The library is a real long-range mate-pair technology (e.g. 10X Genomics or DOVETAIL genomics) where linking information comes from a range of insert sizes and read-pair orientation may be unknown — the paper notes gap-size estimation there is more challenging than with Fast-SG synthetic libraries
- The input is already a per-library statistics table — no re-trimming or re-estimation is needed
- The long reads are shorter than the requested insert size, since Fast-SG restricts the insert size to the actual long-read length and no valid pairs can be extracted

## Inputs

- Read configuration file (space-separated: type, library ID, path, insert sizes, pairing flag) listing long-read libraries and target insert sizes
- Long-read sequence libraries (e.g. PacBio or Oxford Nanopore FASTQ/FASTA)
- Contig set to which synthetic read pairs are aligned (e.g. DiscovarDeNovo assembly FASTA)
- Alignment records of synthetic read pairs restricted to within-contig mappings

## Outputs

- library_stats.tsv: per-library table with library ID, estimated insert size, standard deviation, and orientation (FR/RF)
- FASTQ files of synthetic forward/reverse 200 bp mate pairs per library

## How to apply

Parse the read configuration file (space-separated: type, library ID, path, insert sizes, pairing flag) to obtain each long-read library and its target insert sizes (e.g. 1000, 2000, 3000, 5000), keeping in mind Fast-SG restricts the insert size to the actual long-read length when extracting synthetic pairs. For each library, slide a 100 bp window along the long read and excise forward/reverse 200 bp read pairs at each specified insert size, emit FASTQ synthetic pairs, then align them back to the contig set. Restrict statistics to pairs mapping within the same contig, since cross-contig pairs carry no intra-library insert-size signal. Rank the same-contig pairs by insert size and trim the lowest 10% and highest 10% of the distribution as outliers before computing the mean and standard deviation — this symmetric two-sided trim removes mapping artifacts and extreme mis-estimates. Infer the read-pair orientation (FR/RF) from the mapping coordinates of the retained pairs. Write one row per library (library ID, estimated insert size, standard deviation, orientation) to library_stats.tsv.

## Related tools

- **Fast-SG** (Extracts the synthetic forward/reverse mate-pair libraries from long reads whose same-contig alignments feed the trimmed insert-size statistics) — https://github.com/adigenova/fast-sg
- **PERL** (Scripting language used for the Fast-SG pipeline driver (FAST-SG.pl) that orchestrates library extraction and statistics)
- **C++** (Implementation language of the Fast-SG alignment-free scaffolding graph engine underlying the synthetic library extraction)

## Examples

```
./FAST-SG.pl -k 15 -l example/ecoli-reads.txt -r example/ecoli-illumina.fa.gz -p test
```

## Evaluation signals

- library_stats.tsv exists and contains exactly one row per library configured in the read configuration file
- Estimated insert sizes are consistent with the user-specified insert sizes (e.g. 1000, 2000, 3000, 5000)
- Each row includes non-empty standard deviation and orientation (FR/RF) fields
- Trimming is symmetric: the lowest 10% and highest 10% of the insert-size distribution are excluded before computing the mean and SD
- Only read pairs mapping within the same contig are included in the statistics

## Limitations

- The insert size is restricted to the actual long-read size, so libraries cannot exceed the length of the source long reads
- The fixed 10% two-sided trim is a heuristic outlier removal; distributions with many extreme mapping artifacts may still bias the mean and SD
- This trimming-based estimation suits Fast-SG synthetic libraries; it does not transfer directly to 10X Genomics or DOVETAIL genomics data where insert sizes span a range and orientation may be unknown

## Evidence

- [other] Trimmed-tail insert-size statistics (source card workflow, task_008): [evidence span withheld — anchor status: not_found]
- [discussion] Insert size bounded by long-read length: "The insert size is restricted to the actual long-read size."
- [methods] Read configuration file used to define libraries and insert sizes: "Example of a read configuration file can be found in examples/ecoli-reads.txt"
