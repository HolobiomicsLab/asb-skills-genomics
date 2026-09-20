---
name: multi-tool-transcript-overlap-set-analysis
description: Use when comparing transcriptome reconstruction output from multiple
  LRTS tools (TALON, FLAIR, IsoQuant, Bambu, IsoTools) on the same BAM alignment,
  especially to assess how filtering criteria affect uniquely identified transcripts.
when_to_use_negative:
- The input is short-read RNA-seq data rather than aligned full-length long-read transcripts
  — IsoTools is defined as a python module for Long Read Transcriptome Sequencing
  (LRTS) analysis
- The goal is alternative splicing event detection or differential splicing analysis
  rather than cross-tool transcriptome reconstruction comparison
- Only a single tool's transcript set is available, so no multi-way overlap can be
  computed
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: IsoTools
  role: Python LRTS framework used for transcriptome reconstruction under permissive
    and balanced filter configurations (add_filter with DEFAULT_GENE_FILTER / DEFAULT_TRANSCRIPT_FILTER)
  repo: https://github.com/MatthiasLienhard/isotools
- name: TALON
  role: Reconstructs a transcript set from the shared LRTS BAM using default filtering
    parameters for the comparison
- name: FLAIR
  role: Reconstructs a transcript set from the shared LRTS BAM using default filtering
    parameters for the comparison
- name: IsoQuant
  role: Reconstructs a transcript set from the shared LRTS BAM using default filtering
    parameters for the comparison
- name: Bambu
  role: Reconstructs a transcript set from the shared LRTS BAM using default filtering
    parameters for the comparison
- name: Python
  role: Language in which IsoTools is implemented and in which the overlap-set computation
    and table writing are scripted
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad364
    title: 'IsoTools: a flexible workflow for long-read transcriptome sequencing analysis'
  source_content_hash: sha256:9daa8e4625fa9028ddedc491fdc1d02e746d7a4c2e118033215b7ba97666818e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MatthiasLienhard/isotools
  tools:
  - Bambu
  - FLAIR
  - IsoQuant
  - IsoTools
  - Python
  - TALON
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# multi-tool-transcript-overlap-set-analysis

## Summary

A benchmarking skill that reconstructs transcriptomes from a shared long-read RNA-seq (LRTS) BAM alignment using five tools (TALON, FLAIR, IsoQuant, Bambu, IsoTools) and computes multi-way transcript overlap sets to quantify shared and tool-unique transcript recovery. It exposes how filtering stringency — e.g. IsoTools permissive vs balanced filters — changes the number of uniquely recovered transcripts.

## When to use

Use when comparing transcriptome reconstruction output from multiple LRTS tools (TALON, FLAIR, IsoQuant, Bambu, IsoTools) on the same BAM alignment, especially to assess how filtering criteria affect uniquely identified transcripts.

## When NOT to use

- The input is short-read RNA-seq data rather than aligned full-length long-read transcripts — IsoTools is defined as a python module for Long Read Transcriptome Sequencing (LRTS) analysis
- The goal is alternative splicing event detection or differential splicing analysis rather than cross-tool transcriptome reconstruction comparison
- Only a single tool's transcript set is available, so no multi-way overlap can be computed

## Inputs

- LRTS BAM alignment of full-length transcripts (PacBio SequelII hepatocyte data)
- Reference annotation in gff3/gtf format
- IsoTools DEFAULT_GENE_FILTER and DEFAULT_TRANSCRIPT_FILTER settings exported from the package
- Reconstructed transcript sets from TALON, FLAIR, IsoQuant, and Bambu (default filtering)

## Outputs

- tool_transcript_overlap_table.tsv (multi-way overlap counts: five-tool intersection plus per-tool unique sets)
- Filtered IsoTools transcriptome object (saved .pkl) for permissive and balanced configurations

## How to apply

Run TALON, FLAIR, IsoQuant, and Bambu on the same LRTS BAM alignment using each tool's default filtering parameters, alongside reference annotation in gff3/gtf format. Reconstruct the transcriptome with IsoTools twice: once with the permissive filter configuration (add_filter using the exported DEFAULT_GENE_FILTER / DEFAULT_TRANSCRIPT_FILTER settings) to maximize novel transcript recovery, and once with the balanced filter, under which FRAGMENTS-class reads not using a reference TSS or PAS are excluded. Parse all five transcript sets and compute multi-way overlap sets (five-tool intersection plus per-tool unique sets), then tabulate counts. Judge correctness by the expected invariants: 20,642 transcripts shared by all five tools, and uniquely-IsoTools counts dropping from 72,733 (permissive) to 5,564 (balanced, <10% unique).

## Related tools

- **IsoTools** (Python LRTS framework used for transcriptome reconstruction under permissive and balanced filter configurations (add_filter with DEFAULT_GENE_FILTER / DEFAULT_TRANSCRIPT_FILTER)) — https://github.com/MatthiasLienhard/isotools
- **TALON** (Reconstructs a transcript set from the shared LRTS BAM using default filtering parameters for the comparison)
- **FLAIR** (Reconstructs a transcript set from the shared LRTS BAM using default filtering parameters for the comparison)
- **IsoQuant** (Reconstructs a transcript set from the shared LRTS BAM using default filtering parameters for the comparison)
- **Bambu** (Reconstructs a transcript set from the shared LRTS BAM using default filtering parameters for the comparison)
- **Python** (Language in which IsoTools is implemented and in which the overlap-set computation and table writing are scripted)

## Examples

```
from isotools import Transcriptome; isoseq = Transcriptome.from_reference('ref.gff.gz'); isoseq.add_sample_from_bam('hepatocyte.bam', sample_name='CTL', group='CTL', platform='SequelII'); isoseq.add_filter(); isoseq.save('example_isotools.pkl')
```

## Evaluation signals

- The overlap table reports 20,642 transcripts shared by all five tools
- Uniquely-IsoTools transcripts drop from 72,733 under the permissive filter to 5,564 under the balanced filter (less than 10% unique)
- tool_transcript_overlap_table.tsv exists and contains five-tool intersection plus per-tool unique set counts
- The balanced-filter IsoTools run excludes FRAGMENTS-class reads that do not use a reference TSS or PAS
- IsoTools transcriptome objects import cleanly from the LRTS BAM plus gff3/gtf reference annotation before reconstruction

## Limitations

- Uniquely recovered transcript counts are highly sensitive to filtering criteria: the proportion of unique IsoTools transcripts drops from 72,733 to 5,564 (under 10%) when switching from permissive to balanced filtering
- The balanced filter tags FRAGMENTS only when reads do not use a reference TSS or PAS, so reads relying on non-reference boundaries drive the inflated unique counts in the permissive setting
- Each tool's default filtering parameters differ, so cross-tool comparisons conflate reconstruction with tool-specific filtering

## Evidence

- [abstract] IsoTools, a Python-based LRTS analysis framework that offers a wide range of functionality for transcriptome reconstruction and quantification of transcripts: "We introduce IsoTools, a Python-based LRTS analysis framework that offers a wide range of functionality for transcriptome reconstruction and quantification of transcripts."
- [intro] Import of LRTS bam files and gff3/gtf reference annotation: "Import of LRTS bam files (aligned full length transcripts). Import of reference annotation in gff3/gtf format."
- [methods] add_filter and save usage in methods: "isoseq.add_filter()
    isoseq.save('example_isotools.pkl')"
- [discussion] Default FRAGMENTS filter tags reads without reference TSS or PAS: "Default filter of FRAGMENTS now only tags reads that do not use a reference TSS or PAS"
- [readme] IsoTools is a python module for LRTS analysis: "IsoTools is a python module for Long Read Transcriptome Sequencing (LRTS) analysis."
