---
name: dna-barcoding-gap-analysis
description: Use when deciding whether an alternative barcode locus (e.g. long Nanopore
  rDNA amplicons) discriminates species as well as the standard COI barcode, given
  consensus sequences from multiple specimens with known species assignments.
when_to_use_negative:
- Input is raw MinION signal or un-demultiplexed dual-indexed reads — run minibar
  demultiplexing and consensus calling first; this skill starts from consensus sequences
- Goal is metabarcoding of bulk community samples — the article found long rDNA amplicon
  sequencing recovers highly skewed diversity estimates in bulk samples and is not
  yet reliable there
- Specimens lack species assignments, so intra- vs interspecific distance classes
  and barcode gaps cannot be defined
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_0114
tools:
- name: MiniBar
  role: Demultiplexes dual barcode- and primer-indexed Nanopore reads into per-specimen
    sample files (-F) with barcode/primer trimming (-T), producing the per-specimen
    reads that precede consensus sequence generation for the distance analysis
  repo: https://github.com/calacademy-research/minibar
- name: Edlib
  role: Python/C++ edit-distance (Levenshtein) library used by minibar for fast approximate
    matching of barcodes and primers during demultiplexing
  repo: https://github.com/Martinsos/edlib
- name: Python 2.7
  role: Scripting language for computing pairwise distance matrices, Mantel permutation
    tests, and barcode-gap comparisons
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giz006
    title: Nanopore sequencing of long ribosomal DNA amplicons enables portable and
      simple biodiversity assessments with high phylogenetic resolution across broad
      taxonomic scale
  source_content_hash: sha256:3c2495f39aa552d4a0adee5971ba7dedeee6ef3677a149e4c41bec5133c6d6aa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/calacademy-research/minibar
  tools:
  - Edlib
  - MiniBar
  - Python 2.7
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_0114
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# dna-barcoding-gap-analysis

## Summary

Compares the taxonomic resolution of candidate DNA barcode loci — here mitochondrial COI (418 bp) versus the whole ribosomal DNA cluster and its separate gene regions — by partitioning pairwise genetic distances into intra- vs interspecific classes, correlating marker distance matrices with a Mantel test, and contrasting barcode-gap behavior between markers.

## When to use

Use when deciding whether an alternative barcode locus (e.g. long Nanopore rDNA amplicons) discriminates species as well as the standard COI barcode, given consensus sequences from multiple specimens with known species assignments.

## When NOT to use

- Input is raw MinION signal or un-demultiplexed dual-indexed reads — run minibar demultiplexing and consensus calling first; this skill starts from consensus sequences
- Goal is metabarcoding of bulk community samples — the article found long rDNA amplicon sequencing recovers highly skewed diversity estimates in bulk samples and is not yet reliable there
- Specimens lack species assignments, so intra- vs interspecific distance classes and barcode gaps cannot be defined

## Inputs

- FASTA/FASTQ of specimen consensus sequences (COI 418 bp and long rDNA consensus sequences, e.g. from the GigaScience repository deposit)
- Species assignment table mapping each of the 47 specimens to one of 16 Tetragnatha species
- Optional: per-read demultiplexed sample files from minibar (-F) for downstream consensus construction

## Outputs

- Pairwise genetic distance matrices for COI, whole rDNA cluster, and individual rDNA gene regions, partitioned into intra- vs interspecific classes
- Mantel test correlation statistics (R2 = 0.70, P < 0.001 for COI vs rDNA) with permutation-based significance
- Barcode-gap comparison contrasting maximum intraspecific vs minimum interspecific distances per marker, written to tetragnatha_distance_correlation_results.tsv

## How to apply

Load the consensus sequences of all specimens (here 47 Hawaiian Tetragnatha specimens in 16 species: 418 bp COI and the long rDNA consensus from the MinION runs, demultiplexed with MiniBar at default barcode edit distance 4 and primer edit distance 11). For each marker — whole rDNA cluster and each rDNA gene region separately, plus COI — compute the pairwise genetic distance matrix over all specimen pairs and partition distances into intraspecific vs interspecific classes using the 16 species assignments. Correlate the COI matrix with the rDNA matrix using a Mantel test with permutation-based significance assessment, and fit the relationship to quantify R2. Assess barcode gaps by contrasting, per marker, each species' maximum intraspecific distance against the minimum interspecific distance; a marker with a clean gap separates species reliably. Write the distance matrices, Mantel correlation statistics (expecting R2 = 0.70, P < 0.001), and barcode-gap comparison to an output table such as tetragnatha_distance_correlation_results.tsv.

## Related tools

- **MiniBar** (Demultiplexes dual barcode- and primer-indexed Nanopore reads into per-specimen sample files (-F) with barcode/primer trimming (-T), producing the per-specimen reads that precede consensus sequence generation for the distance analysis) — https://github.com/calacademy-research/minibar
- **Edlib** (Python/C++ edit-distance (Levenshtein) library used by minibar for fast approximate matching of barcodes and primers during demultiplexing) — https://github.com/Martinsos/edlib
- **Python 2.7** (Scripting language for computing pairwise distance matrices, Mantel permutation tests, and barcode-gap comparisons)

## Examples

```
./minibar.py -T -F Demultiplex.txt my_sequences.fastq
```

## Evaluation signals

- Mantel correlation between COI and rDNA pairwise distance matrices reproduces R2 = 0.70 with P < 0.001 as reported in the paper
- Permutation-based Mantel significance is P < 0.001
- COI and rDNA distance matrices cover the same 47 specimens × 47 specimen pairs with identical ordering, partitioned into intra- vs interspecific classes across the 16 species
- Barcode-gap table reports, for each marker and each rDNA gene region, maximum intraspecific vs minimum interspecific distances, allowing direct COI-vs-rDNA contrast
- Output TSV (tetragnatha_distance_correlation_results.tsv) contains distance matrices, Mantel statistics, and the barcode-gap comparison

## Limitations

- Long rDNA amplicon sequencing recovers highly skewed diversity estimates for bulk community (metabarcoding) samples; the source of the bias is undetermined and further research is required before use on bulk samples
- Minibar demultiplexing depends on dual-index barcode and primer matches within default edit-distance thresholds (barcode 4, primer 11) and a 80 nt end-search window; sequences with primer misses (lowercase 'h' hits, -1 primer distance) may fail sample identification
- The correlation is validated for the 47-specimen Hawaiian Tetragnatha dataset; R2 = 0.70 need not generalize to other taxa or markers

## Evidence

- [results] rDNA-based pairwise distances within and between Hawaiian Tetragnatha species are significantly correlated with COI-based distances: [evidence span withheld — anchor status: not_found]
- [intro] Using a dataset of spiders, we compare the taxonomic resolution of the ribosomal cluster with that offered by molecular barcoding using mitochondrial COI: "Using a dataset of spiders, we compare the taxonomic resolution of the ribosomal cluster with that offered by molecular barcoding using mitochondrial COI"
- [methods] Compare barcode-gap behavior between markers by contrasting maximum intraspecific against minimum interspecific distances for COI versus the rDNA regions.: [evidence span withheld — anchor status: not_found]
- [discussion] The program Minibar can be found at https://github.com/calacademy-research/minibar: "The program Minibar can be found at https://github.com/calacademy-research/minibar"
- [readme] ./minibar.py -T -F Demultiplex.txt my_sequences.fastq: "a typical run might look like:
```
./minibar.py -T -F Demultiplex.txt my_sequences.fastq
```"
