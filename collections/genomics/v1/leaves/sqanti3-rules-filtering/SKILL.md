---
name: sqanti3-rules-filtering
description: Use when a collapsed representative isoform FASTA (e.g. all.collapsed.rep.fa
  from cDNA_Cupcake collapse_isoforms_by_sam.py) needs structural classification against
  a reference annotation and genome, with per-class counts and post-filter retained
  isoform numbers.
when_to_use_negative:
- Input is raw CCS/FLNC reads or uncollapsed alignments — collapse isoforms with cDNA_Cupcake
  collapse_isoforms_by_sam.py first; SQANTI3 expects nonredundant isoforms
- Input is an isoform/gene expression matrix from scIsoform_matrix or scGene_matrix
  — no structural classification is needed there
- Reference annotation and genome for the species are unavailable, since structural-category
  assignment (FSM/ISM/NIC/NNC) depends on comparison against ref.genes.gtf and ref.genome.fa
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0797
tools:
- name: SQANTI3
  role: Performs characterization, quality control, and rules filtering of nonredundant
    isoforms; sqanti3_qc.py assigns structural categories and emits the classification
    file
  repo: https://github.com/ConesaLab/SQANTI3
- name: cDNA_Cupcake
  role: Upstream collapse_isoforms_by_sam.py produces the collapsed representative
    isoform FASTA that serves as SQANTI3's input
  repo: https://github.com/Magdoll/cDNA_Cupcake
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-38324-9
    title: High-throughput and high-accuracy single-cell RNA isoform analysis using
      PacBio circular consensus sequencing
  source_content_hash: sha256:9b6ff7d8d9fe30a42ffa227434b706e2ff56d6bc9951b78f4023ecc96ac97bcd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ConesaLab/SQANTI3
  tools:
  - SQANTI3
  - cDNA_Cupcake
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# sqanti3-rules-filtering

## Summary

Characterize, quality-control, and apply the SQANTI3 rules filter to nonredundant collapsed isoforms, tallying structural categories (FSM, ISM, NIC, NNC, antisense, genic) and tracking how many isoforms are retained per sample.

## When to use

Use when a collapsed representative isoform FASTA (e.g. all.collapsed.rep.fa from cDNA_Cupcake collapse_isoforms_by_sam.py) needs structural classification against a reference annotation and genome, with per-class counts and post-filter retained isoform numbers.

## When NOT to use

- Input is raw CCS/FLNC reads or uncollapsed alignments — collapse isoforms with cDNA_Cupcake collapse_isoforms_by_sam.py first; SQANTI3 expects nonredundant isoforms
- Input is an isoform/gene expression matrix from scIsoform_matrix or scGene_matrix — no structural classification is needed there
- Reference annotation and genome for the species are unavailable, since structural-category assignment (FSM/ISM/NIC/NNC) depends on comparison against ref.genes.gtf and ref.genome.fa

## Inputs

- Collapsed representative isoform FASTA (all.collapsed.rep.fa) from cDNA_Cupcake collapse_isoforms_by_sam.py
- Reference annotation GTF (ref.genes.gtf, Ensembl Macaca_fascicularis_5.0.99)
- Reference genome FASTA (ref.genome.fa)

## Outputs

- SQANTI3 classification file (TEST_classification.txt) with per-isoform structural categories
- Tally of isoforms per structural category (FSM, ISM, NIC, NNC, antisense, genic, etc.)
- Retained isoform counts after rules filtering per sample (29,392 for s1; 31,793 for s2)
- Class distribution summary written alongside TEST_classification.txt

## How to apply

Run SQANTI3 QC (sqanti3_qc.py) on the collapsed isoform FASTA with the reference GTF (ref.genes.gtf) and genome (ref.genome.fa), writing to an output directory (./TEST) with --isoAnnotLite and 30 threads (-t 30). Parse the classification output (TEST_classification.txt) to tally each isoform's structural category (FSM, ISM, NIC, NNC, antisense, genic, etc.). Apply the SQANTI3 rules filter to remove suspect isoforms, then count retained isoforms per sample — in the HIT-scISOseq corneal limbus dataset this yielded 29,392 isoforms for s1 and 31,793 for s2. Validate by confirming FSM (full splice match) is the most abundant isoform class at the single-cell level in both samples, consistent with the paper's Fig. 4c class distribution. Write the class distribution summary alongside the SQANTI3 classification file so counts can be re-checked against the acceptance criteria.

## Related tools

- **SQANTI3** (Performs characterization, quality control, and rules filtering of nonredundant isoforms; sqanti3_qc.py assigns structural categories and emits the classification file) — https://github.com/ConesaLab/SQANTI3
- **cDNA_Cupcake** (Upstream collapse_isoforms_by_sam.py produces the collapsed representative isoform FASTA that serves as SQANTI3's input) — https://github.com/Magdoll/cDNA_Cupcake

## Examples

```
python sqanti3_qc.py ./all.collapsed.rep.fa ref.genes.gtf ref.genome.fa --isoAnnotLite -t 30 -o TEST
```

## Evaluation signals

- TEST_classification.txt exists in the output directory with a structural category (FSM, ISM, NIC, NNC, antisense, genic, etc.) per isoform
- Retained isoform counts after QC/filtering equal 29,392 for sample s1 and 31,793 for sample s2
- FSM (full splice match) is the most abundant isoform class at the single-cell level in both samples, matching the reported class distribution (Fig. 4c)
- Class distribution summary is written alongside the SQANTI3 classification file so per-category frequencies can be recomputed from TEST_classification.txt

## Limitations

- Retained-count acceptance criteria (29,392 and 31,793) are specific to the two corneal limbus samples and this particular upstream collapse; other inputs will yield different counts and cannot use these numbers as validation
- The study only demonstrated the workflow on two samples (one animal each), with no randomization or blinding, so per-sample counts may not generalize
- cDNA_Cupcake's collapse_isoforms_by_sam.py is being deprecated in favor of official isoseq3 collapse, which may shift upstream results feeding into SQANTI3

## Evidence

- [discussion] SQANTI3 (https://github.com/ConesaLab/SQANTI3)25 was used for the characterization, quality control, and rules filter of nonredundant isoforms: "SQANTI3 (https://github.com/ConesaLab/SQANTI3)25 was used for the characterization, quality control, and rules filter of nonredundant isoforms"
- [methods] sqanti3_qc.py invocation with ref.genes.gtf and ref.genome.fa: "we using `SQANTI3`(https://github.com/ConesaLab/SQANTI3) to characterization non-redundant isoforms"
- [results] Retained isoform counts and FSM dominance from the source task card: [evidence span withheld — anchor status: not_found]
- [readme] SQANTI3 README on long read-defined transcriptome characterization: [evidence span withheld — anchor status: external_doc]
