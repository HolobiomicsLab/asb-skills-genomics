---
name: cell-type-isoform-usage-analysis
description: Use when single-cell PacBio Iso-Seq data must be compared across cell
  types to identify which isoforms (e.g., ENST transcripts) dominate per cell type,
  or to test whether a gene undergoes major isoform switching.
when_to_use_negative:
- Data are short-read RNA-seq or bulk long-read data without per-read cell-type labels
  — isoSeQL comparison requires cell-type-labeled reads from multiple Iso-Seq runs.
- Isoforms have not been annotated and filtered by SQANTI3 (sqanti3_qc.py plus RulesFilter)
  — isoSeQL was created to work with SQANTI3 output and cannot match raw PB.X.Y IDs
  across samples reliably.
- Only a single sample/run is available and no cross-sample or cross-cell-type isoform
  comparison is needed.
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: isoSeQL
  role: Compiles filtered SQANTI3 classification files from multiple samples with
    per-read cell-type labels into a database and runs built-in queries to extract
    per-cell-type isoform usage and structural category proportions
  repo: https://github.com/christine-liu/isoSeQL
- name: ccs
  role: Generates HiFi consensus reads (sample_isoSeqccs.bam) from sample_subreads.bam
    with a ZMW pass/fail report
- name: lima
  role: Removes Iso-Seq primers and orients reads 5'→3', selecting the expected asymmetric
    primer combination with --peek-guess
- name: isoseq
  role: Refines full-length reads (--require-polya), clusters isoforms (cluster2,
    ≥2 FLNC reads), and collapses redundant transcripts into GFF plus flnc_count
  repo: https://isoseq.how
- name: pbmm2
  role: Aligns clusters to the reference genome with --preset ISOSEQ -G 2500000 --sort
  repo: https://github.com/PacificBiosciences/pbmm2
- name: SQANTI3
  role: Annotates collapsed isoforms (sqanti3_qc.py with --genename, --report skip,
    flnc_count input) and applies the RulesFilter to produce high-confidence classification
    files
  repo: https://github.com/ConesaLab/SQANTI3
- name: minimap2
  role: Alignment engine underlying pbmm2 in the long-read pipeline
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btaf680
    title: 'isoSeQL: comparing long-read isoforms across multiple datasets'
  source_content_hash: sha256:f2ded3c943ea2788f7f344133a35332a58b1c96ba6d658052a8d0eddfe259e9c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/christine-liu/isoSeQL
  tools:
  - Python
  - R
  - SQANTI3
  - ccs
  - isoSeQL
  - isoseq
  - lima
  - minimap2
  - pbmm2
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# cell-type-isoform-usage-analysis

## Summary

Quantify long-read isoform usage per cell type by combining SQANTI3-filtered Iso-Seq classification outputs and per-read cell-type labels in an isoSeQL database, enabling detection of major isoform switching across cell types (e.g., IKZF1 in PBMCs).

## When to use

Use when single-cell PacBio Iso-Seq data must be compared across cell types to identify which isoforms (e.g., ENST transcripts) dominate per cell type, or to test whether a gene undergoes major isoform switching.

## When NOT to use

- Data are short-read RNA-seq or bulk long-read data without per-read cell-type labels — isoSeQL comparison requires cell-type-labeled reads from multiple Iso-Seq runs.
- Isoforms have not been annotated and filtered by SQANTI3 (sqanti3_qc.py plus RulesFilter) — isoSeQL was created to work with SQANTI3 output and cannot match raw PB.X.Y IDs across samples reliably.
- Only a single sample/run is available and no cross-sample or cross-cell-type isoform comparison is needed.

## Inputs

- sample_subreads.bam (PacBio HiFi subreads per PBMC sample)
- Iso-Seq 5'/3' primer sequences for lima demultiplexing
- Reference genome (FASTA) for pbmm2 alignment
- GTF annotation for SQANTI3 QC
- flnc_count file from isoseq collapse
- Filtered SQANTI3 classification files per sample
- Seurat v4.3 count matrices with per-read cell-type labels

## Outputs

- isoSeQL database of compiled long-read isoforms across samples
- Structural category proportions of reads and isoforms per cell type
- Per-cell-type isoform usage table for target genes (e.g., pbmc_isoseql_results.csv with IKZF1 ENST00000646110.1 vs ENST00000698576.1 usage)

## How to apply

Process each PBMC sample through the Iso-Seq chain: generate HiFi reads with ccs (with --report-file), demultiplex and orient with lima using --isoseq and --peek-guess, refine with isoseq refine --require-polya, cluster with isoseq cluster2 (isoforms supported by at least 2 FLNC reads), align with pbmm2 align --preset ISOSEQ -G 2500000 --sort, and collapse with isoseq collapse to obtain a GFF and flnc_count file. Annotate with SQANTI3 sqanti3_qc.py (--genename, --report skip, supplying the flnc_count file), then apply sqanti3_RulesFilter.py to retain high-confidence isoforms. Load each sample's filtered classification file into an isoSeQL database together with per-read cell-type labels derived from the Seurat v4.3 count matrices, so isoform IDs (PB.X.Y) that differ between samples can be matched across samples. Run isoSeQL's built-in queries to tabulate structural category proportions of reads and isoforms per cell type and to extract gene-level isoform usage (e.g., IKZF1 ENST00000646110.1 vs ENST00000698576.1) across CD14+ monocytes, FCGR3A+ monocytes, platelets, CD4+ T cells, NK cells, and B cells.

## Related tools

- **isoSeQL** (Compiles filtered SQANTI3 classification files from multiple samples with per-read cell-type labels into a database and runs built-in queries to extract per-cell-type isoform usage and structural category proportions) — https://github.com/christine-liu/isoSeQL
- **ccs** (Generates HiFi consensus reads (sample_isoSeqccs.bam) from sample_subreads.bam with a ZMW pass/fail report)
- **lima** (Removes Iso-Seq primers and orients reads 5'→3', selecting the expected asymmetric primer combination with --peek-guess)
- **isoseq** (Refines full-length reads (--require-polya), clusters isoforms (cluster2, ≥2 FLNC reads), and collapses redundant transcripts into GFF plus flnc_count) — https://isoseq.how
- **pbmm2** (Aligns clusters to the reference genome with --preset ISOSEQ -G 2500000 --sort) — https://github.com/PacificBiosciences/pbmm2
- **SQANTI3** (Annotates collapsed isoforms (sqanti3_qc.py with --genename, --report skip, flnc_count input) and applies the RulesFilter to produce high-confidence classification files) — https://github.com/ConesaLab/SQANTI3
- **minimap2** (Alignment engine underlying pbmm2 in the long-read pipeline)

## Examples

```
ccs sample_subreads.bam --report-file /outdir/sample_isoSeqccs_report.txt /outdir/sample_isoSeqccs.bam
```

## Evaluation signals

- Structural category proportions of both reads and isoforms remain constant across cell types after loading into isoSeQL.
- IKZF1 isoform usage reproduces the reported switch: ENST00000646110.1 dominant in CD14+ monocytes, FCGR3A+ monocytes, and platelets; ENST00000698576.1 dominant in CD4+ T cells, NK cells, and B cells.
- Every isoform in the isoSeQL database carries a sample/cell-type label, resolving the cross-sample PB.X.Y ID mismatch that motivated the tool.
- Clustered isoforms are supported by at least 2 FLNC reads and only RulesFilter-passing isoforms are loaded into the database.
- The final isoform usage table is written to pbmc_isoseql_results.csv covering all six cell types (CD14+ monocytes, FCGR3A+ monocytes, platelets, CD4+ T cells, NK cells, B cells).

## Limitations

- isoSeQL was created to work with SQANTI2 (now SQANTI3) output; raw or non-SQANTI3-annotated isoform lists cannot be matched across samples.
- The legacy cross-sample workaround — concatenating labeled HiFi reads into a single 'mega sample' and running isoseq > minimap2 > cDNA_cupcake > SQANTI2 — does not scale well with increased sample numbers or SMRTcell throughput.
- isoSeQL depends on per-read cell-type labels supplied externally (here from Seurat v4.3 count matrices); without those labels per-cell-type usage cannot be computed.

## Evidence

- [abstract] isoSeQL compiles long-read transcriptomic data and identifies common and unique isoforms across multiple samples: "We developed isoSeQL, a tool for compiling long-read transcriptomic data, identifying common and unique isoforms across multiple samples"
- [intro] isoform ID mismatch across samples motivated isoSeQL: "I was working with multiple samples that I wanted to compare to each other, but quickly realized that the isoform ID #s (PB.X.Y) in each sample couldn't be matched."
- [methods] cluster2 minimum FLNC support: "isoseq cluster2 generates clusters of isoforms with the support of at least 2 FLNC reads"
- [methods] RulesFilter invocation on classification file: "python /path/to/SQANTI3/sqanti3_RulesFilter.py /outdir/sample_isoSeqccs_demux_flnc_cluster_mapped_collapsed_classification.txt"
- [abstract] isoSeQL works with SQANTI3 output and examines isoform variation across cell types: "isoSeQL will augment approaches that utilize long-read sequencing to discover novel isoforms and to examine how isoforms vary across different experimental and biological conditions and cell types."
- [intro] mega-sample concatenation approach does not scale: "This method 100% works, but does not scale well with increased sample numbers or especially as the throughput of SMRTcells has increased over the years."
