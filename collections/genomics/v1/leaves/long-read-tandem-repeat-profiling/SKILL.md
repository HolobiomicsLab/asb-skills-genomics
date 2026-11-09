---
name: long-read-tandem-repeat-profiling
description: Use when haplotype-resolved BAM files from ONT or PacBio whole-genome
  sequencing must be scanned de novo for tandem repeats with motif length >6 bp (mini-satellites)
  without prior motif or locus knowledge.
when_to_use_negative:
- The target is unassembled genomic regions such as centromeres and telomeres, which
  TRiCoLOR cannot profile
- The genome is polyploid or contains haploid chromosomes (e.g., the human Y chromosome),
  which TRiCoLOR does not support
- Motifs and loci are already known and only genotyping against a catalogue is needed
  — an alignment-based tool like NCRF or Tandem-genotypes may be simpler
edam_operation: http://edamontology.org/operation_2944
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: TRiCoLOR
  role: End-to-end de novo tandem repeat profiling and genotyping pipeline from haplotype-resolved
    long-read alignments
  repo: https://github.com/davidebolo1993/TRiCoLOR
- name: TRiCoLOR SENSoR
  role: Shannon-entropy-based detection of candidate repetitive segments in sequencing
    reads, output as BED
  repo: https://github.com/davidebolo1993/TRiCoLOR
- name: TRiCoLOR REFER
  role: Resolves repeat motif and multiplicity on candidate regions; --size parameter
    overrides the default motif length (<=6 bp) to profile mini-satellites
  repo: https://github.com/davidebolo1993/TRiCoLOR
- name: minimap2
  role: Aligns haplotype-specific SPOA consensus sequences to the GRCh38 reference
    genome
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/gigascience/giaa101
    title: 'TRiCoLOR: tandem repeat profiling using whole-genome long-read sequencing
      data'
  source_content_hash: sha256:902126fffc9b577e2624f0400e266ada5e77e8680c2392bae0a07c51ae732ac1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/davidebolo1993/TRiCoLOR
  tools:
  - TRiCoLOR
  - TRiCoLOR REFER
  - TRiCoLOR SENSoR
  - minimap2
  edam_operation: http://edamontology.org/operation_2944
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# long-read-tandem-repeat-profiling

## Summary

De novo identification and genotyping of tandem repeats (extended to mini-satellites via the --size parameter) from haplotype-resolved long-read alignments using the TRiCoLOR SENSoR/REFER pipeline.

## When to use

Use when haplotype-resolved BAM files from ONT or PacBio whole-genome sequencing must be scanned de novo for tandem repeats with motif length >6 bp (mini-satellites) without prior motif or locus knowledge.

## When NOT to use

- The target is unassembled genomic regions such as centromeres and telomeres, which TRiCoLOR cannot profile
- The genome is polyploid or contains haploid chromosomes (e.g., the human Y chromosome), which TRiCoLOR does not support
- Motifs and loci are already known and only genotyping against a catalogue is needed — an alignment-based tool like NCRF or Tandem-genotypes may be simpler

## Inputs

- Haplotype-resolved long-read alignment BAM files (ONT or PacBio) from whole-genome sequencing
- Reference genome in FASTA format (GRCh38 used in the study)
- Optional: sample pedigree for Mendelian consistency tracing

## Outputs

- BCF-compliant call set of genotyped tandem repeats
- BED file describing the identified tandem repeats
- Haplotype-specific BAM files with minimap2-aligned SPOA consensus sequences

## How to apply

Run TRiCoLOR SENSoR on haplotype-resolved BAM files, scanning 20 bp non-overlapping sliding windows for Shannon entropy drops <=1.23 supported by >=5 reads and merging nearby windows within 100 bp intervals into a candidate BED. Feed the BED to TRiCoLOR REFER, overriding the default motif-length setting (motifs <=6 bp) via the '--size' parameter so the RegEx matcher identifies perfectly repeated motifs of length >=7 bp with a minimum of 5 perfect repeats. REFER computes haplotype-specific consensus sequences with SPOA (partial order alignment, ~77% error reduction for ONT and ~88% for PB), aligns them to GRCh38 with minimap2, then screens them with an approximate string-matching RegEx allowing imperfect repeats up to edit distance <=1, resolving overlapping predictions with an N-gram model. Retain only TRs >=50 bp that vary between haplotypes or versus the reference, and write the call set in BCF-compliant format plus BED files of identified TRs and haplotype-specific BAM files of aligned consensus sequences.

## Related tools

- **TRiCoLOR** (End-to-end de novo tandem repeat profiling and genotyping pipeline from haplotype-resolved long-read alignments) — https://github.com/davidebolo1993/TRiCoLOR
- **TRiCoLOR SENSoR** (Shannon-entropy-based detection of candidate repetitive segments in sequencing reads, output as BED) — https://github.com/davidebolo1993/TRiCoLOR
- **TRiCoLOR REFER** (Resolves repeat motif and multiplicity on candidate regions; --size parameter overrides the default motif length (<=6 bp) to profile mini-satellites) — https://github.com/davidebolo1993/TRiCoLOR
- **minimap2** (Aligns haplotype-specific SPOA consensus sequences to the GRCh38 reference genome)

## Examples

```
tricolor REFER --bam sample.bam --output calls.bcf --size 100 --threads 8
```

## Evaluation signals

- Pipeline completes without errors on the HGSVC input BAMs (HG00733, HG00514, NA19240 ONT; HG00731/HG00732/HG00733 PB)
- Output BCF and BED files exist and are non-empty
- Every retained call is >=50 bp and varies between haplotypes or versus the reference
- Reported regions derive from merged entropy windows: 20 bp sliding windows with entropy <=1.23 supported by >=5 reads, merged within 100 bp
- Haplotype-specific consensus BAMs are produced with minimap2 mapping qualities consistent with correct GRCh38 placement

## Limitations

- Mini-satellite parameter values via --size are not validated: the paper has extensively applied TRiCoLOR only to micro-satellites, and reports no ground truth for mini-satellite profiling
- No support for polyploid species or haploid chromosomes (e.g., the human Y chromosome)
- Repeats in unassembled regions (centromeres, telomeres) cannot be detected
- Exact entropy threshold and window size values for de novo mini-satellite identification were empirically estimated and only ranges are reported in the discussion

## Evidence

- [methods] identifying motifs (motifs of length ≤6, by default) that are perfectly repeated a minimum number of times (5, by default): "identifying motifs (motifs of length ≤6, by default) that are perfectly repeated a minimum number of times (5, by default)"
- [methods] TRs >=50 bp stored in BCF: "TRs (those ≥50 bp, by default) varying between the haplotypes or the reference are eventually stored in BCF-compliant format"
- [methods] SPOA error reduction: "to compute highly accurate consensus sequences with an approximate error reduction of ~77% and ~88% for ONT and PB, respectively"
- [discussion] No mini-satellite validation: "TRiCoLOR has been extensively applied so far only to micro-satellites"
- [abstract] De novo identification without prior knowledge: "The method can identify repetitive regions in sequencing data without a prior knowledge of their motifs or locations and resolve repeat multiplicity and period size in a haplotype-specific manner."
- [readme] README: installation and documentation pointers: "TRiCoLOR is available to be installed from source or as a [Docker container](https://hub.docker.com/r/davidebolo1993/tricolor)."
