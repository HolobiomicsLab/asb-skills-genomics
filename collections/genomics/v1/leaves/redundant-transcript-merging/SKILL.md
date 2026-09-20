---
name: redundant-transcript-merging
description: Use when you have a coordinate-sorted SAM of mapped FLNC reads (e.g.,
  isoseq_flnc.Transcript.sorted.sam) and need to remove redundant isoforms sharing
  splice junctions before QC or count-matrix generation.
when_to_use_negative:
- You want the officially supported modern equivalent — the cDNA_Cupcake README states
  isoseq3 collapse is now the preferred tool over collapse_isoforms_by_sam.py
- Input alignments are unsorted or not full-length FLNC reads — the script requires
  chromosome/start-coordinate-sorted mapped FLNC alignments
- The goal is isoform count matrix generation (scIsoform_matrix) rather than collapsing
  — that step consumes the group file produced here, not produces it
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: cDNA_Cupcake
  role: Provides collapse_isoforms_by_sam.py, which merges redundant FLNC isoforms
    with matching splice junctions using --min-coverage 0.99 and --min-identity 0.95
  repo: https://github.com/Magdoll/cDNA_Cupcake
- name: SQANTI3
  role: Downstream characterization, quality control, and rules filtering of the nonredundant
    isoforms produced by the collapse step
  repo: https://github.com/ConesaLab/SQANTI3
- name: scISA-Tools
  role: Upstream HIT-scISOseq pipeline that produces the mapped FLNC SAM and FASTQ
    inputs to the collapse step
  repo: https://github.com/shizhuoxing/scISA-Tools
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-38324-9
    title: High-throughput and high-accuracy single-cell RNA isoform analysis using
      PacBio circular consensus sequencing
  source_content_hash: sha256:f46beb9088ff675f8ddc4e706cb80abaf2cbcef17ca469be1c5ef9261f323b70
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Magdoll/cDNA_Cupcake
  tools:
  - SQANTI3
  - cDNA_Cupcake
  - scISA-Tools
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# redundant-transcript-merging

## Summary

Collapse redundant mapped FLNC alignments into nonredundant isoform transcripts using cDNA_Cupcake's collapse_isoforms_by_sam.py, a key HIT-scISOseq step before SQANTI3 characterization. The collapse merges reads sharing identical splice junctions and yields GTF, representative FASTA, and grouping files used downstream.

## When to use

Use when you have a coordinate-sorted SAM of mapped FLNC reads (e.g., isoseq_flnc.Transcript.sorted.sam) and need to remove redundant isoforms sharing splice junctions before QC or count-matrix generation.

## When NOT to use

- You want the officially supported modern equivalent — the cDNA_Cupcake README states isoseq3 collapse is now the preferred tool over collapse_isoforms_by_sam.py
- Input alignments are unsorted or not full-length FLNC reads — the script requires chromosome/start-coordinate-sorted mapped FLNC alignments
- The goal is isoform count matrix generation (scIsoform_matrix) rather than collapsing — that step consumes the group file produced here, not produces it

## Inputs

- Coordinate-sorted SAM of FLNC reads aligned to the reference genome (isoseq_flnc.Transcript.sorted.sam)
- FLNC transcript FASTQ (isoseq_flnc.Transcript.fastq)

## Outputs

- TEST.collapsed.gtf — nonredundant isoform coordinates
- all.collapsed.rep.fa — representative transcript sequences
- all.collapsed.group.txt — isoform-to-representative grouping for isoform count matrices

## How to apply

First sort the mapped FLNC SAM by chromosome and start coordinate (sort -k 3,3 -k 4,4n on isoseq_flnc.Transcript.sam). Then run cDNA_Cupcake's collapse_isoforms_by_sam.py with the FLNC FASTQ (--input isoseq_flnc.Transcript.fastq --fq) and the sorted SAM (-s), keeping the tool's default collapse criteria --min-coverage 0.99 and --min-identity 0.95, which merge isoforms with matching splice junctions. Collect the three outputs: TEST.collapsed.gtf (nonredundant isoform coordinates), all.collapsed.rep.fa (representative transcript sequences), and all.collapsed.group.txt (isoform-to-representative grouping used later for isoform count matrices). Validate that the GTF is well-formed and consistent with the FASTA and group files before feeding nonredundant isoforms to SQANTI3.

## Related tools

- **cDNA_Cupcake** (Provides collapse_isoforms_by_sam.py, which merges redundant FLNC isoforms with matching splice junctions using --min-coverage 0.99 and --min-identity 0.95) — https://github.com/Magdoll/cDNA_Cupcake
- **SQANTI3** (Downstream characterization, quality control, and rules filtering of the nonredundant isoforms produced by the collapse step) — https://github.com/ConesaLab/SQANTI3
- **scISA-Tools** (Upstream HIT-scISOseq pipeline that produces the mapped FLNC SAM and FASTQ inputs to the collapse step) — https://github.com/shizhuoxing/scISA-Tools

## Examples

```
python collapse_isoforms_by_sam.py --input isoseq_flnc.Transcript.fastq --fq -s isoseq_flnc.Transcript.sorted.sam -o TEST
```

## Evaluation signals

- TEST.collapsed.gtf exists and parses as a well-formed GTF of nonredundant isoform models
- all.collapsed.rep.fa sequences correspond to the representatives listed in TEST.collapsed.gtf
- all.collapsed.group.txt consistently maps each input isoform/read to exactly one representative in the GTF
- Collapsed read lengths fall in the expected ~2.2–2.5 kb average collapsed FLNC length range observed across methods in the article
- Default collapse thresholds (--min-coverage 0.99, --min-identity 0.95) were retained so only isoforms with matching splice junctions merge

## Limitations

- The cDNA_Cupcake README warns that collapse_isoforms_by_sam.py is being superseded: 'isoseq3 collapse' is now the preferred tool and redundant Cupcake scripts will be removed from 2023 (accessible via older tagged versions)
- Only reads sharing identical splice junctions are merged; sequences not passing 99% coverage / 95% identity remain as separate isoforms
- The collapse must run on a coordinate-sorted SAM (sort -k 3,3 -k 4,4n) or the input is invalid for the script

## Evidence

- [methods] Sorted SAM and default collapse criteria: "we using `cDNA_Cupcake`(https://github.com/Magdoll/cDNA_Cupcake) python script `collapse_isoforms_by_sam.py`"
- [methods] SQANTI3 consumes collapsed nonredundant isoforms: "we using `SQANTI3`(https://github.com/ConesaLab/SQANTI3) to characterization non-redundant isoforms"
- [readme] Cupcake deprecation notice: [evidence span withheld — anchor status: external_doc]
- [full_text] Collapsed FLNC length range: [evidence span withheld — anchor status: not_found]
