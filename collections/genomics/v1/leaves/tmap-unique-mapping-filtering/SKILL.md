---
name: tmap-unique-mapping-filtering
description: Use when a mapped FLNC SAM/GFF has been compared to a reference GTF with
  gffcompare and multi-mapping reads must be reduced to one unique gene assignment
  per read before quantification.
when_to_use_negative:
- Input is raw SAM/BAM alignments that have not yet been converted to GFF and compared
  with gffcompare against the reference annotation
- The goal is isoform-level quantification with scIsoform_matrix only and multi-mapped
  transcripts should be retained for further SQANTI3-style characterization
- A downstream analysis intentionally examines ambiguous multi-gene overlap patterns
  rather than building a gene count matrix
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0102
tools:
- name: gffcompare
  role: Compares FLNC GFF against the reference GTF to assign transcripts to annotated
    genes via tmap match codes
  repo: https://ccb.jhu.edu/software/stringtie/gffcompare.shtml
- name: filter.tmap.pl
  role: Filters the gffcompare tmap to keep unique mapping results only
  repo: https://github.com/shizhuoxing/scISA-Tools
- name: sam2gff.pl
  role: Converts the minimap2 FLNC SAM alignments to GFF format prior to gffcompare
  repo: https://github.com/gpertea/gscripts/blob/master/sam2gff.pl
- name: scGene_matrix
  role: Downstream consumer of the filtered tmap; computes the single-cell gene expression
    matrix
  repo: https://github.com/shizhuoxing/scISA-Tools
- name: minimap2
  role: Aligns FLNC transcripts to the reference genome in splice mode upstream of
    the tmap filtering
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-38324-9
    title: High-throughput and high-accuracy single-cell RNA isoform analysis using
      PacBio circular consensus sequencing
  source_content_hash: sha256:0c305da68affa9106db16eb4a6263ec22ec45c13f77fe8ef35b98e03c7f376f6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - filter.tmap.pl
  - gffcompare
  - minimap2
  - sam2gff.pl
  - scGene_matrix
  - scISA-Tools
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# tmap-unique-mapping-filtering

## Summary

Filters the gffcompare tmap file so that only FLNC reads with a single, unique gene assignment (via tmap match codes) are retained, providing clean read-to-gene mappings before single-cell gene count matrix generation with scGene_matrix.

## When to use

Use when a mapped FLNC SAM/GFF has been compared to a reference GTF with gffcompare and multi-mapping reads must be reduced to one unique gene assignment per read before quantification.

## When NOT to use

- Input is raw SAM/BAM alignments that have not yet been converted to GFF and compared with gffcompare against the reference annotation
- The goal is isoform-level quantification with scIsoform_matrix only and multi-mapped transcripts should be retained for further SQANTI3-style characterization
- A downstream analysis intentionally examines ambiguous multi-gene overlap patterns rather than building a gene count matrix

## Inputs

- gffcompare tmap file (e.g. flnc.isoseq_flnc.Transcript.gff.tmap) produced from minimap2-aligned FLNC reads
- Reference genome annotation GTF (ref.genes.gtf) used for the gffcompare -r run

## Outputs

- Filtered tmap file with unique mapping results only (flnc.filtered.tmap)

## How to apply

After aligning FLNC reads to the reference genome with minimap2 in splice mode ('-ax splice -uf --secondary=no -C5'), convert the SAM to GFF with sam2gff.pl and run 'gffcompare -r ref.genes.gtf' to assign each FLNC transcript to annotated genes via tmap match codes. Because gffcompare can emit multiple tmap rows per query transcript (one per overlapping reference gene), reduce the tmap to unique mapping results only with 'perl filter.tmap.pl flnc.isoseq_flnc.Transcript.gff.tmap > flnc.filtered.tmap'. The rationale is that the downstream scGene_matrix step must build the per-cell x per-gene count table exclusively from uniquely mapped FLNC reads with corrected cellBC/UMI assignments, so ambiguous gene assignments would double-count or misassign UMIs. The filtered tmap then feeds scGene_matrix together with the cellBC_UMI-corrected file and the barcode whitelist.

## Related tools

- **gffcompare** (Compares FLNC GFF against the reference GTF to assign transcripts to annotated genes via tmap match codes) — https://ccb.jhu.edu/software/stringtie/gffcompare.shtml
- **filter.tmap.pl** (Filters the gffcompare tmap to keep unique mapping results only) — https://github.com/shizhuoxing/scISA-Tools
- **sam2gff.pl** (Converts the minimap2 FLNC SAM alignments to GFF format prior to gffcompare) — https://github.com/gpertea/gscripts/blob/master/sam2gff.pl
- **scGene_matrix** (Downstream consumer of the filtered tmap; computes the single-cell gene expression matrix) — https://github.com/shizhuoxing/scISA-Tools
- **minimap2** (Aligns FLNC transcripts to the reference genome in splice mode upstream of the tmap filtering)

## Examples

```
perl filter.tmap.pl flnc.isoseq_flnc.Transcript.gff.tmap > flnc.filtered.tmap
```

## Evaluation signals

- flnc.filtered.tmap exists and contains at most one tmap row per query transcript (unique mapping results only)
- Every filtered tmap entry carries a defined gffcompare match code tying the FLNC transcript to exactly one annotated gene
- The subsequent scGene_matrix output for the sample is a per-cell x per-gene count table built exclusively from uniquely mapped FLNC reads with corrected cellBC/UMI assignments
- The cell barcode universe of the count matrix matches the supplied whitelist-derived barcode list (e.g. the NGS top-rank cellBC list or -topbc setting)

## Limitations

- Filtering to unique mappings discards FLNC reads that overlap multiple annotated genes, which reduces sensitivity for ambiguous loci
- Gene assignment depends entirely on the completeness of the reference annotation GTF (e.g. Ensembl Macaca_fascicularis_5.0.99); genes absent from the reference cannot be assigned
- Tmap match codes conflate match strictness levels, so 'unique' here means one retained row per query, not necessarily a full-length exact match

## Evidence

- [methods] filter.tmap.pl keeps unique tmap rows: "perl filter.tmap.pl flnc.isoseq_flnc.Transcript.gff.tmap > flnc.filtered.tmap"
- [readme] gffcompare maps FLNC to reference annotation: "gffcompare -r ref.genes.gtf isoseq_flnc.Transcript.gff -o flnc"
- [readme] sam2gff.pl source conversion step: "perl sam2gff.pl isoseq_flnc.Transcript.sam > isoseq_flnc.Transcript.gff"
- [methods] scGene_matrix generates single-cell gene expression quantity: "we are using `scGene_matrix` utility to get the single cell gene expression quantity in each sample"
