---
name: novel-isoform-model-merging-with-unique-ids
description: Use when region-split IsoQuant runs (distrreg, e.g. g5000000) produce
  separate per-region transcript reconstructions that must be combined into one coherent
  transcript model set for read assignment and counting.
when_to_use_negative:
- The genome contains organelle chromosomes that were not excluded via the refdir
  organelles configuration — the isoquant-based code often hangs or crashes on organelle
  data
- The analysis is a small single-region job where region-splitting and merging add
  overhead without benefit
- Only PacBio data is being analyzed without the appropriate PacBio minimap2 index
  created via '-pacbioindex 1' in scywalker_makerefdir
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scywalker
  role: Orchestrates region distribution (distrreg), per-region IsoQuant reconstruction,
    and merging of per-region models using unique location-based transcript IDs
  repo: https://github.com/derijkp/scywalker
- name: IsoQuant
  role: Reconstructs known and novel transcript models per genomic region from the
    aligned long reads
  repo: https://github.com/ablab/IsoQuant
- name: minimap2
  role: Aligns barcoded FASTQ reads to the reference genome with the splice preset
    (minimap2_splice), providing the input alignments for reconstruction
- name: genomecomb
  role: Implementation framework in which scywalker (and the merging/assignment logic)
    is largely implemented; provided in the scywalker distribution
  repo: https://github.com/derijkp/genomecomb
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae549
    title: 'Scywalker: scalable end-to-end data analysis workflow for long-read single-cell
      transcriptome sequencing'
  source_content_hash: sha256:77baf5e955b989acafbf9e8c522c4696bbf7e522bc88baba767665c36a044e66
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/derijkp/scywalker
  tools:
  - IsoQuant
  - genomecomb
  - minimap2
  - scywalker
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# Novel isoform model merging with unique IDs

## Summary

Reconstructs known and novel transcript models per genomic region with IsoQuant and merges the per-region models into a single set using unique location-based transcript IDs, avoiding duplicate or conflicting model identifiers when region-splitting is used.

## When to use

Use when region-split IsoQuant runs (distrreg, e.g. g5000000) produce separate per-region transcript reconstructions that must be combined into one coherent transcript model set for read assignment and counting.

## When NOT to use

- The genome contains organelle chromosomes that were not excluded via the refdir organelles configuration — the isoquant-based code often hangs or crashes on organelle data
- The analysis is a small single-region job where region-splitting and merging add overhead without benefit
- Only PacBio data is being analyzed without the appropriate PacBio minimap2 index created via '-pacbioindex 1' in scywalker_makerefdir

## Inputs

- Barcoded long-read alignments to the reference genome (minimap2 splice preset, from the refdir)
- Reference directory (refdir) with genome fasta, transcript GTF/gene tsv, minimap2 indexes, and organelles configuration
- IsoQuant per-region transcript reconstruction outputs
- Optional: iso_joint_min integer (minimum number of samples an isoform must be found in for joint analysis)

## Outputs

- Merged transcript model set (known and novel isoforms) with unique location-based transcript IDs
- Adapted read_assignment table with ambiguity, gambiguity, umicount, covered_pct, polyA, and completeness fields
- Per-droplet isoform and gene count tables (_raw and _filtered variants)

## How to apply

First split the barcoded long-read alignments into smaller regions (default target ~5 Mb via distrreg g5000000, with splits only allowed in 250kbase gene-free regions) so each region can be processed independently, reducing peak memory. Run IsoQuant transcript reconstruction per region on the minimap2-splice alignments, excluding organelle chromosomes configured in the refdir because IsoQuant-based code hangs or crashes on organelle data. Merge the per-region transcript models (known and novel) into one model set, assigning each merged model a unique location-based transcript ID so identical models from different regions collapse and distinct models never collide. The merged models then serve as the reference for read assignment and the per-droplet isoform/gene count tables. Joint analysis of isoforms across samples can additionally be tuned with -iso_joint isoquant and -iso_joint_min (minimum number of samples an isoform must be found in).

## Related tools

- **scywalker** (Orchestrates region distribution (distrreg), per-region IsoQuant reconstruction, and merging of per-region models using unique location-based transcript IDs) — https://github.com/derijkp/scywalker
- **IsoQuant** (Reconstructs known and novel transcript models per genomic region from the aligned long reads) — https://github.com/ablab/IsoQuant
- **minimap2** (Aligns barcoded FASTQ reads to the reference genome with the splice preset (minimap2_splice), providing the input alignments for reconstruction)
- **genomecomb** (Implementation framework in which scywalker (and the merging/assignment logic) is largely implemented; provided in the scywalker distribution) — https://github.com/derijkp/genomecomb

## Examples

```
scywalker -v 1 -d 8 -dmaxmem 64G -refdir g17 -sc_expectedcells 183 -cellmarkerfile markers_chr17.tsv -threads 6 test10x
```

## Evaluation signals

- All merged transcript models carry unique location-based transcript IDs with no identifier collisions between per-region reconstructions
- The read_assignment table exposes all required fields: ambiguity, gambiguity, umicount, covered_pct, polyA, completeness
- Resulting _filtered count files contain only emptydrops-approved cells; _raw files contain all detected cells
- Strict counts are restricted to unique reads covering >= 90% of the transcript
- No IsoQuant hang/crash occurred, confirming organelle chromosomes were excluded via the refdir organelles setting

## Limitations

- IsoQuant-based code often hangs or crashes on organelle data, so organelle chromosomes must be specified with scywalker_makerefdir -organelles to route them to a different algorithm
- Region splitting is only allowed in gene-free regions (splits made in 250kbase gene-free regions with default target 5Mbase), so very gene-dense genomes may yield limited parallelism
- Default chromosome grouping concatenates alt/unplaced chromosomes under a name-based pattern (e.g. chr1_KI270706v1_random groups under chr1_); genomes lacking this pattern require -groupchromosomes with custom regular expressions

## Evidence

- [results] Region-split calling with weighted/unique/strict counting: "multimapping reads are weighed (if maps to N genes -> each gets 1/N count)"
- [results] Strict counts threshold: "counts_strict: only unique reads that cover >= 90% of the transcript"
- [methods] Organelle exclusion rationale: "The isoquant based code often hangs or crashes on the very different organelle data."
- [readme] Region distribution default: [evidence span withheld — anchor status: not_found]
- [readme] Chromosome grouping caveat: "the scywalker_makerefdir command supports the option `-groupchromosomes` to specify a different way of grouping"
