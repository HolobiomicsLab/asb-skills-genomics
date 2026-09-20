---
name: long-read-isoform-quality-control
description: Use when you have a collapsed representative isoform FASTA from long-read
  single-cell isoform sequencing (e.g., HIT-scISOseq) and need SQANTI3 structural
  classification plus rules filtering to quantify retained isoforms per sample.
when_to_use_negative:
- Input is still raw CCS or FLNC reads that have not yet been collapsed into non-redundant
  isoforms — collapse first (e.g., cDNA_Cupcake collapse_isoforms_by_sam.py or isoseq3
  collapse)
- Goal is demultiplexing HIT-scISOseq concatenated reads into single-cell cDNA reads
  — use scISA-Tools instead
- Reference annotation/genome for the sample's species are unavailable, since SQANTI3
  QC classification against ref.genes.gtf/ref.genome.fa is required
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0196
tools:
- name: SQANTI3
  role: Characterization, quality control, and rules filter of non-redundant isoforms;
    produces TEST_classification.txt with structural categories
  repo: https://github.com/ConesaLab/SQANTI3
- name: cDNA_Cupcake
  role: Upstream collapse of aligned reads into the non-redundant representative isoform
    FASTA (collapse_isoforms_by_sam.py) that SQANTI3 QC consumes
  repo: https://github.com/Magdoll/cDNA_Cupcake
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-38324-9
    title: High-throughput and high-accuracy single-cell RNA isoform analysis using
      PacBio circular consensus sequencing
  source_content_hash: sha256:2e281431a1952f8c6edcff1e8862bae99a1cc6ca0e9bf45d5b9b1b2ec33e4a7a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ConesaLab/SQANTI3
  tools:
  - SQANTI3
  - cDNA_Cupcake
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# long-read-isoform-quality-control

## Summary

Characterize, quality-control, and structurally classify non-redundant long-read isoforms with SQANTI3, tallying structural categories (FSM, ISM, NIC, NNC, antisense, genic) and applying filters to obtain the final retained isoform set per sample.

## When to use

Use when you have a collapsed representative isoform FASTA from long-read single-cell isoform sequencing (e.g., HIT-scISOseq) and need SQANTI3 structural classification plus rules filtering to quantify retained isoforms per sample.

## When NOT to use

- Input is still raw CCS or FLNC reads that have not yet been collapsed into non-redundant isoforms — collapse first (e.g., cDNA_Cupcake collapse_isoforms_by_sam.py or isoseq3 collapse)
- Goal is demultiplexing HIT-scISOseq concatenated reads into single-cell cDNA reads — use scISA-Tools instead
- Reference annotation/genome for the sample's species are unavailable, since SQANTI3 QC classification against ref.genes.gtf/ref.genome.fa is required

## Inputs

- Collapsed representative isoform FASTA (all.collapsed.rep.fa) from cDNA_Cupcake collapse_isoforms_by_sam.py
- Reference annotation GTF (ref.genes.gtf), e.g., Ensembl Macaca_fascicularis_5.0.99
- Reference genome FASTA (ref.genome.fa)

## Outputs

- SQANTI3 classification file TEST_classification.txt with per-isoform structural categories
- Tally of isoform structural categories (FSM, ISM, NIC, NNC, antisense, genic, etc.)
- Retained isoform counts after SQANTI3 QC and filtering per sample (29,392 for s1; 31,793 for s2)
- Class distribution summary consistent with Fig. 4c

## How to apply

Start from the collapsed representative isoform FASTA (all.collapsed.rep.fa) produced upstream with cDNA_Cupcake's collapse_isoforms_by_sam.py. Run SQANTI3 QC on this FASTA against the reference annotation (ref.genes.gtf) and genome (ref.genome.fa), writing to output directory ./TEST with --isoAnnotLite and 30 threads (-t 30). Parse the resulting TEST_classification.txt to assign each isoform exactly one structural category (FSM, ISM, NIC, NNC, antisense, genic, etc.) and tally the class distribution. Apply the rules filter and treat the retained counts as acceptance criteria — in this dataset 29,392 isoforms for sample s1 and 31,793 for sample s2. Confirm that FSM (full splice match) is the most abundant class at the single-cell level, consistent with the reported distribution (Fig. 4c), then write the class distribution summary alongside TEST_classification.txt.

## Related tools

- **SQANTI3** (Characterization, quality control, and rules filter of non-redundant isoforms; produces TEST_classification.txt with structural categories) — https://github.com/ConesaLab/SQANTI3
- **cDNA_Cupcake** (Upstream collapse of aligned reads into the non-redundant representative isoform FASTA (collapse_isoforms_by_sam.py) that SQANTI3 QC consumes) — https://github.com/Magdoll/cDNA_Cupcake

## Examples

```
sqanti3_qc.py ./all.collapsed.rep.fa ref.genes.gtf ref.genome.fa -o TEST --isoAnnotLite -t 30
```

## Evaluation signals

- TEST_classification.txt exists in ./TEST and assigns a structural category to every input isoform
- Retained isoform counts after QC and filtering equal 29,392 for sample s1 and 31,793 for sample s2
- FSM (full splice match) is the most abundant structural category by isoform frequency in both samples, matching Fig. 4c
- Every classified isoform falls into exactly one SQANTI3 category from the expected set (FSM, ISM, NIC, NNC, antisense, genic, etc.)

## Limitations

- The retained counts (29,392 s1; 31,793 s2) are dataset-specific acceptance criteria for the corneal limbus samples and will not generalize to other datasets
- cDNA_Cupcake is deprecating collapse_isoforms_by_sam.py in favor of the official isoseq3 collapse; the maintainers state they 'will no longer be addressing bugs' for tools with an isoseq equivalent
- SQANTI3 QC classification depends on reference annotation and genome quality (here Ensembl Macaca_fascicularis_5.0.99); without --isoAnnotLite-compatible annotation resources the functional characterization step is limited

## Evidence

- [results] SQANTI3 QC and filtering retained 29,392 isoforms for sample s1 and 31,793 for sample s2, and FSM (full splice match) was the most abundant isoform type at the single-cell level in both samples.: [evidence span withheld — anchor status: not_found]
- [methods] Methods: SQANTI3 usage: "we using `SQANTI3`(https://github.com/ConesaLab/SQANTI3) to characterization non-redundant isoforms"
- [discussion] Discussion: SQANTI3 role: "SQANTI3 (https://github.com/ConesaLab/SQANTI3)25 was used for the characterization, quality control, and rules filter of nonredundant isoforms"
- [readme] README: Cupcake deprecation of collapse tooling: [evidence span withheld — anchor status: external_doc]
- [full_text] Workflow: SQANTI3 QC invocation parameters: [evidence span withheld — anchor status: not_found]
