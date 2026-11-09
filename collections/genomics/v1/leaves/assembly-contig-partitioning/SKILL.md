---
name: assembly-contig-partitioning
description: Use when comparing two long-read whole-genome assemblies and you need
  to determine whether a tandem/repetitive locus (e.g., the ~40 kb HLA repeat unit)
  was collapsed to one copy in one assembly but resolved into two copies in the other.
when_to_use_negative:
- The locus of interest is not repetitive or contains a single copy per haplotype
  — copy-number partitioning adds no discriminating information
- You only need overlap detection itself (Fedrann's core FE/DR/ANN pipeline on reads)
  rather than downstream assembly contig comparison
- The assemblies to compare were produced by the same assembler with identical parameters
  and differ only in polishing, where repeat resolution is unlikely to change
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: Fedrann
  role: Supplies overlaps.tsv (neighbor_rank, distance, orientations) used to group
    contigs that duplicate the same repeat unit; overlap graph built with k=30 on
    the H4C dataset
  repo: https://github.com/jzhang-dev/FEDRANN
- name: Shasta
  role: Assembly framework providing the contigs being partitioned; native Shasta
    vs Fedrann-Shasta outputs are contrasted for copy-number calls
  repo: https://github.com/jzhang-dev/Fedrann-Shasta
- name: minimap2
  role: Aligns contigs against the HLA locus reference to select region-matching contigs,
    and supports pairwise self-alignments to count distinct repeat copies
- name: BLAST
  role: Alternative to minimap2 self-alignments for pairwise comparison of partitioned
    contigs to tally repeat-unit copies
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/gigascience/giag048
    title: 'FEDRANN: effective long-read overlap detection based on dimensionality
      reduction and approximate nearest neighbors'
  source_content_hash: sha256:c45543fdabcdd62ff001563ba406bac806255a85c0dc2a0f55dc1f35199bb361
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jzhang-dev/FEDRANN
  tools:
  - BLAST
  - Fedrann
  - Shasta
  - minimap2
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# assembly-contig-partitioning

## Summary

Partitions assembly contigs that map to a target locus (e.g., the human HLA region) into groups representing distinct copies of a repeat unit, then makes per-assembly copy-number calls to detect collapsed or correctly resolved repeats when comparing two assemblers (e.g., native Shasta vs Fedrann-Shasta).

## When to use

Use when comparing two long-read whole-genome assemblies and you need to determine whether a tandem/repetitive locus (e.g., the ~40 kb HLA repeat unit) was collapsed to one copy in one assembly but resolved into two copies in the other.

## When NOT to use

- The locus of interest is not repetitive or contains a single copy per haplotype — copy-number partitioning adds no discriminating information
- You only need overlap detection itself (Fedrann's core FE/DR/ANN pipeline on reads) rather than downstream assembly contig comparison
- The assemblies to compare were produced by the same assembler with identical parameters and differ only in polishing, where repeat resolution is unlikely to change

## Inputs

- Contigs (FASTA) from two whole-genome assemblies of the same sample (e.g., native Shasta H4C and Fedrann-Shasta H4C)
- Locus reference sequence (e.g., HLA locus reference)
- Fedrann overlaps.tsv (query_name, query_orientation, target_name, target_orientation, neighbor_rank, distance)
- Fedrann overlap graph parameters (k=30 for the H4C dataset)

## Outputs

- Per-assembly partition of locus-matching contigs into repeat-copy groups
- Per-assembly copy-number calls table (e.g., hla_copy_number_calls.tsv)
- Flags identifying collapsed vs resolved repeat copies per assembler

## How to apply

Align the contigs of each assembly (native Shasta and Fedrann-Shasta) against the locus reference using minimap2 and retain only contigs with matches to the region. Partition the region-matching contigs per assembly, using Fedrann overlaps.tsv entries (query_name, query_orientation, target_name, target_orientation, neighbor_rank, distance from the overlap graph built with k=30 on the H4C dataset) to group contigs that represent duplicates of the same repeat unit. Count distinct copies of the ~40 kb repeat unit per assembly by pairwise comparison of partitioned contigs (BLAST or minimap2 self-alignments) and by tallying neighbor_rank groups from the Fedrann overlap graph. Record the per-assembly copy-number call, flagging the collapsed assembly where the unit merges to one copy and the resolved assembly where two copies are recovered, and write the calls to a table such as hla_copy_number_calls.tsv.

## Related tools

- **Fedrann** (Supplies overlaps.tsv (neighbor_rank, distance, orientations) used to group contigs that duplicate the same repeat unit; overlap graph built with k=30 on the H4C dataset) — https://github.com/jzhang-dev/FEDRANN
- **Shasta** (Assembly framework providing the contigs being partitioned; native Shasta vs Fedrann-Shasta outputs are contrasted for copy-number calls) — https://github.com/jzhang-dev/Fedrann-Shasta
- **minimap2** (Aligns contigs against the HLA locus reference to select region-matching contigs, and supports pairwise self-alignments to count distinct repeat copies)
- **BLAST** (Alternative to minimap2 self-alignments for pairwise comparison of partitioned contigs to tally repeat-unit copies)

## Examples

```
minimap2 -ax asm5 hla_locus_ref.fa shasta_contigs.fa > hla_contigs.sam
```

## Evaluation signals

- The call table (e.g., hla_copy_number_calls.tsv) reports one copy for native Shasta and two copies for Fedrann-Shasta in the HLA region, matching the paper's claim
- Both assemblies partition the HLA region into two contigs with a breakpoint at roughly the same location, confirming the assemblies are otherwise comparable
- The number of distinct neighbor_rank groups in the Fedrann overlaps.tsv partition equals the number of repeat-unit copies inferred from contig self-alignments (BLAST/minimap2)
- Only contigs with confirmed minimap2 matches to the HLA locus reference are included in each partition (no off-locus contigs counted)
- Copy-number counts are concordant between the pairwise-alignment method and the overlap-graph neighbor_rank tally

## Limitations

- Fedrann-Shasta results are only a proof-of-concept, constrained by the internal architecture of established assemblers
- Incorporating Fedrann into existing assemblers is non-trivial due to opaque in-memory data structures and tight coupling between pipeline stages
- Reproducibility depends solely on GitHub repositories; no DOI or Zenodo archive of the code/data is provided

## Evidence

- [abstract] By integrating Fedrann into the Shasta assembler, we successfully reconstructed human whole genomes, achieving high assembly contiguity and quality: "By integrating Fedrann into the Shasta assembler, we successfully reconstructed human whole genomes, achieving high assembly contiguity and quality"
- [readme] neighbor_rank ... distance columns of overlaps.tsv: "neighbor_rank: The similarity rank of the target sequence among all potential matches for the query sequence, where 1 is the best match"
- [readme] distance column description: "distance: Measures the dissimilarity between the embedded vectors of the query and target sequences. A smaller value indicates higher similarity between the sequences."
- [discussion] k=30 for H4C dataset: [evidence span withheld — anchor status: not_found]
- [discussion] Fedrann-Shasta proof-of-concept caveat: "we consider our current Fedrann-Shasta results a successful proof-of-concept rather than the upper limit of Fedrann’s potential"
