---
name: pacbio-ccs-read-processing
description: Use when input is PacBio Iso-seq CCS read FASTA from long-read transcriptome
  sequencing and it must be prepared/filtered before TAGET alignment, annotation,
  or fusion detection.
when_to_use_negative:
- Input is short-read RNA-seq (e.g., Illumina FASTQ) rather than PacBio Iso-seq CCS
  reads — the cluster filtering logic does not apply
- 'The analysis goal is to maximize gene detection at low coverage: filtering single-read
  CCS clusters discards ~80% of reads, many from true expressed isoforms, so alternative
  tools that use these reads may be preferable'
- The data are already full-length transcript FASTA post-filtering and only alignment/annotation
  (TransAnnot.py) is needed
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0080
tools:
- name: TAGET
  role: Toolkit that consumes the filtered CCS-read FASTA for transcript alignment,
    annotation, and gene fusion detection (TransAnnot.py, TAGET_fusion_2-3_ajust.r)
  repo: https://github.com/XiDsLab/TAGET
- name: PacBio Iso-seq (PacBio)
  role: Sequencing platform generating very long and accurate HiFi/CCS reads that
    are the input to this processing step
- name: samtools
  role: Environmental dependency of TAGET used in the alignment/processing pipeline
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-41649-0
    title: 'TAGET: a toolkit for analyzing full-length transcripts from long-read
      sequencing'
  source_content_hash: sha256:b01e5209bcc8ccde302d6a029a3db2357559d0afbd1d67440fd57d750214158e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiDsLab/TAGET
  tools:
  - GMAP
  - Minimap2
  - PacBio Iso-seq (PacBio)
  - TAGET
  - samtools
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# pacbio-ccs-read-processing

## Summary

Handles PacBio Iso-seq CCS reads as input to TAGET, including the quality-motivated filtering of circular-consensus-sequence read clusters that contain only a single read, which feeds TAGET's transcript alignment, annotation, and gene fusion detection.

## When to use

Use when input is PacBio Iso-seq CCS read FASTA from long-read transcriptome sequencing and it must be prepared/filtered before TAGET alignment, annotation, or fusion detection.

## When NOT to use

- Input is short-read RNA-seq (e.g., Illumina FASTQ) rather than PacBio Iso-seq CCS reads — the cluster filtering logic does not apply
- The analysis goal is to maximize gene detection at low coverage: filtering single-read CCS clusters discards ~80% of reads, many from true expressed isoforms, so alternative tools that use these reads may be preferable
- The data are already full-length transcript FASTA post-filtering and only alignment/annotation (TransAnnot.py) is needed

## Inputs

- PacBio Iso-seq CCS read FASTA (e.g., ${i}.fa from the PacBio platform)
- CCS read cluster information (to identify clusters containing only one read)
- hg38 reference genome FASTA (hg38.fa)
- Ensembl annotation GTF (hg38.ensembl.gtf) and its pickle database built with gtf_db_make.py

## Outputs

- Filtered full-length transcript FASTA of retained CCS reads (~20% of original reads)
- TAGET alignment/annotation outputs ({sample_id}.annot.stat, {sample_id}.annot.bed, {sample_id}.annot.junction, etc.)
- minimap2- and HISAT2-generated bed files used by the fusion detection step

## How to apply

Start from the CCS read FASTA produced by PacBio Iso-seq (e.g., the ${i}.fa file used in TAGET's fusion workflow). Apply the quality-based cluster filter: CCS-read clusters containing only one read are discarded because of their lower quality, and this step removes about 80% of the CCS reads. The retained full-length transcript FASTA is then passed to TAGET's TransAnnot.py (python TransAnnot.py -f [fasta] -g hg38.fa -o [outdir] -a hg38.ensembl.gtf -p [process]) for alignment against hg38 with minimap2/HISAT2/GMAP and downstream annotation. For gene fusion detection, the CCS read FASTA plus the minimap2- and HISAT2-generated bed files and annot.stat feed TAGET_fusion_2-3_ajust.r. Rationale: single-read CCS clusters are noisy, so filtering them improves junction prediction and isoform accuracy even though it reduces effective coverage and detected genes.

## Related tools

- **TAGET** (Toolkit that consumes the filtered CCS-read FASTA for transcript alignment, annotation, and gene fusion detection (TransAnnot.py, TAGET_fusion_2-3_ajust.r)) — https://github.com/XiDsLab/TAGET
- **PacBio Iso-seq (PacBio)** (Sequencing platform generating very long and accurate HiFi/CCS reads that are the input to this processing step)
- **samtools** (Environmental dependency of TAGET used in the alignment/processing pipeline)

## Examples

```
python TransAnnot.py -f ${i}.fa -g hg38.fa -o outdir -a hg38.ensembl.gtf -p 8
```

## Evaluation signals

- Approximately 80% of CCS reads should be removed by the single-read-cluster quality filter; a much lower or higher removal rate suggests the filter was misapplied
- Running TransAnnot.py should complete in less than 1 hour with 8 cores on a Linux server for the demo data (data.7z, demo.7z, script.7z)
- Expected output files must be present, including {sample_id}.annot.stat, {sample_id}.annot.bed, {sample_id}.annot.junction, and {sample_id}.annot.db.pickle
- On the COLO829 Iso-seq benchmark, junction predictions from the filtered-read pipeline should reach ~97.4% Ensembl match for TAGET (vs 95.2% minimap2, 94.3% GMAP)
- gtf_db_make.py must have produced hg38.ensembl.v20200306.1.pickle before annotation, otherwise annotation will fail or be empty

## Limitations

- Filtering single-read CCS clusters discards ~80% of CCS reads, many of which come from true expressed isoforms, limiting detected genes; more advanced tools that use these reads could increase detected genes
- Effective coverage of Iso-seq is substantially reduced because single nucleotides are covered and sequenced several times within CCS reads
- TAGET is designed for Iso-seq full-length transcript data (Linux CentOS, R>=3.3, python3 with rpy2/pandas/numpy); it is not intended for short-read RNA-seq

## Evidence

- [discussion] Single-read CCS cluster filter removes ~80% of reads: "it is suggested that clusters with only one CCS read should be filtered because of their lower quality. This step filters about 80% of the CCS reads."
- [discussion] Filtered reads come from true expressed isoforms: "Many of these filtered reads come from true expressed isoforms. More advanced computational tools that can properly use these CCS reads should be able to significantly increase the detected genes"
- [readme] CCS read FASTA is the input to TAGET fusion detection: "`${i}.fa: the CCS read from Pacbio platform`"
- [methods] TransAnnot invocation for full-length transcript FASTA: "python TransAnnot.py -f [fasta] -g [genome fasta] -o [output directory] -a[annot gtf] -p [process]"
