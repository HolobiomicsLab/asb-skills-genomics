---
name: pseudo-read-fragmentation-from-reference-genome
description: Use when you need a ground-truth-based accuracy comparison of two or
  more genome assemblies (original vs polished variants) and lack sequencing reads,
  by simulating long reads from a trusted reference such as CHM13 T2T.
when_to_use_negative:
- You have real sequencing reads (e.g. PacBio HiFi) or short reads available — use
  k-mer-based QV evaluation (Merqury/yak) instead of simulated pseudo-reads
- The reference itself is unverified — error correction performance estimates will
  only be as good as the ground-truth assembly
- Assemblies are from a different species/individual than the reference, since divergent
  haplotypes will masquerade as low alignment identity
edam_operation: http://edamontology.org/operation_3180
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3693
tools:
- name: minimap2
  role: Aligns 500 kb pseudo-long-reads against each candidate assembly with the map-hifi
    preset (-cx map-hifi, -t 5)
  repo: https://github.com/lh3/minimap2/
- name: samtools
  role: Sorts and indexes the pseudo-read mapping BAMs before filtering
- name: NextPolish2
  role: One of the polished assemblies evaluated via the pseudo-read alignment-identity
    QV; raises the 100%-identity read fraction from 9.48% to 9.58%
  repo: https://github.com/Nextomics/NextPolish2
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gpbjnl/qzad009
    title: 'NextPolish2: A Repeat-aware Polishing Tool for Genomes Assembled Using
      HiFi Long Reads'
  source_content_hash: sha256:b6ce117e7eacb56d6e838b466bd326d86567d892729b92b702e64bbe8e2530ea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - NextPolish2
  - minimap2
  - samtools
  edam_operation: http://edamontology.org/operation_3180
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3693
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# pseudo-read-fragmentation-from-reference-genome

## Summary

Fragments a trusted reference genome (e.g. the corrected CHM13 T2T assembly) into fixed-size pseudo-long reads (6250 reads × 500 kb) so the reference can be aligned back against candidate polished assemblies to measure alignment-identity-based consensus accuracy (QV).

## When to use

Use when you need a ground-truth-based accuracy comparison of two or more genome assemblies (original vs polished variants) and lack sequencing reads, by simulating long reads from a trusted reference such as CHM13 T2T.

## When NOT to use

- You have real sequencing reads (e.g. PacBio HiFi) or short reads available — use k-mer-based QV evaluation (Merqury/yak) instead of simulated pseudo-reads
- The reference itself is unverified — error correction performance estimates will only be as good as the ground-truth assembly
- Assemblies are from a different species/individual than the reference, since divergent haplotypes will masquerade as low alignment identity

## Inputs

- Ground-truth reference genome FASTA (corrected CHM13 T2T genome assembly)
- Candidate assembly FASTAs (original, RM-polished, NextPolish2-polished)

## Outputs

- 6250 pseudo-long-reads of 500 kb each (split reference FASTA)
- Sorted and indexed BAM mappings of pseudo-reads against each assembly
- qv_report.tsv with per-assembly alignment-identity QV and fraction of 100%-identity pseudo-reads

## How to apply

Split the ground-truth reference FASTA (e.g. corrected CHM13 T2T) into 6250 pseudo-long reads of 500 kb each using a sequence-splitting utility such as seqkit split or a Python/Bio.SeqIO script. Align the 500 kb pseudo-reads against each candidate assembly (original, RM-polished, NextPolish2-polished) with minimap2 using the -cx map-hifi preset and -t 5, then sort and index each mapping with samtools. Filter alignments to retain only high-confidence mappings, discarding those with query coverage <=90% or alignment identity <=98%, which yields ~5868 reads (~2.93 Gb). From the identity distribution of retained alignments per assembly, compute the alignment-identity QV and the fraction of 100%-identity pseudo-reads, tabulated in qv_report.tsv. The rationale is that a correct polishing pipeline should raise, not lower, the identity of pseudo-reads derived from the ground truth when mapped back to the polished assembly.

## Related tools

- **minimap2** (Aligns 500 kb pseudo-long-reads against each candidate assembly with the map-hifi preset (-cx map-hifi, -t 5)) — https://github.com/lh3/minimap2/
- **samtools** (Sorts and indexes the pseudo-read mapping BAMs before filtering)
- **NextPolish2** (One of the polished assemblies evaluated via the pseudo-read alignment-identity QV; raises the 100%-identity read fraction from 9.48% to 9.58%) — https://github.com/Nextomics/NextPolish2

## Examples

```
seqkit split --size 500000 chm13.fa; minimap2 -cx map-hifi -t 5 asm.np2.fa pseudo_reads.fa | samtools sort -o pseudo.map.sort.bam
```

## Evaluation signals

- Fragmentation produces exactly 6250 pseudo-reads of 500 kb each from the CHM13 reference
- Alignment filtering retains approximately 5868 reads (~2.93 Gb) after discarding alignments with query coverage <=90% or identity <=98%
- NextPolish2-polished assembly raises the fraction of 100%-identity pseudo-reads from 9.48% to 9.58%
- RM-polished assembly shows a decreased alignment-identity QV relative to the original assembly, consistent with the expected validation criterion
- qv_report.tsv contains per-assembly QV and 100%-identity read fraction for all three assemblies

## Limitations

- Only ~14% of pseudo-reads decreased in identity after NextPolish2 polishing, but the small magnitude of the 9.48%→9.58% improvement means this metric is sensitive to alignment filtering thresholds
- QV estimated from alignment identity reflects only errors within alignable, high-confidence regions (coverage >90%, identity >98%); errors in unaligned or poorly aligned regions are excluded
- Pseudo-reads from a haploid CHM13 reference cannot assess polishing-induced haplotype switch errors in diploid assemblies

## Evidence

- [other] Fragment the CHM13 T2T reference FASTA into 6250 pseudo-long reads of 500 kb each using a sequence-splitting utility (e.g. seqkit split or a Python/seqio script).: [evidence span withheld — anchor status: not_found]
- [other] Align the 500 kb pseudo-reads against each of the three assemblies with Minimap2 (-cx map-hifi, -t 5) and sort/index each mapping with samtools.: [evidence span withheld — anchor status: not_found]
- [other] Filter the alignments, discarding those with query coverage <=90% or alignment identity <=98%, retaining approximately 5868 reads (~2.93 Gb).: [evidence span withheld — anchor status: not_found]
- [other] NextPolish2 slightly increased the proportion of reads with 100% mapping identity from 9.48% to 9.58%, with only 14% of reads decreasing in identity.: [evidence span withheld — anchor status: not_found]
- [readme] genomes obtained using HiFi long reads have considerably higher qualities, however, they still contain a handful of assembly errors: "Although genomes obtained using HiFi long reads have considerably higher qualities, however, they still contain a handful of assembly errors"
