---
name: collapsed-isoform-gtf-generation
description: Use when mapped full-length non-chimeric (FLNC) reads in SAM format must
  be collapsed into nonredundant isoform models (GTF) for downstream SQANTI3 characterization
  or isoform count matrices.
when_to_use_negative:
- Input is raw CCS or unclassified reads rather than genome-mapped FLNC SAM alignments
  — primer classification and minimap2 mapping must happen first
- The isoform set has already been collapsed (a collapsed.gtf exists) — re-collapsing
  would not add information and may break the group-file consistency
- Working in the official PacBio isoseq3 ecosystem where isoseq3 collapse is preferred
  over the deprecated Cupcake script
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3572
tools:
- name: cDNA_Cupcake
  role: Provides collapse_isoforms_by_sam.py, which collapses mapped FLNC alignments
    into nonredundant isoforms using --min-coverage 0.99 and --min-identity 0.95
  repo: https://github.com/Magdoll/cDNA_Cupcake
- name: SQANTI3
  role: Downstream characterization, quality control, and rules filtering of the nonredundant
    isoforms produced by this step
  repo: https://github.com/ConesaLab/SQANTI3
- name: scISA-Tools
  role: Upstream pipeline (classify_by_primer, minimap2 mapping) that produces the
    FLNC FASTQ and SAM inputs consumed by the collapsing step
  repo: https://github.com/shizhuoxing/scISA-Tools
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-38324-9
    title: High-throughput and high-accuracy single-cell RNA isoform analysis using
      PacBio circular consensus sequencing
  source_content_hash: sha256:d64d1c136d80213acb79ce808f605cfac4fbddaf1455b1c3dab87b328c5afe9d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Magdoll/cDNA_Cupcake
  tools:
  - SQANTI3
  - cDNA_Cupcake
  - scISA-Tools
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3572
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# collapsed-isoform-gtf-generation

## Summary

Generates a nonredundant isoform transcript set (collapsed GTF plus representative FASTA and group file) from mapped FLNC alignments using the cDNA_Cupcake script collapse_isoforms_by_sam.py, as done in the HIT-scISOseq pipeline.

## When to use

Use when mapped full-length non-chimeric (FLNC) reads in SAM format must be collapsed into nonredundant isoform models (GTF) for downstream SQANTI3 characterization or isoform count matrices.

## When NOT to use

- Input is raw CCS or unclassified reads rather than genome-mapped FLNC SAM alignments — primer classification and minimap2 mapping must happen first
- The isoform set has already been collapsed (a collapsed.gtf exists) — re-collapsing would not add information and may break the group-file consistency
- Working in the official PacBio isoseq3 ecosystem where isoseq3 collapse is preferred over the deprecated Cupcake script

## Inputs

- isoseq_flnc.Transcript.sam — SAM alignments of FLNC reads to the reference genome (e.g., from minimap2 -ax splice)
- isoseq_flnc.Transcript.fastq — full-length non-chimeric transcript sequences with cell barcode, UMI, and polyA trimmed

## Outputs

- TEST.collapsed.gtf — GTF of nonredundant collapsed isoform models
- all.collapsed.rep.fa — representative transcript sequences for collapsed isoforms
- all.collapsed.group.txt — isoform-to-representative grouping used to build isoform count matrices

## How to apply

First sort the mapped FLNC SAM (isoseq_flnc.Transcript.sam, produced by minimap2 -ax splice from isoseq_flnc.Transcript.fastq) by chromosome and start coordinate: sort -k 3,3 -k 4,4n. Then run collapse_isoforms_by_sam.py with --input isoseq_flnc.Transcript.fastq --fq, -s the sorted SAM, and -o TEST, applying the tool's default collapse criteria (--min-coverage 0.99 and --min-identity 0.95) to merge isoforms with matching splice junctions. Collect three outputs: TEST.collapsed.gtf (nonredundant isoform coordinates), all.collapsed.rep.fa (representative transcript sequences), and all.collapsed.group.txt (isoform-to-representative grouping later used for single-cell isoform count matrices). Validate that the GTF is well-formed, contains only nonredundant models, and is consistent with the rep.fa and group file before SQANTI3 quality control.

## Related tools

- **cDNA_Cupcake** (Provides collapse_isoforms_by_sam.py, which collapses mapped FLNC alignments into nonredundant isoforms using --min-coverage 0.99 and --min-identity 0.95) — https://github.com/Magdoll/cDNA_Cupcake
- **SQANTI3** (Downstream characterization, quality control, and rules filtering of the nonredundant isoforms produced by this step) — https://github.com/ConesaLab/SQANTI3
- **scISA-Tools** (Upstream pipeline (classify_by_primer, minimap2 mapping) that produces the FLNC FASTQ and SAM inputs consumed by the collapsing step) — https://github.com/shizhuoxing/scISA-Tools

## Examples

```
collapse_isoforms_by_sam.py --input isoseq_flnc.Transcript.fastq --fq -s isoseq_flnc.Transcript.sorted.sam -o TEST
```

## Evaluation signals

- TEST.collapsed.gtf exists and parses as a well-formed GTF containing only nonredundant isoform models
- all.collapsed.rep.fa contains one representative sequence per collapsed isoform, consistent with the GTF transcript IDs
- all.collapsed.group.txt maps every input FLNC transcript to exactly one representative and is consistent with the GTF
- Collapsed isoforms satisfy the default collapse criteria (splice-junction coverage ≥ 0.99, identity ≥ 0.95)
- Collapsed read lengths summarize to expected ranges (e.g., ~2.2–2.5 kb average collapsed FLNC length per sample)

## Limitations

- cDNA_Cupcake has deprecated collapse_isoforms_by_sam.py in favor of the official isoseq3 collapse tool and no longer fixes bugs in the Cupcake version
- Collapse quality depends on upstream mapping quality filtering; poorly mapped FLNC alignments will produce spurious isoform models
- Default thresholds (--min-coverage 0.99, --min-identity 0.95) merge only isoforms with matching splice junctions; slight junction differences remain separate models

## Evidence

- [methods] Methods citation of the collapsing tool: "we using `cDNA_Cupcake`(https://github.com/Magdoll/cDNA_Cupcake) python script `collapse_isoforms_by_sam.py`"
- [readme] README note on deprecation of the Cupcake collapse script: [evidence span withheld — anchor status: external_doc]
- [methods] Downstream use of collapsed nonredundant isoforms: "we using `SQANTI3`(https://github.com/ConesaLab/SQANTI3) to characterization non-redundant isoforms"
- [readme] README on Cupcake maintenance status: [evidence span withheld — anchor status: not_found]
