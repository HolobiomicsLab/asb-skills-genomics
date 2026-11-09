---
name: long-read-structural-variant-benchmarking
description: Use when you need to quantitatively compare long-read nanopore TE/SV
  detection tools against an established truth set — e.g. replicating Table 1 of the
  RetroInspector paper where PALMER (F1 60.72%) and GraffiTE (F1 48.40%) are benchmarked
  against RetroInspector's cuteSV+Sniffles2 callset (F1 83.
when_to_use_negative:
- Input reads are short-read (Illumina) — RetroInspector and this benchmark target
  nanopore long reads; MELT is the short-read alternative named in the article.
- You need to evaluate a single tool's output quality without a truth-set comparison
  — the skill's value is the SURVIVOR-based head-to-head scoring, not individual callset
  generation.
- You intend to include xTea in the comparison — the authors report they were not
  able to use it, so no benchmark protocol exists for it here.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0091
tools:
- name: RetroInspector
  role: 'Snakemake pipeline under benchmark: minimap2 alignment, cuteSV+Sniffles2
    variant calling with default minimum read support of 3, run on 31 threads with
    --stats timing'
  repo: https://github.com/javiercguard/retroinspector
- name: PALMER
  role: Comparator tool, executed once per TE type (4 per sample) with GNU parallel
    on 20 threads; output converted to VCF
- name: GraffiTE
  role: Comparator tool using graph genomes ('pangenie' method), fed Dfam 3.8 libraries
    prepared with the FamDB toolkit and nanopore reads via --longreads/--ont
- name: SURVIVOR
  role: Matches each tool's called variants against the 5,630-insertion truth set
    with 100 bp (strict) and 500 bp (permissive) distance margins
- name: FamDB
  role: Converts the complete Dfam 3.8 release into GraffiTE's required format, extracting
    human and ancestor repeats
  repo: https://github.com/Dfam-consortium/FamDB
- name: minimap2
  role: Aligns basecalled nanopore reads against GRCh38 inside the RetroInspector
    pipeline
  repo: https://github.com/lh3/minimap2
- name: cuteSV
  role: Structural variant caller within RetroInspector whose combination with Sniffles2
    agreed best with the HGSV truth set
  repo: https://github.com/tjiangHIT/cuteSV
- name: Sniffles2
  role: Structural variant caller within RetroInspector used in the strict callset
    configuration scored at 83.89% F1
  repo: https://github.com/fritzsedlazeck/Sniffles
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-025-98847-7
    title: Advanced analysis of retrotransposon variation in the human genome with
      nanopore sequencing using RetroInspector
  source_content_hash: sha256:085fe57ebc0e51f75bcdea255993f59a48408324bd2cf7d50a9e881f6a133d7e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/javiercguard/retroinspector
  repo_url: https://github.com/javiercguard/retroinspector
  tools:
  - FamDB
  - FamDB toolkit
  - GNU parallel
  - GraffiTE
  - PALMER
  - PanGenIE
  - RetroInspector
  - SURVIVOR
  - Sniffles2
  - cuteSV
  - minimap2
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# long-read-structural-variant-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A reproducible head-to-head benchmarking protocol that compares long-read TE insertion/structural variant callers (RetroInspector, PALMER, GraffiTE) against the 5,630-insertion HGSVC truth set using SURVIVOR interval matching at 100 bp and 500 bp margins, yielding precision, recall, and F1 per tool.

## When to use

Use when you need to quantitatively compare long-read nanopore TE/SV detection tools against an established truth set — e.g. replicating Table 1 of the RetroInspector paper where PALMER (F1 60.72%) and GraffiTE (F1 48.40%) are benchmarked against RetroInspector's cuteSV+Sniffles2 callset (F1 83.89%) on HGSV samples.

## When NOT to use

- Input reads are short-read (Illumina) — RetroInspector and this benchmark target nanopore long reads; MELT is the short-read alternative named in the article.
- You need to evaluate a single tool's output quality without a truth-set comparison — the skill's value is the SURVIVOR-based head-to-head scoring, not individual callset generation.
- You intend to include xTea in the comparison — the authors report they were not able to use it, so no benchmark protocol exists for it here.

## Inputs

- Three HGSV nanopore FASTQ files (HG00733, HG00514, NA19240)
- GRCh38 human reference genome FASTA (hg38)
- HGSVC truth set of 5,630 TE insertions across the three samples
- Complete Dfam release 3.8 (for GraffiTE library preparation via FamDB)
- PALMER output per TE type, converted to VCF

## Outputs

- Per-tool variant callsets in VCF format (RetroInspector, PALMER, GraffiTE)
- SURVIVOR truth-set match results at 100 bp and 500 bp distance margins
- Precision, recall, and F1 table per tool (e.g. RetroInspector strict: 80.90% / 87.10% / 83.89%)
- Per-sample runtime comparison across tools and pipeline stages

## How to apply

Run the RetroInspector Snakemake pipeline on the three HGSV nanopore FASTQ files (HG00733, HG00514, NA19240) against GRCh38 using minimap2 alignment and cuteSV+Sniffles2 variant calling with the default minimum read support of 3, on 31 threads, recording time with the --stats flag. Run PALMER on the same samples (one execution per TE type, 4 per sample), filtering to variants with at least 1 high-confidence read and 2 supporting reads (10% of depth coverage, rounded down), orchestrated with GNU parallel on 20 threads, then convert output to VCF with the repository's script. Run GraffiTE per its documentation: convert a complete Dfam 3.8 release to its required format with the FamDB toolkit (v1.0.2), extracting human and ancestor repeats; select GRCh38, choose 'pangenie' as graph method, and supply FASTQ paths via --longreads with 'ont'. Match each tool's variants to the truth set with SURVIVOR using a 100 bp distance margin, repeating at 500 bp as a permissive setting, then compute precision, recall, and F1 per tool; exclude GraffiTE from the runtime table because it used 96 threads despite being assigned 32.

## Related tools

- **RetroInspector** (Snakemake pipeline under benchmark: minimap2 alignment, cuteSV+Sniffles2 variant calling with default minimum read support of 3, run on 31 threads with --stats timing) — https://github.com/javiercguard/retroinspector
- **PALMER** (Comparator tool, executed once per TE type (4 per sample) with GNU parallel on 20 threads; output converted to VCF)
- **GraffiTE** (Comparator tool using graph genomes ('pangenie' method), fed Dfam 3.8 libraries prepared with the FamDB toolkit and nanopore reads via --longreads/--ont)
- **SURVIVOR** (Matches each tool's called variants against the 5,630-insertion truth set with 100 bp (strict) and 500 bp (permissive) distance margins)
- **FamDB** (Converts the complete Dfam 3.8 release into GraffiTE's required format, extracting human and ancestor repeats) — https://github.com/Dfam-consortium/FamDB
- **minimap2** (Aligns basecalled nanopore reads against GRCh38 inside the RetroInspector pipeline) — https://github.com/lh3/minimap2
- **cuteSV** (Structural variant caller within RetroInspector whose combination with Sniffles2 agreed best with the HGSV truth set) — https://github.com/tjiangHIT/cuteSV
- **Sniffles2** (Structural variant caller within RetroInspector used in the strict callset configuration scored at 83.89% F1) — https://github.com/fritzsedlazeck/Sniffles

## Examples

```
snakemake --use-conda --config referenceGenome="/home/yourname/hg38.fa" outputPath="/home/yourname/experiment_results" fastq_directory="/home/yourname/data/samples"
```

## Evaluation signals

- RetroInspector's strict cuteSV+Sniffles2 callset (minimum 3 supporting reads) reproduces precision 80.90% (85.63% over all SVs), recall 87.10%, F1 83.89% against the 5,630-insertion truth set.
- PALMER metrics land at F1 60.72%, precision 71.77%, recall 52.61%; GraffiTE at recall 96.18%, precision 32.33%, F1 48.40% — reproducing the precision/recall gaps in Table 1.
- SURVIVOR comparisons are run twice: strict 100 bp margin and permissive 500 bp margin; rankings and relative gaps should hold across both settings.
- Per-sample runtime ordering shows RetroInspector faster than PALMER, with alignment the slowest internal stage (mean 231.88 ± 32 min per sample); GraffiTE is excluded from timing because it used 96 threads when assigned 32.
- PALMER filtering is verifiable: retained variants have at least 1 high-confidence read and 2 supporting reads, where 2 equals 10% of depth coverage rounded down.

## Limitations

- GraffiTE consumed 96 threads despite being assigned 32, so it is excluded from the runtime comparison — thread-controlled timing is not comparable across all three tools.
- PALMER's low precision may stem from an incomplete truth set rather than genuine false positives, so precision figures should be interpreted cautiously.
- The public HGSVC data was basecalled with an older basecaller, so sequences may be imperfect, particularly for homopolymers, affecting all tools equally but capping achievable recall.
- No software version numbers are stated for cuteSV, Sniffles2, PALMER, or GraffiTE in the article, complicating exact replication.

## Evidence

- [results] PALMER and GraffiTE benchmark numbers: [evidence span withheld — anchor status: not_found]
- [methods] SURVIVOR truth-set matching procedure: [evidence span withheld — anchor status: not_found]
