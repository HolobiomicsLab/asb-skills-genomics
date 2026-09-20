---
name: circrna-back-splice-junction-coordinate-matching
description: Use when you have a table of BSJ genomic coordinates (chromosome, startCoor0based,
  endCoor, geneStrand, readCount) from long-read circRNA detection and need to determine
  what fraction overlaps annotated circRNAs in circBase or MiOncoCirc versus is novel.
when_to_use_negative:
- Input is raw isoCirc long reads (FASTA/FASTQ) rather than the post-pipeline isocirc.out
  BSJ table — run the isoCirc pipeline first to call BSJs
- You need full-length circRNA isoform sequences or internal alternative-splicing
  characterization, not junction-level known/novel annotation — use the isoCirc full-length
  isoform pipeline instead
- The circRNA of interest has no circBase/MiOncoCirc-style coordinate annotation available
  for the organism or build; coordinate matching is undefined without reference databases
edam_operation: http://edamontology.org/operation_2403
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3308
tools:
- name: isoCirc
  role: Produces the isocirc.out tables containing BSJ genomic coordinates and readCount
    fields that are the input to the known/novel coordinate matching
  repo: https://github.com/Xinglab/isoCirc
- name: bedtools
  role: Intersection of BSJ intervals against circBase and MiOncoCirc annotation files
    to flag database-known junctions (requires >= v2.27.0)
  repo: https://bedtools.readthedocs.io/
- name: Python
  role: pandas loading of isocirc.out tables, readCount aggregation over technical
    replicates, interval construction from startCoor0based/endCoor/geneStrand, and
    summary-statistics computation
  repo: https://github.com/Xinglab/isoCirc
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-20459-8
    title: isoCirc catalogs full-length circular RNA isoforms in human transcriptomes
  source_content_hash: sha256:4c44299cee930cc25b0cb1b3e4301fd2b3ad8f3a43e028b76e13f94a389b1b2c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Xinglab/isoCirc
  tools:
  - Python
  - bedtools
  - isoCirc
  edam_operation: http://edamontology.org/operation_2403
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# circRNA back-splice junction coordinate matching

## Summary

Classify long-read-detected circRNA back-splice junctions (BSJs) as known or novel by coordinate-level intersection against circRNA databases (circBase, MiOncoCirc), quantifying the known/novel fraction stratified by replicate support. Used in the isoCirc study to benchmark the novelty of 49,993 high-confidence HEK293 BSJs.

## When to use

Use when you have a table of BSJ genomic coordinates (chromosome, startCoor0based, endCoor, geneStrand, readCount) from long-read circRNA detection and need to determine what fraction overlaps annotated circRNAs in circBase or MiOncoCirc versus is novel.

## When NOT to use

- Input is raw isoCirc long reads (FASTA/FASTQ) rather than the post-pipeline isocirc.out BSJ table — run the isoCirc pipeline first to call BSJs
- You need full-length circRNA isoform sequences or internal alternative-splicing characterization, not junction-level known/novel annotation — use the isoCirc full-length isoform pipeline instead
- The circRNA of interest has no circBase/MiOncoCirc-style coordinate annotation available for the organism or build; coordinate matching is undefined without reference databases

## Inputs

- isocirc.out tables per biological replicate (35-column format: BSJ genomic coordinates, readCount, isKnownBSJ fields)
- circBase circRNA annotation file(s) (BED/BED-style circRNA records)
- MiOncoCirc circRNA annotation file(s) (supplied as comma-separated circRNA annotation files)

## Outputs

- Per-BSJ known/novel labels with database membership flags
- Summary statistics table (e.g., known_novel_bsj_summary.csv) with known/novel fractions stratified by one-replicate vs both-replicates support

## How to apply

Load the isoCirc `isocirc.out` tables (35-column format with BSJ genomic coordinates, readCount, and isKnownBSJ fields) per biological replicate, aggregate readCount over technical replicates, and retain high-confidence BSJs supported by >=2 reads. Build BED-style intervals from the startCoor0based/endCoor/geneStrand fields using pandas, then intersect them against circBase and MiOncoCirc annotation files (supplied as comma-separated circRNA annotation files) with `bedtools intersect` to flag BSJs present in either database. Label each BSJ known (True if the BSJ exists in the circRNA annotations) or novel, and compute known/novel fractions separately for BSJs detected in one biological replicate and BSJs detected in both replicates — replicate-level concordance strongly raises the known fraction because reproducibly detected BSJs are less likely to be artifacts. Write per-BSJ labels plus summary statistics to a CSV (e.g., known_novel_bsj_summary.csv).

## Related tools

- **isoCirc** (Produces the isocirc.out tables containing BSJ genomic coordinates and readCount fields that are the input to the known/novel coordinate matching) — https://github.com/Xinglab/isoCirc
- **bedtools** (Intersection of BSJ intervals against circBase and MiOncoCirc annotation files to flag database-known junctions (requires >= v2.27.0)) — https://bedtools.readthedocs.io/
- **Python** (pandas loading of isocirc.out tables, readCount aggregation over technical replicates, interval construction from startCoor0based/endCoor/geneStrand, and summary-statistics computation) — https://github.com/Xinglab/isoCirc

## Examples

```
bedtools intersect -a hek293_bsj.bed -b circBase.bed,MiOncoCirc.bed -c -names circBase,MiOncoCirc > known_novel_bsj_labeled.bed
```

## Evaluation signals

- Known/novel summary must reproduce the published HEK293 reference numbers: 49,993 total high-confidence BSJs; 46.9% known / 53.1% novel for BSJs found in one biological replicate; 85.1% known / 14.9% novel for BSJs found in both biological replicates (16,934 BSJs at read count >=2 in both replicates)
- All BSJs retained pass the >=2 readCount threshold after aggregating over technical replicates
- Every BSJ receives exactly one label (known = True iff present in circBase and/or MiOncoCirc; novel = present in neither)
- Bed-format intervals derive from startCoor0based/endCoor with geneStrand consistency (0-based start convention preserved)

## Limitations

- Novelty assessment is only as complete as circBase and MiOncoCirc; isoCirc is far from saturation at current sequencing depth, so novel calls may simply reflect database gaps
- Known/novel fraction depends on replicate-support filtering — single-replicate BSJs skew heavily novel (53.1%) because many are low-confidence artifacts
- Filtering criteria in isoCirc are much stricter for BSJs involving novel splice sites than for known splice sites, biasing the novel set toward near-perfectly validated junctions

## Evidence

- [results] Of the 16,934 high-confidence HEK293 BSJs detected in both biological replicates at read count ≥2, 85.1% were known BSJs annotated in circBase and/or MiOncoCirc: [evidence span withheld — anchor status: not_found]
- [methods] Number of candidate BSJs reported in existing circRNA BSJ database (circBase / MiOncoCirc): "Number of candidate BSJs reported in existing circRNA BSJ database (circBase / MiOncoCirc)"
- [readme] isoCirc is dependent on two open-source software packages: `bedtools`(>= v2.27.0): "isoCirc is dependent on two open-source software packages: [`bedtools`(>= v2.27.0)]"
- [discussion] the filtering criteria are much stricter (as compared to BSJs involving known splice sites): "the filtering criteria are much stricter (as compared to BSJs involving known splice sites)"
- [discussion] we use stringent criteria to remove potential false-positive detection: "After calling consensus sequences from isoCirc raw reads, we use stringent criteria to remove potential false-positive detection"
