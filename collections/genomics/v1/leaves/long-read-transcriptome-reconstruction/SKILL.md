---
name: long-read-transcriptome-reconstruction
description: Use when you have BAM files of aligned full-length LRTS transcripts (e.g.
  PacBio SequelII hepatocyte CTL/VPA data) and a gff3/gtf reference annotation, and
  need gene/transcript reconstruction with novel-transcript classification and tunable
  filtering.
when_to_use_negative:
- Input is short-read RNA-seq data rather than BAM files of aligned full-length LRTS
  transcripts
- The goal is only differential alternative splicing between prebuilt transcript models,
  not de novo transcriptome reconstruction from alignments
- No reference annotation in gff3/gtf format is available for the organism
edam_operation: http://edamontology.org/operation_3258
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: IsoTools
  role: 'Primary Python framework: imports LRTS BAMs and gff3/gtf reference, computes
    QC, classifies novel transcripts, and applies filter strategies via add_filter'
  repo: https://github.com/MatthiasLienhard/isotools
- name: Python
  role: Runtime environment for the isotools module and analysis scripts
- name: pip
  role: Package manager used to install isotools (python3 -m pip install isotools)
- name: TALON
  role: Comparison tool run with default filtering on the same alignment to compute
    transcript overlap sets
- name: FLAIR
  role: Comparison tool run with default filtering on the same alignment to compute
    transcript overlap sets
- name: IsoQuant
  role: Comparison tool run with default filtering on the same alignment to compute
    transcript overlap sets
- name: Bambu
  role: Comparison tool run with default filtering on the same alignment to compute
    transcript overlap sets
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad364
    title: 'IsoTools: a flexible workflow for long-read transcriptome sequencing analysis'
  source_content_hash: sha256:6200277d5211d89425efff10cb27e005c2dec338b83e5cd110ef140f642fff85
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
  - pip
  edam_operation: http://edamontology.org/operation_3258
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# long-read-transcriptome-reconstruction

## Summary

Reconstruct transcriptomes from long-read transcriptome sequencing (LRTS) data using the IsoTools Python framework: import aligned full-length PacBio BAM files and a GENCODE gff3/gtf reference, compute QC, classify novel transcripts, and apply gene/transcript filter strategies (permissive, balanced, strict).

## When to use

Use when you have BAM files of aligned full-length LRTS transcripts (e.g. PacBio SequelII hepatocyte CTL/VPA data) and a gff3/gtf reference annotation, and need gene/transcript reconstruction with novel-transcript classification and tunable filtering.

## When NOT to use

- Input is short-read RNA-seq data rather than BAM files of aligned full-length LRTS transcripts
- The goal is only differential alternative splicing between prebuilt transcript models, not de novo transcriptome reconstruction from alignments
- No reference annotation in gff3/gtf format is available for the organism

## Inputs

- BAM files of aligned full-length long-read transcripts (e.g. PacBio SequelII hepatocyte CTL and VPA samples)
- Reference transcript annotation in gff3/gtf format (GENCODE)
- Genome FASTA file for quality control metrics

## Outputs

- Filtered Transcriptome object (saved as .pkl) with gene and transcript sets per filter strategy
- Per-strategy gene/transcript counts (e.g. 157,134 / 57,698 / 21,582 transcripts in 21,447 / 15,315 / 12,267 genes)
- Fraction of GENCODE-novel transcripts per strategy (32% under strict filtering)
- Multi-way tool transcript overlap table (e.g. tool_transcript_overlap_table.tsv) and filter_strategy_summary.csv

## How to apply

Install the isotools Python package with pip and build a Transcriptome object from the GENCODE gff3/gtf reference using Transcriptome.from_reference. Import each deposited LRTS BAM with add_sample_from_bam, assigning samples to groups (e.g. CTL vs valproic-acid-treated VPA hepatocytes), then run add_qc_metrics against the genome FASTA and make_index. Apply add_filter with the DEFAULT_GENE_FILTER, DEFAULT_TRANSCRIPT_FILTER, and DEFAULT_REF_TRANSCRIPT_FILTER constants once per strategy; the strict strategy keeps transcripts contributing at least 5% of gene expression with at least 7 reads, while the balanced filter excludes FRAGMENTS-class reads that do not use a reference TSS or PAS. Because filtering strongly affects results, run permissive, balanced, and strict strategies and record gene/transcript totals per strategy; compare overlap with other LRTS tools (TALON, FLAIR, IsoQuant, Bambu) via multi-way transcript overlap sets. Compute the fraction of GENCODE-novel transcripts from the biologically motivated transcript classification tags.

## Related tools

- **IsoTools** (Primary Python framework: imports LRTS BAMs and gff3/gtf reference, computes QC, classifies novel transcripts, and applies filter strategies via add_filter) — https://github.com/MatthiasLienhard/isotools
- **Python** (Runtime environment for the isotools module and analysis scripts)
- **pip** (Package manager used to install isotools (python3 -m pip install isotools))
- **TALON** (Comparison tool run with default filtering on the same alignment to compute transcript overlap sets)
- **FLAIR** (Comparison tool run with default filtering on the same alignment to compute transcript overlap sets)
- **IsoQuant** (Comparison tool run with default filtering on the same alignment to compute transcript overlap sets)
- **Bambu** (Comparison tool run with default filtering on the same alignment to compute transcript overlap sets)

## Examples

```
from isotools import Transcriptome; isoseq = Transcriptome.from_reference('example.gff.gz'); isoseq.add_sample_from_bam('example_1_CTL.bam', sample_name='CTL', group='CTL', platform='SequelII'); isoseq.add_qc_metrics('example.fa'); isoseq.add_filter(); isoseq.save('example_isotools.pkl')
```

## Evaluation signals

- Strict filtering of combined CTL and VPA hepatocyte long reads recovers 12,267 genes and 21,582 transcripts
- Under strict filtering, 32% of recovered transcripts are not found in GENCODE
- Permissive filtering yields 72,733 IsoTools-unique transcripts; balanced filtering drops this to 5,564 (less than 10%)
- Five-tool comparison table shows 20,642 transcripts shared by all five tools
- Imported transcriptome saves and reloads as a .pkl file after add_qc_metrics runs against the genome FASTA

## Limitations

- The number of identified (and uniquely recovered) transcripts is strongly dependent on the chosen filtering criteria, so single-strategy counts are not comparable across tools without matched filters
- Balanced filtering excludes FRAGMENTS-class reads that do not use a reference TSS or PAS, removing a class of transcripts from the reconstruction

## Evidence

- [intro] IsoTools is a python module for Long Read Transcriptome Sequencing (LRTS) analysis.: "IsoTools is a python module for Long Read Transcriptome Sequencing (LRTS) analysis."
- [intro] Annotation and classification of novel transcripts with biologically motivated classification scheme: "Annotation and classification of novel transcripts with biologically motivated classification scheme."
- [methods] pip installation: "python3 -m pip install isotools"
- [methods] add_filter usage: "isoseq.add_filter()
    isoseq.save('example_isotools.pkl')"
- [discussion] Default FRAGMENTS filter: "Default filter of FRAGMENTS now only tags reads that do not use a reference TSS or PAS"
- [readme] README usage example: [evidence span withheld — anchor status: external_doc]
- [readme] README key features: "Import of LRTS bam files (aligned full length transcripts)."
