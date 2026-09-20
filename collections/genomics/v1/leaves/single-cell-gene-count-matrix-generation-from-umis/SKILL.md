---
name: single-cell-gene-count-matrix-generation-from-umis
description: Use when uniquely mapped FLNC reads with corrected cellBC/UMI assignments
  and a filtered tmap already exist, and a per-cell x per-gene count matrix is the
  required final deliverable.
when_to_use_negative:
- The goal is isoform-level rather than gene-level quantification — use scIsoform_matrix
  instead
- The tmap still contains multi-mapping FLNC transcripts (filter.tmap.pl has not been
  run), since the matrix must be built exclusively from uniquely mapped reads
- Cell barcodes and UMIs have not yet been corrected (no cellBC_UMI_correction.xls
  output from cellBC_UMI_corrector)
edam_operation: http://edamontology.org/operation_3221
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0080
tools:
- name: scISA-Tools
  role: Pipeline containing the scGene_matrix utility that computes single-cell gene
    expression quantity per sample
  repo: https://github.com/shizhuoxing/scISA-Tools
- name: gffcompare
  role: Assigns FLNC transcripts to annotated genes via tmap match codes, providing
    the gene identity column aggregated by scGene_matrix
  repo: https://ccb.jhu.edu/software/stringtie/gffcompare.shtml
- name: filter.tmap.pl
  role: Retains unique mapping results so the count matrix is built exclusively from
    uniquely mapped FLNC reads
- name: cellBC_UMI_corrector
  role: Wraps CellRanger barcode/UMI correction to supply the corrected cellBC/UMI
    assignments consumed by scGene_matrix
  repo: https://github.com/shizhuoxing/scISA-Tools
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-38324-9
    title: High-throughput and high-accuracy single-cell RNA isoform analysis using
      PacBio circular consensus sequencing
  source_content_hash: sha256:0bc6f16c6a7309d42e55db23f75586cc5a5be2e7b3fd3cbe98c83ab3efe0107c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shizhuoxing/scISA-Tools
  tools:
  - cellBC_UMI_corrector
  - filter.tmap.pl
  - gffcompare
  - minimap2
  - sam2gff.pl
  - scGene_matrix
  - scISA-Tools
  edam_operation: http://edamontology.org/operation_3221
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# single-cell gene count matrix generation from UMIs

## Summary

Generates the per-sample single-cell gene expression quantity table from uniquely mapped FLNC reads in the scISA-Tools/HIT-scISOseq pipeline, using the scGene_matrix utility downstream of gffcompare-based gene assignment and cellBC/UMI correction.

## When to use

Use when uniquely mapped FLNC reads with corrected cellBC/UMI assignments and a filtered tmap already exist, and a per-cell x per-gene count matrix is the required final deliverable.

## When NOT to use

- The goal is isoform-level rather than gene-level quantification — use scIsoform_matrix instead
- The tmap still contains multi-mapping FLNC transcripts (filter.tmap.pl has not been run), since the matrix must be built exclusively from uniquely mapped reads
- Cell barcodes and UMIs have not yet been corrected (no cellBC_UMI_correction.xls output from cellBC_UMI_corrector)

## Inputs

- flnc.filtered.tmap (unique-mapping gffcompare tmap from filter.tmap.pl)
- flnc.cellBC_UMI_correction.xls (corrected cell barcode/UMI assignments from cellBC_UMI_corrector)
- NGS top-rank cell barcode whitelist list (e.g. NGScellBC.list derived from 3M-february-2018.txt)
- reference annotation GTF (ref.genes.gtf) used for gene assignment via gffcompare

## Outputs

- per-cell x per-gene count matrix for the labeled sample (single-cell gene expression quantity table)

## How to apply

After gffcompare assigns each FLNC transcript to annotated genes and filter.tmap.pl retains unique tmap matches only, run scGene_matrix to aggregate corrected UMIs into per-cell, per-gene counts. Provide the NGS-derived top-rank cell barcode whitelist via -ngsbc (e.g. NGScellBC.list from the 3M-february-2018 whitelist) together with the cellBC_UMI_correction.xls file and the filtered tmap, or alternatively restrict to the TGS top-rank barcodes with -topbc (e.g. 2500). Set -sample to a label (e.g. TEST) and -outdir for outputs. The resulting matrix is built exclusively from uniquely mapped FLNC reads with corrected cellBC/UMI assignments, so prior correction quality (posterior probability >0.975 for cellBC replacement) directly determines matrix accuracy. For isoform-level quantities rather than gene-level, the companion scIsoform_matrix utility serves the same role.

## Related tools

- **scISA-Tools** (Pipeline containing the scGene_matrix utility that computes single-cell gene expression quantity per sample) — https://github.com/shizhuoxing/scISA-Tools
- **gffcompare** (Assigns FLNC transcripts to annotated genes via tmap match codes, providing the gene identity column aggregated by scGene_matrix) — https://ccb.jhu.edu/software/stringtie/gffcompare.shtml
- **filter.tmap.pl** (Retains unique mapping results so the count matrix is built exclusively from uniquely mapped FLNC reads)
- **cellBC_UMI_corrector** (Wraps CellRanger barcode/UMI correction to supply the corrected cellBC/UMI assignments consumed by scGene_matrix) — https://github.com/shizhuoxing/scISA-Tools

## Examples

```
perl scGene_matrix.pl -ngsbc NGScellBC.list -tgsbcumi flnc.cellBC_UMI_correction.xls -tmap flnc.filtered.tmap -outdir ./ -sample TEST
```

## Evaluation signals

- Output matrix for the labeled sample exists as a per-cell x per-gene count table in the specified -outdir
- Counts derive exclusively from uniquely mapped FLNC reads (only flnc.filtered.tmap entries were used)
- Cell barcode universe in the matrix matches the supplied whitelist-derived barcode list (NGScellBC.list or -topbc selection)
- All counted UMIs carry corrected cellBC/UMI assignments traceable to flnc.cellBC_UMI_correction.xls
- Gene identifiers correspond to the reference annotation GTF used in the gffcompare -r step

## Limitations

- Using -ngsbc requires an external NGS-derived cell barcode whitelist; using -topbc alone restricts the matrix to TGS top-rank barcodes only, which changes the cell universe
- Matrix quality depends on upstream cellBC correction posterior probability threshold (>0.975) and UMI quality filters (no homopolymers, no N, no base quality <10)

## Evidence

- [methods] scGene_matrix utility usage: "we are using `scGene_matrix` utility to get the single cell gene expression quantity in each sample"
- [methods] filter.tmap.pl usage: "perl filter.tmap.pl flnc.isoseq_flnc.Transcript.gff.tmap > flnc.filtered.tmap"
- [discussion] cellBC correction threshold: [evidence span withheld — anchor status: not_found]
- [readme] pipeline purpose from README: "This is a comprehensive tool including basic data processing, cellBC extraction, expression matrix generation, and so on."
