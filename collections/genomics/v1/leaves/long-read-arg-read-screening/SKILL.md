---
name: long-read-arg-read-screening
description: Use when quality-controlled Nanopore/PacBio metagenomic reads must be
  screened for ARG-carrying reads before species-resolved host assignment, and the
  identity threshold must adapt to platform-specific sequencing error.
when_to_use_negative:
- Input reads are short second-generation reads — Argo is built on long-read overlapping
  and Lynx is the short-read counterpart
- The goal is de novo identification of ARG genomic contexts or hosts in new, understudied
  environments, where assembly-based (metaFlye + Kraken2) or binning-based (SemiBin2
  + GTDB-Tk) methods are better suited
- Detecting ARGs arising from point mutations, since Argo's database consistently
  excludes point-mutation-derived ARGs due to SNP-vs-error ambiguity
edam_operation: http://edamontology.org/operation_0338
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0798
tools:
- name: Argo
  role: Long-read ARG profiler whose first screening layer implements the adaptive
    cutoff and per-read ARG flagging described here
  repo: https://github.com/xinehc/argo
- name: DIAMOND
  role: Runs frameshift-aware blastx alignment of reads against the SARG+ protein
    database with --evalue 1e-15, --range-culling, --frameshift 15
- name: minimap2
  role: Approximate read overlapping ('-x ava-ont') to obtain dv tags for estimating
    the adaptive identity cutoff; '-x map-ont' to index nucleotide references
- name: SARG+
  role: Manually curated ARG database (reference + RefSeq-evidence extension) serving
    as the alignment target
  repo: https://github.com/xinehc/sarg-curation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-57088-y
    title: Species-resolved profiling of antibiotic resistance genes in complex metagenomes
      through long-read overlapping with Argo
  source_content_hash: sha256:9226e7a3eba62436cc462d31227552b06265e6694a659faf44e44115f1882174
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/xinehc/argo
  repo_url: https://github.com/xinehc/argo
  tools:
  - Argo
  - DIAMOND
  - SARG+
  - minimap2
  edam_operation: http://edamontology.org/operation_0338
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0798
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# long-read-arg-read-screening

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Argo's initial screening layer flags ARG-containing quality-controlled long reads by aligning them to the SARG+ protein database with DIAMOND's frameshift-aware blastx, using an adaptive identity cutoff derived from read-overlap divergence so profiles stay comparable across Nanopore and PacBio platforms.

## When to use

Use when quality-controlled Nanopore/PacBio metagenomic reads must be screened for ARG-carrying reads before species-resolved host assignment, and the identity threshold must adapt to platform-specific sequencing error.

## When NOT to use

- Input reads are short second-generation reads — Argo is built on long-read overlapping and Lynx is the short-read counterpart
- The goal is de novo identification of ARG genomic contexts or hosts in new, understudied environments, where assembly-based (metaFlye + Kraken2) or binning-based (SemiBin2 + GTDB-Tk) methods are better suited
- Detecting ARGs arising from point mutations, since Argo's database consistently excludes point-mutation-derived ARGs due to SNP-vs-error ambiguity

## Inputs

- Quality-controlled long-read FASTA/FASTQ (Nanopore or PacBio metagenomic reads)
- SARG+ protein database (sarg.fa from xinehc/sarg-curation, Zenodo database.tar.gz record 20451539)
- DIAMOND protein database index built with 'diamond makedb'

## Outputs

- Per-read annotation structure (e.g. example.sarg.) with 'remark', 'hit', 'plasmid', and 'lineage' fields for each ARG-flagged read
- Set of ARG-containing reads with type|subtype|reference sequence identifiers and alignment coordinates
- Adaptive identity cutoff value computed from median overlap divergence

## How to apply

First overlap reads with minimap2's approximate mapping mode ('-x ava-ont') to obtain per-base sequence divergence ('dv') tags, and compute the median divergence from overlaps of the first 10,000 reads. Set the adaptive identity cutoff as 90 - 2.5 * 100 * median(dv) (e.g. 77.45 for a median divergence of 0.0502 in the 10,000-read Zenodo example, record 12571849). Then run DIAMOND blastx of the reads against the indexed SARG+ protein database with 'blastx --evalue 1e-15 --range-culling --frameshift 15 --range-cover 25 --max-target-seqs 25 --max-hps 0', which is frameshift-aware to tolerate long-read indel errors. For each flagged read record the ARG hits (type|subtype|reference identifiers with coordinates), plasmid flag, and lineage into a per-read annotation structure. The rationale is that filtering ARG-free reads bypasses unnecessary downstream alignments, and adapting the cutoff to median overlap divergence keeps ARG profiles comparable across sequencing platforms.

## Related tools

- **Argo** (Long-read ARG profiler whose first screening layer implements the adaptive cutoff and per-read ARG flagging described here) — https://github.com/xinehc/argo
- **DIAMOND** (Runs frameshift-aware blastx alignment of reads against the SARG+ protein database with --evalue 1e-15, --range-culling, --frameshift 15)
- **minimap2** (Approximate read overlapping ('-x ava-ont') to obtain dv tags for estimating the adaptive identity cutoff; '-x map-ont' to index nucleotide references)
- **SARG+** (Manually curated ARG database (reference + RefSeq-evidence extension) serving as the alignment target) — https://github.com/xinehc/sarg-curation

## Examples

```
minimap2 -x ava-ont reads.fa reads.fa | awk '{print "dv=" $9}' | sort | head -n 10000 > overlaps.dv; diamond blastx --db database/sarg --query reads.fa --evalue 1e-15 --range-culling --frameshift 15 --range-cover 25 --max-target-seqs 25 --max-hsps 0
```

## Evaluation signals

- On the Zenodo 10,000-read example (record 12571849) with database record 20451539, the run reports 'candidate HSPs: 10000 | ARG-containing reads: 579' at identity cutoff 77.45
- The computed identity cutoff equals 90 - 2.5 * 100 * median(dv); for the example's median divergence of 0.0502 this must be 77.45
- The emitted per-read annotation file exists, parses as valid JSON, and every per-read record contains 'remark', 'hit', 'plasmid', and 'lineage' fields
- Database indices were built correctly: 'diamond makedb' on prot.fa and sarg.fa, and minimap2 '-x map-ont -d' on the nucleotide FASTA files
- Each flagged read's hit field carries type|subtype|reference sequence identifiers with alignment coordinates

## Limitations

- Argo is inherently reference-based and may face classification issues with unseen species or novel plasmid syntenies
- Argo's database consistently excludes ARGs originating from point mutations because SNPs cannot be reliably distinguished from sequencing errors in read-based profiling
- Highly similar ARGs are clustered (e.g., blaOXA-1 and blaOXA-1042 grouped as blaOXA), lowering resolution for ORF-based annotation or detailed sequence typing
- The screening step requires long reads; insufficient coverage leads to unstable normalization and inadequate read overlaps

## Evidence

- [other] adaptive identity cutoff: [evidence span withheld — anchor status: not_found]
- [readme] SARG+ database composition: [evidence span withheld — anchor status: not_found]
- [other] platform comparability: [evidence span withheld — anchor status: not_found]
