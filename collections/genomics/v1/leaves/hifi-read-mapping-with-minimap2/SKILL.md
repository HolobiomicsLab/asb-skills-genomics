---
name: hifi-read-mapping-with-minimap2
description: Use when HiFi/CCS long reads or simulated pseudo-long-reads must be aligned
  to a genome assembly (e.g., CHM13 T2T) to evaluate assembly accuracy or prepare
  a HiFi mapping file for polishing with NextPolish2.
when_to_use_negative:
- Aligning Oxford Nanopore reads (use -x map-ont / lr:hq presets) or Illumina short
  reads (use -x sr), since the map-hifi preset is tuned for PacBio HiFi/CCS data
- Mapping within highly repetitive regions such as centromeric satellite arrays where
  Winnowmap's weighted minimizer sampling is preferred over minimap2
- Quality assessment that requires k-mer-based QV (Merqury/yak) rather than alignment-identity
  QV against a ground-truth reference
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: minimap2
  role: Aligns HiFi/CCS genomic reads (or pseudo-long-reads) to the assembly using
    the map-hifi preset with base-level alignment
  repo: https://github.com/lh3/minimap2/
- name: samtools
  role: Sorts and indexes minimap2 SAM output into the sorted BAM required by NextPolish2
    and downstream filtering
- name: Winnowmap
  role: Alternative long-read mapper for HiFi reads in repetitive regions, used in
    place of minimap2 when preparing the HiFi mapping file
  repo: https://github.com/marbl/Winnowmap
- name: meryl
  role: Counts repetitive k-mers (k=15, >0.9998 distinct) supplied to Winnowmap's
    -W option when replacing minimap2
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gpbjnl/qzad009
    title: 'NextPolish2: A Repeat-aware Polishing Tool for Genomes Assembled Using
      HiFi Long Reads'
  source_content_hash: sha256:972871b19fcfe3096de6636ff3c13da6d3c67b2d5eec60c4822aa75caa3a3a3b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - NextPolish2
  - Winnowmap
  - meryl
  - minimap2
  - samtools
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# hifi-read-mapping-with-minimap2

## Summary

Map PacBio HiFi long reads (or pseudo-long-reads derived from a reference) against a genome assembly using minimap2's map-hifi preset, sort and index the resulting BAM with samtools, and filter alignments by query coverage and identity for downstream accuracy evaluation such as alignment-identity QV.

## When to use

Use when HiFi/CCS long reads or simulated pseudo-long-reads must be aligned to a genome assembly (e.g., CHM13 T2T) to evaluate assembly accuracy or prepare a HiFi mapping file for polishing with NextPolish2.

## When NOT to use

- Aligning Oxford Nanopore reads (use -x map-ont / lr:hq presets) or Illumina short reads (use -x sr), since the map-hifi preset is tuned for PacBio HiFi/CCS data
- Mapping within highly repetitive regions such as centromeric satellite arrays where Winnowmap's weighted minimizer sampling is preferred over minimap2
- Quality assessment that requires k-mer-based QV (Merqury/yak) rather than alignment-identity QV against a ground-truth reference

## Inputs

- PacBio HiFi/CCS reads in FASTA/FASTQ (optionally gzip-compressed) or pseudo-long-reads split from a reference FASTA (e.g., 6250 × 500 kb CHM13 fragments)
- Target genome assembly FASTA (e.g., original, RM-polished, or NextPolish2-polished CHM13 assembly)
- Repetitive k-mer list from meryl (only if using the Winnowmap alternative)

## Outputs

- Coordinate-sorted, indexed BAM of HiFi alignments (e.g., hifi.map.sort.bam) ready for NextPolish2 or QV computation
- Filtered alignment set satisfying query coverage >90% and identity >98% (≈5868 reads, ~2.93 Gb)
- Per-assembly alignment-identity QV and fraction of 100%-identity reads (qv_report.tsv)

## How to apply

Index the target assembly with minimap2 and align the HiFi reads using the map-hifi preset (minimap2 -cx map-hifi -t 5), which is tuned for high-accuracy PacBio HiFi/CCS reads (v2.19+). Pipe the SAM output through 'samtools sort' to produce a coordinate-sorted BAM and index it; this sorted BAM is the required input format for NextPolish2 polishing. For accuracy evaluation against a ground truth such as the CHM13 T2T genome, first fragment the reference into ~6250 pseudo-long reads of 500 kb, map them against each candidate assembly (original, RM-polished, NextPolish2-polished), then discard alignments with query coverage <=90% or alignment identity <=98% (retaining roughly 5868 reads, ~2.93 Gb) before computing alignment-identity QV. When repetitive regions (e.g., centromeric satellites, segmental duplications) dominate, consider Winnowmap with meryl-counted repetitive k-mers instead, since minimap2 may mask frequent k-mers during seeding in long tandem repeats.

## Related tools

- **minimap2** (Aligns HiFi/CCS genomic reads (or pseudo-long-reads) to the assembly using the map-hifi preset with base-level alignment) — https://github.com/lh3/minimap2/
- **samtools** (Sorts and indexes minimap2 SAM output into the sorted BAM required by NextPolish2 and downstream filtering)
- **Winnowmap** (Alternative long-read mapper for HiFi reads in repetitive regions, used in place of minimap2 when preparing the HiFi mapping file) — https://github.com/marbl/Winnowmap
- **meryl** (Counts repetitive k-mers (k=15, >0.9998 distinct) supplied to Winnowmap's -W option when replacing minimap2)

## Examples

```
minimap2 -cx map-hifi -t 5 asm.fa.gz chm13_pseudo_reads.fa | samtools sort -o hifi.map.sort.bam - && samtools index hifi.map.sort.bam
```

## Evaluation signals

- Alignment files are coordinate-sorted and indexed BAMs (samtools quickcheck / .bai present)
- After filtering (query coverage >90%, identity >98%), approximately 5868 pseudo-reads (~2.93 Gb) are retained per assembly
- NextPolish2-polished assembly shows the fraction of 100%-identity pseudo-reads rising from 9.48% to 9.58%, with only ~14% of reads decreasing in identity
- RM-polished assembly shows a decreased alignment-identity QV relative to the original assembly, while NextPolish2 does not
- Per-assembly QV and 100%-identity fractions are tabulated in qv_report.tsv

## Limitations

- minimap2 masking of frequently occurring k-mers can reduce mapping accuracy within complex repeats (e.g., long tandem repeats), which is why Winnowmap's weighted minimizer sampling exists
- The map-hifi preset requires minimap2 v2.19 or later
- Pseudo-long-read QV evaluation depends on the chosen coverage/identity filter thresholds (90%/98%); stricter or looser cutoffs change the retained read set and QV estimates

## Evidence

- [readme] PacBio HiFi/CCS genomic reads (v2.19+): [evidence span withheld — anchor status: external_doc]
- [methods] Prepare HiFi mapping file with minimap2 or winnowmap: "1. Prepare HiFi mapping file ([winnowmap](https://github.com/marbl/Winnowmap) or [minimap2](https://github.com/lh3/minimap2/))."
- [readme] Winnowmap motivation for repeats: [evidence span withheld — anchor status: not_found]
- [other] Filter thresholds for QV evaluation: [evidence span withheld — anchor status: not_found]
- [methods] NextPolish2 QV improvement result: "`NextPolish2` improved the QV of the assembled genome from 53.8544 to 55.6257 and reduced the number of homozygous high-quality variants (potential errors) from 14,155 to 6,955."
