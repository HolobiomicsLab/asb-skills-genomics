---
name: isoform-collapsing-long-read-rna
description: Use when you have splice-aligned FLNC reads (SAM from minimimap2) in
  a long-read isoform sequencing workflow and need a nonredundant isoform set (GTF,
  representative FASTA, group file) for downstream QC.
when_to_use_negative:
- Input reads have not yet been splice-aligned to a reference genome — the script
  consumes a coordinate-sorted SAM, not raw CCS or FLNC fastq alone
- You are starting from already-collapsed or reference-annotated isoforms (e.g., an
  Ensembl GTF) rather than per-read alignments needing redundancy removal
- Your pipeline can use the official isoseq3 collapse tool instead — the cDNA_Cupcake
  README states isoseq3 collapse is now the preferred tool and Cupcake will remove
  redundant scripts
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0080
tools:
- name: cDNA_Cupcake
  role: Provides collapse_isoforms_by_sam.py, the collapse script that merges mapped
    FLNC reads with matching splice junctions into nonredundant isoforms using --min-coverage
    0.99 and --min-identity 0.95
  repo: https://github.com/Magdoll/cDNA_Cupcake
- name: SQANTI3
  role: Downstream characterization, quality control, and rules filtering of the collapsed
    nonredundant isoforms
  repo: https://github.com/ConesaLab/SQANTI3
- name: scISA-Tools
  role: Upstream pipeline that generates the FLNC fastq and SAM inputs (classify_by_primer,
    minimap2 alignment) fed into the collapsing step
  repo: https://github.com/shizhuoxing/scISA-Tools
- name: gpertea/gscripts
  role: sam2gff.pl converts FLNC mapped SAM to GFF for gffcompare-based annotation
    mapping adjacent to the collapse step
  repo: https://github.com/gpertea/gscripts
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-38324-9
    title: High-throughput and high-accuracy single-cell RNA isoform analysis using
      PacBio circular consensus sequencing
  source_content_hash: sha256:aa82e072cbe4c64c9c9d1f89db4796426016cb3b44eda57e73dbf28543f8349f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Magdoll/cDNA_Cupcake
  tools:
  - SQANTI3
  - cDNA_Cupcake
  - gpertea/gscripts
  - scISA-Tools
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# isoform-collapsing-long-read-rna

## Summary

Collapse mapped full-length non-chimeric (FLNC) alignments from the HIT-scISOseq pipeline into nonredundant isoform transcripts using cDNA_Cupcake's collapse_isoforms_by_sam.py, merging reads with matching splice junctions under default coverage/identity criteria.

## When to use

Use when you have splice-aligned FLNC reads (SAM from minimimap2) in a long-read isoform sequencing workflow and need a nonredundant isoform set (GTF, representative FASTA, group file) for downstream QC.

## When NOT to use

- Input reads have not yet been splice-aligned to a reference genome — the script consumes a coordinate-sorted SAM, not raw CCS or FLNC fastq alone
- You are starting from already-collapsed or reference-annotated isoforms (e.g., an Ensembl GTF) rather than per-read alignments needing redundancy removal
- Your pipeline can use the official isoseq3 collapse tool instead — the cDNA_Cupcake README states isoseq3 collapse is now the preferred tool and Cupcake will remove redundant scripts

## Inputs

- isoseq_flnc.Transcript.sam — SAM of FLNC reads splice-aligned to the reference genome (e.g., via minimap2 -ax splice)
- isoseq_flnc.Transcript.fastq — FLNC transcript sequences after primer, cellBC, UMI, and polyA trimming
- Sorted SAM file (sorted by chromosome and start coordinate)

## Outputs

- TEST.collapsed.gtf — nonredundant isoform coordinate models in GTF format
- all.collapsed.rep.fa — representative transcript sequences in FASTA
- all.collapsed.group.txt — isoform-to-representative grouping file for downstream isoform count matrices
- Per-sample collapsed FLNC length summaries (kb), e.g., ~2.2-2.5 kb average

## How to apply

First sort the mapped FLNC SAM (produced by minimap2 -ax splice from isoseq_flnc.Transcript.fastq) by chromosome and start coordinate with `sort -k 3,3 -k 4,4n`. Then run collapse_isoforms_by_sam.py with the FLNC fastq (--input ... --fq), the sorted SAM (-s), and an output prefix, keeping the tool's default collapse criteria (--min-coverage 0.99 and --min-identity 0.95) so isoforms sharing the same splice junction chain are merged. Collect the three key outputs: TEST.collapsed.gtf (nonredundant isoform coordinates), all.collapsed.rep.fa (representative transcript sequences), and all.collapsed.group.txt (isoform-to-representative grouping used later for isoform count matrices). Collapsed reads are characterized by length in kilobases and summarized per sample (e.g., ~2.2-2.5 kb average collapsed FLNC length). Note the cDNA_Cupcake README now recommends transitioning to `isoseq3 collapse` as the official equivalent, though this paper's pipeline uses the Cupcake script.

## Related tools

- **cDNA_Cupcake** (Provides collapse_isoforms_by_sam.py, the collapse script that merges mapped FLNC reads with matching splice junctions into nonredundant isoforms using --min-coverage 0.99 and --min-identity 0.95) — https://github.com/Magdoll/cDNA_Cupcake
- **SQANTI3** (Downstream characterization, quality control, and rules filtering of the collapsed nonredundant isoforms) — https://github.com/ConesaLab/SQANTI3
- **scISA-Tools** (Upstream pipeline that generates the FLNC fastq and SAM inputs (classify_by_primer, minimap2 alignment) fed into the collapsing step) — https://github.com/shizhuoxing/scISA-Tools
- **gpertea/gscripts** (sam2gff.pl converts FLNC mapped SAM to GFF for gffcompare-based annotation mapping adjacent to the collapse step) — https://github.com/gpertea/gscripts

## Examples

```
sort -k 3,3 -k 4,4n isoseq_flnc.Transcript.sam > isoseq_flnc.Transcript.sorted.sam && collapse_isoforms_by_sam.py --input isoseq_flnc.Transcript.fastq --fq -s isoseq_flnc.Transcript.sorted.sam -o TEST
```

## Evaluation signals

- TEST.collapsed.gtf exists and is a well-formed GTF containing only nonredundant isoform models
- all.collapsed.rep.fa representative sequences and all.collapsed.group.txt are consistent with TEST.collapsed.gtf (every representative and grouping maps to a collapsed model) for downstream SQANTI3 characterization
- Collapse used default thresholds --min-coverage 0.99 and --min-identity 0.95; rerunning with the sorted SAM reproduces the same grouping
- Collapsed read length distribution falls in the expected ~2.2-2.5 kb average collapsed FLNC length range per sample
- group.txt group counts sum to the number of input FLNC reads, enabling isoform count matrices via scIsoform_matrix

## Limitations

- cDNA_Cupcake maintainers no longer address bugs for functions with isoseq equivalents and plan to remove redundant scripts (e.g., collapse_isoforms_by_sam.py) starting 2023; isoseq3 collapse is the preferred replacement
- Only demonstrated on HIT-scISOseq data from 10× Genomics single-cell libraries; compatibility with other platforms is claimed but not shown
- Collapse merges strictly by matching splice junctions under the 0.99 coverage / 0.95 identity defaults; isoforms differing only slightly but sharing junctions are collapsed together

## Evidence

- [discussion] The cDNA_Cupcake (https://github.com/Magdoll/cDNA_Cupcake) Python script "collapse_isoforms_by_sam.py" was used: "The cDNA_Cupcake (https://github.com/Magdoll/cDNA_Cupcake) Python script "collapse_isoforms_by_sam.py" was used"
- [discussion] SQANTI3 was used for characterization, QC, and rules filter of nonredundant isoforms: "SQANTI3 (https://github.com/ConesaLab/SQANTI3)25 was used for the characterization, quality control, and rules filter of nonredundant isoforms"
- [readme] isoseq3 collapse is now the preferred tool over collapse_isoforms_by_sam.py: [evidence span withheld — anchor status: external_doc]
- [methods] we using cDNA_Cupcake python script collapse_isoforms_by_sam.py: "we using `cDNA_Cupcake`(https://github.com/Magdoll/cDNA_Cupcake) python script `collapse_isoforms_by_sam.py`"
- [abstract] HIT-scISOseq can yield >10 million high-accuracy long-reads in a single PacBio Sequel II SMRT Cell 8M: "HIT-scISOseq can yield >10 million high-accuracy long-reads in a single PacBio Sequel II SMRT Cell 8M."
