---
name: long-read-genome-alignment
description: Use when long nanopore reads or RCA-derived consensus sequences must
  be mapped to a reference genome (e.g. hg19) to identify back-splice junctions and
  full-length circRNA isoforms from isoCirc data.
when_to_use_negative:
- Input is short-read RNA-seq (the article's motivation is that short-read RNA-seq
  does not experimentally determine full-length circRNA sequences); use short-read
  spliced alignment instead
- The aim is raw long-read alignment only (genomic ONT/PacBio reads) without isoCirc
  circRNA consensus filtering — use minimap2 directly with its standard presets (map-ont,
  splice) rather than via the isoCirc wrapper
- Sequences have not passed the TRF consensus-calling and min-copy 2.0 filtering steps;
  unfiltered RCA repeats will inflate false-positive BSJ calls
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: minimap2
  role: Splice-aware long-read aligner used to map isoCirc consensus sequences to
    the reference genome with tuned identity and mapping-quality thresholds
  repo: https://github.com/lh3/minimap2
- name: isoCirc
  role: Pipeline wrapper that orchestrates consensus filtering and minimap2 alignment,
    then derives high-confidence BSJs and full-length circRNA isoforms from the alignments
  repo: https://github.com/Xinglab/isoCirc
- name: bedtools
  role: Dependency (>= v2.27.0) of the isoCirc pipeline that performs the annotation-aware
    BSJ identification after alignment
  repo: https://bedtools.readthedocs.io/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-20459-8
    title: isoCirc catalogs full-length circular RNA isoforms in human transcriptomes
  source_content_hash: sha256:c685c3ec52c417b2ea16bdfa9f46891355a0e7f78006d65d2b623265905fc9cf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - Python
  - bedtools
  - isoCirc
  - minimap2
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# long-read-genome-alignment

## Summary

Aligns long nanopore reads or isoCirc consensus sequences to a reference genome using minimap2 with splice-aware, quality-tuned settings so back-splice junctions and full-length circRNA isoforms can be placed on genomic coordinates.

## When to use

Use when long nanopore reads or RCA-derived consensus sequences must be mapped to a reference genome (e.g. hg19) to identify back-splice junctions and full-length circRNA isoforms from isoCirc data.

## When NOT to use

- Input is short-read RNA-seq (the article's motivation is that short-read RNA-seq does not experimentally determine full-length circRNA sequences); use short-read spliced alignment instead
- The aim is raw long-read alignment only (genomic ONT/PacBio reads) without isoCirc circRNA consensus filtering — use minimap2 directly with its standard presets (map-ont, splice) rather than via the isoCirc wrapper
- Sequences have not passed the TRF consensus-calling and min-copy 2.0 filtering steps; unfiltered RCA repeats will inflate false-positive BSJ calls

## Inputs

- Filtered long-read consensus sequences (FASTA) called from isoCirc RCA tandem repeats
- Reference genome sequence file (e.g. hg19 FASTA)
- Gene annotation GTF (e.g. Ensembl GRCh37.87)
- circRNA database annotation in BED/GTF format (multiple files joined with ',')

## Outputs

- Genomic alignments of circRNA consensus sequences with high-quality alignment ratios 1.1/0.9 and identity >= 0.75
- High-confidence back-splice junction calls with genomic coordinates (isocirc.out, isocirc.bed)
- Per-library statistics (isocirc_stats.out)

## How to apply

Within the isoCirc pipeline, first call consensus sequences from RCA tandem-repeat reads with Tandem Repeats Finder (match 2, mismatch 7, indel 7, match-frac 80, indel-frac 10, min-score 100, max-period 2000), filter them (min-len 30, min-copy 2.0), then map the consensus sequences to the reference genome (e.g. hg19 FASTA) with minimap2 (>= 2.11). Alignment quality is controlled by high-mapping-quality ratio thresholds (1.1/0.9) and a minimum identity of 0.75, with stricter near-perfect alignment required for BSJs involving novel splice sites than for known splice sites. High-confidence BSJs are then identified using the canonical GT/AG motif, allowing at most 1 mis/ins/del in the 20-bp exonic flanks (0 in the 4-bp key flanks) and a minimum BSJ genomic distance of 150. minimap2 should be built or installed before running isoCirc, which invokes it internally (custom path via --minimap2); per-library results land in isocirc.out, isocirc.bed, and isocirc_stats.out.

## Related tools

- **minimap2** (Splice-aware long-read aligner used to map isoCirc consensus sequences to the reference genome with tuned identity and mapping-quality thresholds) — https://github.com/lh3/minimap2
- **isoCirc** (Pipeline wrapper that orchestrates consensus filtering and minimap2 alignment, then derives high-confidence BSJs and full-length circRNA isoforms from the alignments) — https://github.com/Xinglab/isoCirc
- **bedtools** (Dependency (>= v2.27.0) of the isoCirc pipeline that performs the annotation-aware BSJ identification after alignment) — https://bedtools.readthedocs.io/

## Examples

```
isocirc -t 1 read_toy.fa chr16_toy.fa chr16_toy.gtf chr16_circRNA_toy.bed output
```

## Evaluation signals

- Per-library high-confidence BSJ counts fall in the reported 47,434-66,194 range across the six HEK293 nanopore libraries
- Per-library full-length circRNA isoform counts fall in the reported 41,095-57,763 range
- The combined read >= 2 catalog contains 47,061 full-length circRNA isoforms
- Each mapped BSJ shows the canonical GT/AG motif with at most 1 mis/ins/del in the 20-bp exonic flanks (0 in the 4-bp key flanks) and a minimum BSJ genomic distance of 150
- Novel-splice-site BSJs meet the stricter near-perfect alignment criterion between consensus and genomic sequence around the BSJ

## Limitations

- Stringent filtering is stricter for BSJs with novel splice sites; the pipeline cannot interrogate circRNAs beyond the near-perfect-alignment requirement for novel sites
- isoCirc is far from saturation at current sequencing depth; more circRNA isoforms are expected with deeper sequencing
- minimap2 indexing parameters (-k, -w, -H, -I) cannot be changed during mapping once an index is built with -d

## Evidence

- [methods] minimap2 filtering and mapping thresholds: "Filtering and mapping of consensus sequences (minimap2):"
- [discussion] stringent novel splice-site criteria: "the filtering criteria are much stricter (as compared to BSJs involving known splice sites)"
- [discussion] near-perfect alignment requirement: "requiring near-perfect alignment between the isoCirc consensus sequence and the corresponding genomic sequence around the BSJ"
- [readme] minimap2 dependency statement: "and minimap2 [`minimap2`(>= 2.11)](https://github.com/lh3/minimap2)"
- [readme] minimap2 splice-aware use case: [evidence span withheld — anchor status: external_doc]
