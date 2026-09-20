---
name: read-length-distribution-analysis
description: 'Reference for building QC panels that assess Oxford Nanopore (or PacBio)
  full-length single-cell cDNA read quality: you have a minimap2_splice BAM and per-cell
  counts from a scywalker run and need read-length histograms plus alignment metrics
  in a report.'
when_to_use_negative:
- Input is short-read scRNA-seq data — the read-length vs exon-count relationship
  inspected here is specific to full-length long-read cDNA (Oxford Nanopore or PacBio)
- You only need cell calling/QC (knee plot, emptydrops filtering) and not alignment/read-quality
  metrics — no BAM or cramino metrics required
edam_operation: http://edamontology.org/operation_0531
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: scywalker
  role: End-to-end pipeline whose reporting module assembles the read-length histograms
    and cramino metrics into the interactive HTML QC report
  repo: https://github.com/derijkp/scywalker
- name: cramino
  role: Computes alignment metrics from the minimap2 BAM using its spliced mode for
    the report
- name: minimap2
  role: Produces the minimap2_splice BAM of aligned reads whose length distributions
    are summarized
- name: genomecomb
  role: Framework in which scywalker (and its reporting) is largely implemented; provides
    supporting tooling
  repo: https://github.com/derijkp/genomecomb
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae549
    title: 'Scywalker: scalable end-to-end data analysis workflow for long-read single-cell
      transcriptome sequencing'
  source_content_hash: sha256:ca3f110dfe0d8697e3868ebcfc7cd6b50534719d8bfecdfce6f2012ac8c8e3b0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/derijkp/scywalker
  tools:
  - cramino
  - genomecomb
  - minimap2
  - scywalker
  edam_operation: http://edamontology.org/operation_0531
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# read-length-distribution-analysis

## Summary

Generates read-length distribution QC plots (overall read-length histogram and read length versus number of exons per read) with cramino-derived alignment metrics, as part of scywalker's interactive HTML QC report for long-read single-cell/nuclei data.

## When to use

Reference for building QC panels that assess Oxford Nanopore (or PacBio) full-length single-cell cDNA read quality: you have a minimap2_splice BAM and per-cell counts from a scywalker run and need read-length histograms plus alignment metrics in a report.

## When NOT to use

- Input is short-read scRNA-seq data — the read-length vs exon-count relationship inspected here is specific to full-length long-read cDNA (Oxford Nanopore or PacBio)
- You only need cell calling/QC (knee plot, emptydrops filtering) and not alignment/read-quality metrics — no BAM or cramino metrics required

## Inputs

- minimap2_splice BAM file of aligned long reads produced by the scywalker run
- per-cell gene counts (emptydrops-approved filtered set) from the calling module
- per-cell cell-type group assignments from the typing module

## Outputs

- Overall read-length histogram QC panel
- Read-length versus number-of-exons-per-read histogram QC panel
- Cramino-derived alignment metrics
- Single interactive HTML report (report.html) containing all QC panels

## How to apply

Load the minimap2_splice BAM produced by the scywalker run and compute alignment metrics with cramino in its spliced mode, since the reads are genomic alignments of spliced full-length cDNA. From the same alignments build two distribution panels: an overall read-length histogram, and a read-length histogram stratified by (versus) number of exons per read, which checks that longer multi-exon reads are recovered as expected for full-length long reads. Combine these panels with the knee-plot (emptydrops-approved cells marked), genes-per-cell histogram, genomic interval overlap summary, and the UMAP of assigned cell types. Assemble all metrics and plots into a single interactive HTML document (report.html). Correctness is judged by verifying report.html exists, parses as HTML, and contains both the cramino-derived metrics and all named QC panels including the read-length plots.

## Related tools

- **scywalker** (End-to-end pipeline whose reporting module assembles the read-length histograms and cramino metrics into the interactive HTML QC report) — https://github.com/derijkp/scywalker
- **cramino** (Computes alignment metrics from the minimap2 BAM using its spliced mode for the report)
- **minimap2** (Produces the minimap2_splice BAM of aligned reads whose length distributions are summarized)
- **genomecomb** (Framework in which scywalker (and its reporting) is largely implemented; provides supporting tooling) — https://github.com/derijkp/genomecomb

## Examples

```
scywalker -v 1 -d 8 -dmaxmem 64G -refdir g17 -sc_expectedcells 183 -cellmarkerfile markers_chr17.tsv -threads 6 test10x
```

## Evaluation signals

- report.html exists in the sample/project directory and is parseable HTML
- report.html contains the cramino-derived alignment metrics computed in spliced mode
- The overall read-length histogram panel is present and populated from the minimap2_splice BAM
- The read-length versus number-of-exons-per-read panel is present in the report
- The report also contains the knee plot, genes-per-cell histogram, interval-overlap summary, and UMAP panels alongside the read-length plots

## Limitations

- The reporting depends on the upstream minimap2_splice alignment; organelle-containing references can make the isoquant-based code hang or crash, so organelles must be specified at scywalker_makerefdir time to get a complete run to report on
- Read-length panels alone do not confirm correct cell assignment; they must be interpreted together with emptydrops knee-plot filtering and cell-type panels

## Evidence

- [results] cramino spliced-mode metrics in report: [evidence span withheld — anchor status: not_found]
- [results] QC panels including read-length histograms: [evidence span withheld — anchor status: not_found]
- [abstract] scywalker includes interactive report: "Scywalker includes data quality control, cell type identification, and an interactive report."
- [results] validation of report.html contents: [evidence span withheld — anchor status: not_found]
- [methods] default aligner minimap2_splice: "The default aligner is minimap2_splice (minimap2 with the splice preset)"
