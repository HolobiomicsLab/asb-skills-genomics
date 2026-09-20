---
name: per-read-annotation-generation
description: Use when a long-read metagenomics pipeline must flag ARG-carrying reads
  and emit per-read records (hit coordinates, plasmid status, lineage) that feed Argo's
  read-overlap clustering and host-assignment steps.
when_to_use_negative:
- Input is short-read data — Argo's overlapping-based screening (and this per-read
  annotation step) is designed for long reads; use a short-read profiler such as Lynx
  instead
- The goal is de novo identification of ARG genomic contexts and hosts in new, understudied
  environments — assembly-based (metaFlye + Kraken2) or binning-based (SemiBin2 +
  GTDB-Tk) approaches may be better suited
- Accurate ORF-level annotation or detailed sequence typing of highly similar ARGs
  (e.g., distinguishing blaOXA-1 from blaOXA-1042) is required — Argo clusters similar
  ARGs, lowering resolution
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0091
tools:
- name: Argo
  role: Pipeline that consumes the per-read annotations; its screening step emits
    the per-read annotation structure with remark, hit, plasmid, and lineage fields
  repo: https://github.com/xinehc/argo
- name: DIAMOND
  role: Performs frameshift-aware DNA-to-protein (blastx) alignment of long reads
    against the SARG+ protein database to flag ARG-containing reads
  repo: https://github.com/xinehc/argo
- name: minimap2
  role: Approximate read overlapping ('-x ava-ont') supplies 'dv' per-base divergence
    tags used to set the adaptive identity cutoff; 'map-ont' indexes nucleotide database
    files
  repo: https://github.com/xinehc/argo
- name: SARG+
  role: Curated ARG reference database (NDARO/CARD/SARG plus evidence-matched RefSeq
    homologs) that defines the 'hit' type/subtype annotations
  repo: https://github.com/xinehc/sarg-curation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-57088-y
    title: Species-resolved profiling of antibiotic resistance genes in complex metagenomes
      through long-read overlapping with Argo
  source_content_hash: sha256:a6732a49a0d43b22b9e4cfa0e979f5efd373ebd9e1aa5ccc71b14ab4b4f5accd
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
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# per-read-annotation--generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates a per-read ARG annotation structure (e.g., Argo's example.sarg JSON) by screening quality-controlled long reads with DIAMOND frameshift-aware blastx against the SARG+ protein database, recording each flagged read's ARG hits, plasmid flag, and lineage for downstream species-resolved ARG profiling.

## When to use

Use when a long-read metagenomics pipeline must flag ARG-carrying reads and emit per-read records (hit coordinates, plasmid status, lineage) that feed Argo's read-overlap clustering and host-assignment steps.

## When NOT to use

- Input is short-read data — Argo's overlapping-based screening (and this per-read annotation step) is designed for long reads; use a short-read profiler such as Lynx instead
- The goal is de novo identification of ARG genomic contexts and hosts in new, understudied environments — assembly-based (metaFlye + Kraken2) or binning-based (SemiBin2 + GTDB-Tk) approaches may be better suited
- Accurate ORF-level annotation or detailed sequence typing of highly similar ARGs (e.g., distinguishing blaOXA-1 from blaOXA-1042) is required — Argo clusters similar ARGs, lowering resolution

## Inputs

- Quality-controlled long-read FASTA (ONT or PacBio), e.g., the 10,000-read QC'd example FASTA (Zenodo record 12571849, example.fa.gz)
- Indexed SARG+ protein database (database/sarg DIAMOND index built from database/sarg.fa in the Zenodo record 20451539 database bundle)
- Median per-base sequence divergence estimated from minimap2 approximate overlaps of the first 10,000 reads

## Outputs

- Per-read ARG annotation JSON (e.g., example.sarg) with records containing 'remark', 'hit', 'plasmid', and 'lineage' fields
- Per-read ARG hit details: type|subtype|reference sequence identifiers with alignment coordinates
- Adaptive identity cutoff value (e.g., 77.45) used for screening

## How to apply

First estimate an adaptive identity cutoff from minimap2's approximate read overlaps ('-x ava-ont'), taking the median per-base sequence divergence ('dv' tags) of the first 10,000 reads and computing 90 - 2.5 * 100 * median(dv) — e.g., 77.45 for a median divergence of 0.0502 — so that sensitivity is comparable across ONT and PacBio platforms. Then run DIAMOND blastx (frameshift-aware, '--evalue 1e-15 --range-culling --frameshift 15 --range-cover 25 --max-target-seqs 25 --max-hps 0' style parameters) of reads against the indexed SARG+ protein database ('diamond makedb --in database/sarg.fa --db database/sarg') to detect ARG-carrying reads despite Nanopore frameshift errors. For each flagged read, record its ARG hits as type|subtype|reference-sequence identifiers with alignment coordinates, a plasmid flag, and a lineage, and write the per-read annotation structure (with 'remark', 'hit', 'plasmid', and 'lineage' fields). SARG+ is used as the reference because it augments experimentally validated sequences (NDARO, CARD, SARG) with RefSeq proteins sharing the same PGAP annotation evidence, enabling stringent cutoffs while maintaining sensitivity.

## Related tools

- **Argo** (Pipeline that consumes the per-read annotations; its screening step emits the per-read annotation structure with remark, hit, plasmid, and lineage fields) — https://github.com/xinehc/argo
- **DIAMOND** (Performs frameshift-aware DNA-to-protein (blastx) alignment of long reads against the SARG+ protein database to flag ARG-containing reads) — https://github.com/xinehc/argo
- **minimap2** (Approximate read overlapping ('-x ava-ont') supplies 'dv' per-base divergence tags used to set the adaptive identity cutoff; 'map-ont' indexes nucleotide database files) — https://github.com/xinehc/argo
- **SARG+** (Curated ARG reference database (NDARO/CARD/SARG plus evidence-matched RefSeq homologs) that defines the 'hit' type/subtype annotations) — https://github.com/xinehc/sarg-curation

## Examples

```
diamond blastx --db database/sarg --query example.fa --evalue 1e-15 --range-culling --frameshift 15 --range-cover 25 --max-target-seqs 25 --max-hsps 0
```

## Evaluation signals

- On the 10,000-read example dataset, the run reports 'candidate HSPs: 10000 | ARG-containing reads: 579' at identity cutoff 77.45
- The emitted example.sarg file exists and parses as valid JSON
- Every per-read record contains the required 'remark', 'hit', 'plasmid', and 'lineage' fields
- Hit identifiers follow the type|subtype|reference-sequence format with alignment coordinates
- The adaptive identity cutoff (90 - 2.5 * 100 * median dv) equals 77.45 when median divergence is 0.0502

## Limitations

- Argo is inherently reference-based and may face classification issues with unseen species or novel plasmid syntenies
- Point-mutation-derived ARGs are excluded from the database because SNPs cannot be reliably distinguished from long-read sequencing errors
- Highly similar ARGs are clustered (e.g., blaOXA-1 and blaOXA-1042 grouped as blaOXA), which lowers resolution for ORF-based annotation or detailed sequence typing

## Evidence
