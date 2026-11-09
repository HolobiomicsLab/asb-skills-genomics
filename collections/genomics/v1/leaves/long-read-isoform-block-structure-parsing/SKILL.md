---
name: long-read-isoform-block-structure-parsing
description: Use when you have isocirc.out records from isoCirc long-read circRNA
  catalogs (12 human tissues + HEK293) and need to recover exact exonic compositions
  and count internal alternative splicing events within circRNAs.
when_to_use_negative:
- Input is short-read RNA-seq data (e.g., RNase R-treated libraries) — short reads
  do not determine full-length circRNA sequences or internal splicing events
- Input is a circRNA catalog lacking per-isoform block structure (e.g., BSJ-only records
  from circBase/MiOncoCirc) — internal event classification requires blockType/blockStarts/blockSize
- The goal is BSJ discovery rather than internal isoform composition — use the upstream
  isocirc pipeline BSJ-calling step instead
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0102
tools:
- name: isoCirc
  role: Source of the isocirc.out full-length circRNA isoform records (blockType,
    blockStarts, blockSize, readCount) that this skill parses and classifies
  repo: https://github.com/Xinglab/isoCirc
- name: Python
  role: Scripting language for parsing isocirc.out block fields, intersecting with
    GTF exon coordinates, and tallying event counts
  repo: https://github.com/Xinglab/isoCirc
- name: minimap2
  role: Upstream dependency that maps consensus sequences used to build the isoform
    block structures in isocirc.out
  repo: https://github.com/lh3/minimap2
- name: bedtools
  role: Upstream dependency for interval operations between isoform blocks and annotated
    exon coordinates
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-20459-8
    title: isoCirc catalogs full-length circular RNA isoforms in human transcriptomes
  source_content_hash: sha256:2888ed08af2ffaa2840e5b0cbb28eeb736a81d108336e903340d1399e388fa93
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Xinglab/isoCirc
  tools:
  - Python
  - bedtools
  - isoCirc
  - minimap2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# long-read-isoform-block-structure-parsing

## Summary

Parses full-length circRNA isoform records from isoCirc's isocirc.out (blockType, blockStarts, blockSize fields) to reconstruct each isoform's exact exonic composition and classify internal alternative splicing events (skipped exon, alt 5' splice site, alt 3' splice site, retained intron) within circRNAs.

## When to use

Use when you have isocirc.out records from isoCirc long-read circRNA catalogs (12 human tissues + HEK293) and need to recover exact exonic compositions and count internal alternative splicing events within circRNAs.

## When NOT to use

- Input is short-read RNA-seq data (e.g., RNase R-treated libraries) — short reads do not determine full-length circRNA sequences or internal splicing events
- Input is a circRNA catalog lacking per-isoform block structure (e.g., BSJ-only records from circBase/MiOncoCirc) — internal event classification requires blockType/blockStarts/blockSize
- The goal is BSJ discovery rather than internal isoform composition — use the upstream isocirc pipeline BSJ-calling step instead

## Inputs

- isocirc.out files per sample, with blockType (E/I/N), blockStarts, blockSize, and readCount columns
- Gene annotation GTF providing exon coordinates
- Optional: reference genome FASTA and circRNA BSJ BED/GTF as used by isocirc

## Outputs

- Per-isoform internal block composition (exon/intron segment lists) per circRNA
- Tally of internal alternative splicing events per category (SE, A5SS, A3SS, retained intron) and per read-count cutoff (>=2, >=5, >=10)
- List of retained-intron events classified as EIciRNA

## How to apply

Load the full-length circRNA isoform structures from isocirc.out for all 12 tissues plus HEK293, parsing the blockType field (E: exon, I: intron, N: intergenic) together with blockStarts and blockSize to recover each isoform's exact exonic composition. Compare each isoform's internal block structure against annotated exon coordinates from the gene annotation GTF to classify internal alternative splicing events into the four major patterns: skipped exon, alternative 5' splice site, alternative 3' splice site, and retained intron. Filter events by the readCount column of isocirc.out at three support cutoffs (>=2, >=5, and >=10 isoCirc reads), tallying counts per category and cutoff into a summary table. Mark retained-intron events (introns retained between exons within the circle) as EIciRNA events and compare their abundance to short-read RNA-seq estimates (~20-fold excess expected). Rationale: short-read RNA-seq cannot experimentally determine full-length circRNA sequences or internal splicing, so only block-structure-level parsing of long-read isoform calls yields these event counts.

## Related tools

- **isoCirc** (Source of the isocirc.out full-length circRNA isoform records (blockType, blockStarts, blockSize, readCount) that this skill parses and classifies) — https://github.com/Xinglab/isoCirc
- **Python** (Scripting language for parsing isocirc.out block fields, intersecting with GTF exon coordinates, and tallying event counts) — https://github.com/Xinglab/isoCirc
- **minimap2** (Upstream dependency that maps consensus sequences used to build the isoform block structures in isocirc.out) — https://github.com/lh3/minimap2
- **bedtools** (Upstream dependency for interval operations between isoform blocks and annotated exon coordinates)

## Examples

```
isocirc -t 1 read_toy.fa chr16_toy.fa chr16_toy.gtf chr16_circRNA_toy.bed output
```

## Evaluation signals

- Reproduced total internal alternative splicing events matches 5,325 when requiring the minor isoform to have at least 2 isoCirc reads across all samples combined
- Retained-intron event count equals 720 EIciRNA events at the read>=2 cutoff
- At the readCount>=10 cutoff, 177 retained introns are recovered
- Every parsed block interval is assigned one of the blockType classes E (exon), I (intron), or N (intergenic)
- Event tallies are consistent across the three support cutoffs (>=2, >=5, >=10 reads), with counts monotonically decreasing as cutoff increases

## Limitations

- Only applicable to full-length isoform calls from isoCirc (RCA + nanopore long-read) data; short-read RNA-seq cannot interrogate internal circRNA splicing and existing reconstruction methods apply only to short circRNAs (200–500 nt)
- isoCirc is far from saturation at current sequencing depth, so more circRNA isoforms and events will be identified with deeper sequencing
- Filtering is stricter for BSJs involving novel splice sites than for known splice sites, which affects which isoforms enter the event-tally step

## Evidence

- [abstract] We identify widespread alternative splicing events within the internal part of circRNAs, including 720 retained intron events corresponding to a class of exon-intron circRNAs (EIciRNAs): "We identify widespread alternative splicing events within the internal part of circRNAs, including 720 retained intron events corresponding to a class of exon-intron circRNAs (EIciRNAs)"
- [readme] README: isoCirc pipeline definition: "computational pipeline to characterize full-length circRNA isoforms using rolling circle amplification (RCA) followed by long-read sequencing"
- [other] Task card workflow on isocirc.out parsing: [evidence span withheld — anchor status: not_found]
- [intro] Discussion on short-read limitation: "an inherent limitation of this approach is that short-read RNA-seq does not experimentally determine the full-length sequences and internal alternative splicing events within circRNAs"
