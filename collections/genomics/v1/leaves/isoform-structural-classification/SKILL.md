---
name: isoform-structural-classification
description: Use when you have a collapsed representative isoform FASTA (e.g., all.collapsed.rep.fa
  from cDNA_Cupcake's collapse_isoforms_by_sam.py) and need SQANTI3 structural-category
  assignments and per-sample retained-isoform counts.
when_to_use_negative:
- Input reads have not yet been demultiplexed and collapsed (e.g., raw CCS reads or
  FLNC reads) — SQANTI3 QC requires a nonredundant isoform set, so scISA-Tools demultiplexing
  and collapse_isoforms_by_sam.py must come first
- The analysis goal is gene-level quantification (scGene_matrix) or single-cell isoform
  count matrices (scIsoform_matrix) rather than transcript structural characterization
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
tools:
- name: SQANTI3
  role: Runs sqanti3_qc.py on the collapsed isoform FASTA against ref.genes.gtf and
    ref.genome.fa to assign structural categories and produce TEST_classification.txt
    for characterization, quality control, and rules filtering of nonredundant isoforms
  repo: https://github.com/ConesaLab/SQANTI3
- name: cDNA_Cupcake
  role: Upstream tool whose collapse_isoforms_by_sam.py produces all.collapsed.rep.fa,
    the input to SQANTI3 classification
  repo: https://github.com/Magdoll/cDNA_Cupcake
- name: scISA-Tools
  role: Demultiplexes HIT-scISOseq concatenated reads into single-cell cDNA reads
    (>99.99% accuracy) so that per-sample isoform counts can be tallied by structural
    class
  repo: https://github.com/shizhuoxing/scISA-Tools
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-38324-9
    title: High-throughput and high-accuracy single-cell RNA isoform analysis using
      PacBio circular consensus sequencing
  source_content_hash: sha256:3678b6409c832d152b223844f2bde95d54bbe502f4d68913af0454b991fab5cc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ConesaLab/SQANTI3
  tools:
  - SQANTI3
  - cDNA_Cupcake
  - scISA-Tools
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# isoform-structural-classification

## Summary

Classify collapsed long-read single-cell isoforms into structural categories (FSM, ISM, NIC, NNC, antisense, genic, etc.) using SQANTI3 QC against a reference genome and annotation, then tally per-class isoform frequencies as an acceptance check on the HIT-scISOseq corneal limbus dataset.

## When to use

Use when you have a collapsed representative isoform FASTA (e.g., all.collapsed.rep.fa from cDNA_Cupcake's collapse_isoforms_by_sam.py) and need SQANTI3 structural-category assignments and per-sample retained-isoform counts.

## When NOT to use

- Input reads have not yet been demultiplexed and collapsed (e.g., raw CCS reads or FLNC reads) — SQANTI3 QC requires a nonredundant isoform set, so scISA-Tools demultiplexing and collapse_isoforms_by_sam.py must come first
- The analysis goal is gene-level quantification (scGene_matrix) or single-cell isoform count matrices (scIsoform_matrix) rather than transcript structural characterization

## Inputs

- Collapsed representative isoform FASTA (all.collapsed.rep.fa) from cDNA_Cupcake collapse_isoforms_by_sam.py
- Reference genome FASTA (ref.genome.fa)
- Reference annotation GTF (ref.genes.gtf)

## Outputs

- SQANTI3 classification file TEST_classification.txt with per-isoform structural categories (FSM, ISM, NIC, NNC, antisense, genic)
- Per-class isoform frequency tally / class distribution summary
- Per-sample retained isoform counts (s1: 29,392; s2: 31,793)

## How to apply

Take the collapsed representative isoform FASTA (all.collapsed.rep.fa) produced upstream with cDNA_Cupcake's collapse_isoforms_by_sam.py and run SQANTI3's sqanti3_qc.py against the reference annotation (ref.genes.gtf) and genome (ref.genome.fa) with --isoAnnotLite and -t 30, writing to an output directory such as ./TEST. Parse the resulting TEST_classification.txt to tally isoform structural categories (FSM, ISM, NIC, NNC, antisense, genic). Use the per-class counts both to characterize the nonredundant isoform set and as acceptance criteria: retained counts must equal 29,392 for sample s1 and 31,793 for sample s2. Rationale: SQANTI3 compares each collapsed transcript's splice junction chain to annotated transcripts, so the structural-category distribution (with FSM expected to dominate at the single-cell level, as in Fig. 4c) validates that upstream collapse and demultiplexing did not distort transcript structure.

## Related tools

- **SQANTI3** (Runs sqanti3_qc.py on the collapsed isoform FASTA against ref.genes.gtf and ref.genome.fa to assign structural categories and produce TEST_classification.txt for characterization, quality control, and rules filtering of nonredundant isoforms) — https://github.com/ConesaLab/SQANTI3
- **cDNA_Cupcake** (Upstream tool whose collapse_isoforms_by_sam.py produces all.collapsed.rep.fa, the input to SQANTI3 classification) — https://github.com/Magdoll/cDNA_Cupcake
- **scISA-Tools** (Demultiplexes HIT-scISOseq concatenated reads into single-cell cDNA reads (>99.99% accuracy) so that per-sample isoform counts can be tallied by structural class) — https://github.com/shizhuoxing/scISA-Tools

## Examples

```
sqanti3_qc.py all.collapsed.rep.fa ref.genes.gtf ref.genome.fa --isoAnnotLite -t 30 -o TEST
```

## Evaluation signals

- TEST_classification.txt exists in the output directory (./TEST) with a structural category (FSM, ISM, NIC, NNC, antisense, genic) per isoform
- Retained isoform counts match 29,392 for sample s1 and 31,793 for sample s2
- FSM (full splice match) is the most abundant isoform class at the single-cell level in both samples, consistent with Fig. 4c
- Every isoform in all.collapsed.rep.fa appears exactly once in the classification output (one row per collapsed transcript)

## Limitations

- Acceptance counts (29,392 / 31,793) depend on the exact upstream demultiplexing and collapse settings; changes to cellBC correction (posterior > 0.975) or UMI filters (-minUMIcount 3, detection in ≥5 cells) will shift retained counts
- cDNA_Cupcake's collapse_isoforms_by_sam.py is being deprecated in favor of official isoseq3 collapse, so the upstream FASTA may differ with newer tooling
- The study characterized isoforms on the Macaca fascicularis reference (Ensembl Macaca_fascicularis_5.0.99); category distributions are genome/annotation-dependent

## Evidence

- [discussion] SQANTI3 was used for the characterization, quality control, and rules filter of nonredundant isoforms: "SQANTI3 (https://github.com/ConesaLab/SQANTI3)25 was used for the characterization, quality control, and rules filter of nonredundant isoforms"
- [methods] SQANTI3 used for characterization of nonredundant isoforms: "we using `SQANTI3`(https://github.com/ConesaLab/SQANTI3) to characterization non-redundant isoforms"
- [methods] cDNA_Cupcake collapse script used upstream: "we using `cDNA_Cupcake`(https://github.com/Magdoll/cDNA_Cupcake) python script `collapse_isoforms_by_sam.py`"
- [readme] Cupcake collapse deprecation note: [evidence span withheld — anchor status: external_doc]
