---
name: exon-structure-clustering-by-umi-support
description: Use when full-length Nanopore scRNA-seq reads (10x-barcoded, UMI-annotated)
  must be collapsed into discrete isoform models and candidate novel isoforms must
  be supported by a minimum number of UMIs.
when_to_use_negative:
- Input reads are not UMI/cell-barcode-annotated long reads (e.g. raw fastq or short-read-only
  data) — CollapseModel relies on per-UMI grouping of Nanopore SAM records
- The goal is gene-level or isoform-level count matrices from already-validated transcriptomes
  (use IsoformMatrix, Steps 4a/4b, instead of novel isoform discovery)
- No external short-read data or CAGE/polyA reference resources are available, so
  novel isoforms cannot satisfy the juncCo/cageCo/polyaCo validation step
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Sicelore-2.1.jar (CollapseModel)
  role: Runs the UMI-based isoform clustering and MINEVIDENCE/RNMIN/DELTA filtering
    (Step 7, novel isoform discovery)
  repo: https://github.com/ucagenomix/sicelore-2.1
- name: Java
  role: Runtime required to execute Sicelore-2.1.jar (Java >= 13)
- name: samtools
  role: SAM/BAM manipulation of the isobam.bam and Illumina bam inputs
  repo: https://github.com/samtools/samtools
- name: Illumina
  role: Short-read sequencing used to externally validate exon-exon junctions of candidate
    novel isoforms (juncCo cutoff)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-17800-6
    title: High throughput error corrected Nanopore single cell transcriptome sequencing
  source_content_hash: sha256:ac45b84ecb3e45d2d34f09abd5d98c2b42248d59821dc8ac1d03df2aa4c13820
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ucagenomix/sicelore-2.1
  tools:
  - Illumina
  - Java
  - Sicelore-2.1.jar (CollapseModel)
  - samtools
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# exon-structure-clustering-by-umi-support

## Summary

Clusters Nanopore long reads into transcript isoform models by grouping SAM records per cell barcode and UMI according to exon makeup, then filters candidate novel isoforms by UMI-based evidence thresholds in the Sicelore CollapseModel step.

## When to use

Use when full-length Nanopore scRNA-seq reads (10x-barcoded, UMI-annotated) must be collapsed into discrete isoform models and candidate novel isoforms must be supported by a minimum number of UMIs.

## When NOT to use

- Input reads are not UMI/cell-barcode-annotated long reads (e.g. raw fastq or short-read-only data) — CollapseModel relies on per-UMI grouping of Nanopore SAM records
- The goal is gene-level or isoform-level count matrices from already-validated transcriptomes (use IsoformMatrix, Steps 4a/4b, instead of novel isoform discovery)
- No external short-read data or CAGE/polyA reference resources are available, so novel isoforms cannot satisfy the juncCo/cageCo/polyaCo validation step

## Inputs

- isobam.bam (cellBC/UMI-annotated Nanopore SAM/BAM from Sicelore IsoformMatrix, ISOBAM=true)
- ValidBarcodes.csv (list of valid cell barcodes)
- mouse refFlat annotation (GENCODE vM24)
- Illumina STAR-aligned short-read bam (e.g. GSE69711 E18) for junction validation
- Fantom5.cage_peaks.bed (CAGE TSS peaks)
- gencode.v38.polyAs.bed (polyadenylation sites)

## Outputs

- CollapseModel.d2.rn1.e5.txt (classified isoform models)
- CollapseModel.d2.rn1.e5.gff
- CollapseModel.d2.rn1.e5.final.gff (validated novel isoform models)
- CollapseModel.d2.rn1.e5.final.refflat.txt

## How to apply

Take the isobam.bam produced by the Sicelore IsoformMatrix pipeline (ISOBAM=true) together with ValidBarcodes.csv and the mouse refFlat annotation as inputs to Sicelore-2.1.jar CollapseModel. Group SAM records per cell barcode and UMI by transcript isoform according to exon makeup, keeping only UMIs supported by at least RNMIN=1 reads. Filter out isoforms whose exon structure is backed by fewer than MINEVIDENCE=5 UMIs, discarding apparent 3'/5' degradation products and longer potential novel isoforms, tolerating DELTA=2 bases of added or lacking sequences at exon-exon junction boundaries. Remaining novel isoforms are then validated externally by requiring all exon-exon junctions to be GENCODE-annotated or confirmed by at least juncCo=1 short-read junction, with 5' starts within cageCo=50 bases of a FANTOM5 CAGE peak and 3' ends within polyaCo=50 bases of a GENCODE polyA site. The rationale is that UMI-level support, not raw read count, distinguishes genuine isoforms from Nanopore sequencing-error artifacts.

## Related tools

- **Sicelore-2.1.jar (CollapseModel)** (Runs the UMI-based isoform clustering and MINEVIDENCE/RNMIN/DELTA filtering (Step 7, novel isoform discovery)) — https://github.com/ucagenomix/sicelore-2.1
- **Java** (Runtime required to execute Sicelore-2.1.jar (Java >= 13))
- **samtools** (SAM/BAM manipulation of the isobam.bam and Illumina bam inputs) — https://github.com/samtools/samtools
- **Illumina** (Short-read sequencing used to externally validate exon-exon junctions of candidate novel isoforms (juncCo cutoff))

## Examples

```
java -jar Sicelore-2.1.jar CollapseModel -i isobam.bam -o CollapseModel.d2.rn1.e5 -f mouse.refFlat --rnmin 1 --minevidence 5 --delta 2 -c ValidBarcodes.csv --juncCo 1 --cageCo 50 --polyaCo 50
```

## Evaluation signals

- The number of novel isoforms in CollapseModel.d2.rn1.e5.final.gff (or the classification .txt) equals the reported 4388 for the 1121-cell E18 mouse brain dataset
- Every listed model satisfies the junction/CAGE/polyA acceptance criteria: junctions in GENCODE or supported by >= juncCo=1 short reads, 5' ends within cageCo=50 bases of a CAGE peak, 3' ends within polyaCo=50 bases of a polyA site
- No retained candidate isoform has an exon structure supported by fewer than MINEVIDENCE=5 UMIs
- Apparent 3'/5' degradation products and longer potential novel isoforms of reference transcripts are absent from the final.gff
- All isoform assignments tolerate at most DELTA=2 bases of added or lacking sequence at exon-exon junction boundaries

## Limitations

- Nanopore sequencing errors can mis-assign UMIs and mimic splice junctions; DELTA=2 and UMI support thresholds mitigate but do not eliminate this, so external Illumina junction validation is required
- MAXCLIP (default > 150 nt) already discards SAM records with extensive hard/soft clipping, so heavily clipped chimeric reads never reach clustering
- Novel isoforms lacking short-read, CAGE, or polyA support are rejected, so truly novel endpoints or lineage-specific junctions absent from references may be missed

## Evidence

- [methods] MINEVIDENCE filter on UMI-supported exon structures: "Transcripts isoforms showing an exon structure supported by less than MINEVIDENCE (default=5) UMIs are filtered out"
- [methods] DELTA junction margin: "authorizing a DELTA (default 2) bases margin of added or lacking sequences at exon boundaries"
- [methods] RNMIN read-per-UMI threshold: "considering only UMI supported by RNMIN (default=1) reads"
- [readme] Step 7 in Sicelore-2.1 workflow: "Step 7 (optional) - [Novel isoform discovery](#new-model)"
- [abstract] ScNaUmi-seq UMI-guided error correction rationale: "UMI guided error correction allows to generate high accuracy full length sequence information"
