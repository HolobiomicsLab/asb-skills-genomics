---
name: long-read-alignment-identity-qv-calculation
description: Use when you need an alignment-based, reference-independent-of-sequencing
  accuracy metric comparing polishing pipelines on HiFi assemblies, using a high-confidence
  reference split into pseudo-long reads.
when_to_use_negative:
- When k-mer-based QV estimation (e.g., Merqury/yak with short reads) is required
  instead, or when no trusted ground-truth reference genome exists to generate pseudo-long
  reads
- When assemblies are too divergent from the reference for the 98% identity / 90%
  query-coverage filters to retain a meaningful number of alignments
edam_operation: http://edamontology.org/operation_3180
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3673
tools:
- name: minimap2
  role: Aligns 500 kb pseudo-long reads against each assembly with the map-hifi preset
    (-cx map-hifi, -t 5)
  repo: https://github.com/lh3/minimap2/
- name: samtools
  role: Sorts and indexes minimap2 BAM output for each assembly mapping
- name: NextPolish2
  role: Produces the polished assembly variant whose accuracy is assessed by this
    evaluation
  repo: https://github.com/Nextomics/NextPolish2
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gpbjnl/qzad009
    title: 'NextPolish2: A Repeat-aware Polishing Tool for Genomes Assembled Using
      HiFi Long Reads'
  source_content_hash: sha256:9b91417d467d22bc89d261797d5f07a07cdeb71c9cf8369548403ed6acc22c9c
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
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# long-read-alignment-identity-qv-calculation

## Summary

Evaluates genome consensus accuracy by aligning pseudo-long reads derived from a trusted reference (e.g., CHM13 T2T split into 6250 × 500 kb chunks) against polished and unpolished assemblies with minimap2, then computing an alignment-identity QV from the retained alignment identity distribution.

## When to use

Use when you need an alignment-based, reference-independent-of-sequencing accuracy metric comparing polishing pipelines on HiFi assemblies, using a high-confidence reference split into pseudo-long reads.

## When NOT to use

- When k-mer-based QV estimation (e.g., Merqury/yak with short reads) is required instead, or when no trusted ground-truth reference genome exists to generate pseudo-long reads
- When assemblies are too divergent from the reference for the 98% identity / 90% query-coverage filters to retain a meaningful number of alignments

## Inputs

- Ground-truth reference FASTA (e.g., corrected CHM13 T2T genome assembly)
- Candidate assembly FASTA files (original draft and polished variants, e.g., RM-polished and NextPolish2-polished)
- Splitting utility (seqkit or Python/seqio script) and minimap2 with samtools installed

## Outputs

- qv_report.tsv with per-assembly alignment-identity QV and fraction of 100%-identity pseudo-reads
- Filtered BAM alignment files per assembly (~5868 retained alignments, ~2.93 Gb)

## How to apply

Split the ground-truth reference FASTA (CHM13 T2T) into 6250 pseudo-long reads of 500 kb each using a sequence-splitting utility such as seqkit split or a Python script. Align these pseudo-reads against each candidate assembly (original, RM-polished, NextPolish2-polished) with minimap2 using '-cx map-hifi -t 5' (the map-hifi preset suits PacBio HiFi-like accuracy queries) and sort/index each BAM with samtools. Filter alignments, discarding those with query coverage <=90% or alignment identity <=98%, which retains ~5868 reads (~2.93 Gb). For each assembly, compute the QV from the identity distribution of the retained alignments and tabulate per-assembly QV plus the fraction of 100%-identity pseudo-reads in qv_report.tsv. The skill is validated when NextPolish2 polishing raises the 100%-identity read fraction from 9.48% to 9.58% while RM-polishing lowers the alignment-identity QV relative to the original assembly.

## Related tools

- **minimap2** (Aligns 500 kb pseudo-long reads against each assembly with the map-hifi preset (-cx map-hifi, -t 5)) — https://github.com/lh3/minimap2/
- **samtools** (Sorts and indexes minimap2 BAM output for each assembly mapping)
- **NextPolish2** (Produces the polished assembly variant whose accuracy is assessed by this evaluation) — https://github.com/Nextomics/NextPolish2

## Examples

```
minimap2 -cx map-hifi -t 5 asm.np2.fa chm13_pseudo_500kb.fa | samtools sort -o aln.sort.bam
```

## Evaluation signals

- Approximately 5868 pseudo-reads (~2.93 Gb) survive the query coverage >90% and identity >98% filters per assembly
- NextPolish2-polished assembly shows the fraction of 100%-identity pseudo-reads increase from 9.48% to 9.58%
- Only ~14% of reads decrease in mapping identity after NextPolish2 polishing
- RM-polished assembly shows a decreased alignment-identity QV relative to the original assembly
- qv_report.tsv contains one QV row per evaluated assembly with 100%-identity fractions

## Limitations

- The metric depends on the availability and quality of a carefully corrected ground-truth reference (CHM13) and would be biased by reference errors
- Filtering to identity >98% and coverage >90% discards poorly mapped pseudo-reads, so QV is conditional on retained high-quality alignments
- This alignment-identity evaluation is one of several approaches; the paper also employs Merqury, DeepVariant, and Paftools.js for overcorrection assessment

## Evidence

- [results] the RM-polished assembly's QV decreased, whereas NextPolish2 slightly increased the proportion of reads with 100% mapping identity from 9.48% to 9.58%, with only 14% of reads decreasing in identity: [evidence span withheld — anchor status: not_found]
- [methods] Filter the alignments, discarding those with query coverage <=90% or alignment identity <=98%, retaining approximately 5868 reads (~2.93 Gb): [evidence span withheld — anchor status: not_found]
- [readme] minimap2 -ax map-hifi ref.fa pacbio-ccs.fq.gz > aln.sam: [evidence span withheld — anchor status: external_doc]
- [methods] we employed three approaches to evaluate the genome's accuracy before and after polishing: 1. Merqury: To assess the quality value (QV).: "we employed three approaches to evaluate the genome's accuracy before and after polishing:

1. **Merqury:** To assess the quality value (QV)."
