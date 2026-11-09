---
name: sam-coordinate-sorting
description: Use when mapped FLNC SAM alignments (e.g. isoseq_flnc.Transcript.sam)
  must be position-sorted before cDNA_Cupcake collapse_isoforms_by_sam.py can collapse
  redundant isoforms.
when_to_use_negative:
- Input alignments are already position-sorted (e.g. minimap2 output piped through
  samtools sort) and the downstream tool accepts sorted or coordinate-grouped input
  directly
- The task is splice-junction annotation comparison (gffcompare/filter.tmap.pl) rather
  than isoform collapsing — that step uses the unsorted SAM converted to GFF via sam2gff.pl
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3170
tools:
- name: cDNA_Cupcake
  role: 'Downstream consumer: its collapse_isoforms_by_sam.py script requires the
    coordinate-sorted SAM to collapse mapped FLNC reads into nonredundant isoforms'
  repo: https://github.com/Magdoll/cDNA_Cupcake
- name: scISA-Tools
  role: Produces the upstream FLNC FASTQ and SAM alignments (classify_by_primer, minimap2
    step) that this sorting step operates on within the HIT-scISOseq pipeline
  repo: github.com/shizhuoxing/scISA-Tools
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-38324-9
    title: High-throughput and high-accuracy single-cell RNA isoform analysis using
      PacBio circular consensus sequencing
  source_content_hash: sha256:45059dc8f066ed40e0ed8d7c579cf82c4949e5a6b26b4bbdaf8f3d48ccc2af73
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Magdoll/cDNA_Cupcake
  tools:
  - cDNA_Cupcake
  - scISA-Tools
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# sam-coordinate-sorting

## Summary

Sort the SAM file of mapped HIT-scISOseq FLNC alignments by chromosome and start coordinate so that the cDNA_Cupcake collapse_isoforms_by_sam.py script can merge reads with matching splice junctions into nonredundant isoform models.

## When to use

Use when mapped FLNC SAM alignments (e.g. isoseq_flnc.Transcript.sam) must be position-sorted before cDNA_Cupcake collapse_isoforms_by_sam.py can collapse redundant isoforms.

## When NOT to use

- Input alignments are already position-sorted (e.g. minimap2 output piped through samtools sort) and the downstream tool accepts sorted or coordinate-grouped input directly
- The task is splice-junction annotation comparison (gffcompare/filter.tmap.pl) rather than isoform collapsing — that step uses the unsorted SAM converted to GFF via sam2gff.pl

## Inputs

- SAM file of FLNC reads aligned to the reference genome (isoseq_flnc.Transcript.sam from minimap2 -ax splice)
- Reference genome FASTA (for the upstream minimap2 alignment)

## Outputs

- Coordinate-sorted SAM file (isoseq_flnc.Transcript.sorted.sam) suitable for collapse_isoforms_by_sam.py

## How to apply

After minimap2 aligns the FLNC transcript FASTQ to the reference genome, sort the resulting SAM (isoseq_flnc.Transcript.sam) with Unix sort keyed on column 3 (chromosome) then column 4 numerically (start coordinate): sort -k 3,3 -k 4,4n. The output, isoseq_flnc.Transcript.sorted.sam, is a required input format for collapse_isoforms_by_sam.py, which is invoked with the FLNC FASTQ (-s sorted SAM, --fq) and applies default collapse criteria (--min-coverage 0.99, --min-identity 0.95) to merge isoforms sharing splice junctions. The rationale is that the collapse algorithm expects records grouped by genomic locus so redundant full-length reads can be grouped and a representative sequence chosen. Validation is by downstream consistency: the collapsed outputs (TEST.collapsed.gtf, all.collapsed.rep.fa, all.collapsed.group.txt) must exist, form a well-formed nonredundant GTF, and the group file must map every input isoform to a representative.

## Related tools

- **cDNA_Cupcake** (Downstream consumer: its collapse_isoforms_by_sam.py script requires the coordinate-sorted SAM to collapse mapped FLNC reads into nonredundant isoforms) — https://github.com/Magdoll/cDNA_Cupcake
- **scISA-Tools** (Produces the upstream FLNC FASTQ and SAM alignments (classify_by_primer, minimap2 step) that this sorting step operates on within the HIT-scISOseq pipeline) — github.com/shizhuoxing/scISA-Tools

## Examples

```
sort -k 3,3 -k 4,4n isoseq_flnc.Transcript.sam > isoseq_flnc.Transcript.sorted.sam
```

## Evaluation signals

- Sorted SAM preserves the same number of alignment records as the input isoseq_flnc.Transcript.sam (sort is lossless)
- Records are non-decreasing by column 3 (chromosome) and column 4 (start position) throughout the file
- collapse_isoforms_by_sam.py runs to completion on the sorted SAM without ordering errors and emits TEST.collapsed.gtf
- all.collapsed.rep.fa and all.collapsed.group.txt are consistent with the GTF: every collapsed isoform has a representative sequence and a group entry
- Collapsed models match splice junctions under the tool's defaults (--min-coverage 0.99, --min-identity 0.95) with ~2.2-2.5 kb average collapsed FLNC length as a plausible sanity range

## Limitations

- cDNA_Cupcake's collapse_isoforms_by_sam.py is deprecated upstream: the README states 'isoseq3 collapse' is now the preferred tool and redundant Cupcake scripts will be removed from 2023 onward
- Unix sort is not a dedicated SAM sorter; it does not add/validate an @HD SO:coordinate header, so header consistency with the SAM spec should be checked separately

## Evidence

- [other] Sort the mapped FLNC alignments by chromosome and start coordinate with: sort -k 3,3 -k 4,4n isoseq_flnc.Transcript.sam > isoseq_flnc.Transcript.sorted.sam.: "sort -k 3,3 -k 4,4n isoseq_flnc.Transcript.sam > isoseq_flnc.Transcript.sorted.sam"
- [other] Collapse command and default criteria as documented in the source card workflow.: [evidence span withheld — anchor status: not_found]
- [methods] README description of the cDNA_Cupcake script used for collapsing.: "we using `cDNA_Cupcake`(https://github.com/Magdoll/cDNA_Cupcake) python script `collapse_isoforms_by_sam.py`"
- [readme] README deprecation note about preferring the official isoseq tool.: [evidence span withheld — anchor status: external_doc]
- [readme] README mapping step that produces the SAM input to this sorting step.: "minimap2 -ax splice -t 6 -uf --secondary=no -C5 -t 5 ref.genome.fa isoseq_flnc.Transcript.fastq > isoseq_flnc.Transcript.sam"
