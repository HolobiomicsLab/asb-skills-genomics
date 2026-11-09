---
name: collapsed-isoform-annotation-comparison
description: Use when a collapsed representative isoform FASTA (e.g. all.collapsed.rep.fa)
  from long-read single-cell data needs SQANTI3 structural classification and per-sample
  retained-isoform count validation.
when_to_use_negative:
- Input is raw CCS or FLNC reads that have not yet been collapsed into representative
  isoforms (run collapse_isoforms_by_sam.py first).
- No reference genome and annotation GTF are available for the organism, since SQANTI3
  QC requires both to assign structural categories.
- Goal is short-read transcript assembly characterization rather than long-read collapsed
  isoform QC.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: SQANTI3
  role: Runs sqanti3_qc.py for characterization, quality control, and rules filter
    of nonredundant isoforms; produces classification with structural categories
  repo: https://github.com/ConesaLab/SQANTI3
- name: cDNA_Cupcake
  role: Provides collapse_isoforms_by_sam.py, which produces the collapsed representative
    isoform FASTA that is the input to SQANTI3 QC
  repo: https://github.com/Magdoll/cDNA_Cupcake
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-38324-9
    title: High-throughput and high-accuracy single-cell RNA isoform analysis using
      PacBio circular consensus sequencing
  source_content_hash: sha256:15e4012b4440c05da7d1765b08338dffa9a702607d0adc1ae2ad4d2d2e283eaa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ConesaLab/SQANTI3
  tools:
  - SQANTI3
  - cDNA_Cupcake
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# collapsed-isoform-annotation-comparison

## Summary

Characterize and quality-control collapsed non-redundant isoforms (from cDNA_Cupcake's collapse_isoforms_by_sam.py) by running SQANTI3 QC against a reference genome and annotation, then tally structural categories (FSM, ISM, NIC, NNC, etc.) to verify retained isoform counts per sample.

## When to use

Use when a collapsed representative isoform FASTA (e.g. all.collapsed.rep.fa) from long-read single-cell data needs SQANTI3 structural classification and per-sample retained-isoform count validation.

## When NOT to use

- Input is raw CCS or FLNC reads that have not yet been collapsed into representative isoforms (run collapse_isoforms_by_sam.py first).
- No reference genome and annotation GTF are available for the organism, since SQANTI3 QC requires both to assign structural categories.
- Goal is short-read transcript assembly characterization rather than long-read collapsed isoform QC.

## Inputs

- Collapsed representative isoform FASTA (all.collapsed.rep.fa) from collapse_isoforms_by_sam.py
- Reference genome FASTA (ref.genome.fa), e.g. Macaca fascicularis (Ensembl Macaca_fascicularis_5.0.99)
- Reference annotation GTF (ref.genes.gtf)

## Outputs

- SQANTI3 classification file (TEST_classification.txt) with per-isoform structural categories
- Class distribution summary tallying FSM, ISM, NIC, NNC, antisense, genic categories
- Retained isoform counts per sample (29,392 for s1; 31,793 for s2)

## How to apply

Run SQANTI3's sqanti3_qc.py on the collapsed representative isoform FASTA (all.collapsed.rep.fa, produced upstream with cDNA_Cupcake's collapse_isoforms_by_sam.py) using the reference annotation (ref.genes.gtf), genome (ref.genome.fa), --isoAnnotLite and 30 threads (-t 30), writing to an output directory such as ./TEST. Parse the resulting TEST_classification.txt to tally isoform structural categories (FSM, ISM, NIC, NNC, antisense, genic). Apply the reported counts as acceptance criteria: 29,392 retained isoforms for sample s1 and 31,793 for sample s2. Confirm that full splice match (FSM) is the most abundant class at the single-cell level, consistent with the reported class distribution (Fig. 4c). SQANTI3 is used for the characterization, quality control, and rules filter of nonredundant isoforms in this HIT-scISOseq workflow.

## Related tools

- **SQANTI3** (Runs sqanti3_qc.py for characterization, quality control, and rules filter of nonredundant isoforms; produces classification with structural categories) — https://github.com/ConesaLab/SQANTI3
- **cDNA_Cupcake** (Provides collapse_isoforms_by_sam.py, which produces the collapsed representative isoform FASTA that is the input to SQANTI3 QC) — https://github.com/Magdoll/cDNA_Cupcake

## Examples

```
python sqanti3_qc.py ./all.collapsed.rep.fa ref.genes.gtf ref.genome.fa --isoAnnotLite -t 30 -d ./TEST
```

## Evaluation signals

- TEST_classification.txt exists and contains a structural category per isoform (FSM, ISM, NIC, NNC, antisense, genic, etc.).
- Retained isoform count equals 29,392 for sample s1.
- Retained isoform count equals 31,793 for sample s2.
- FSM (full splice match) is the most abundant isoform class at the single-cell level in both samples, consistent with Fig. 4c.

## Limitations

- SQANTI3 structural categories and retained counts depend on the choice of reference genome/annotation (here Ensembl Macaca_fascicularis_5.0.99); different annotations will change FSM/ISM/NIC/NNC proportions.
- The cDNA_Cupcake README notes collapse_isoforms_by_sam.py is being superseded by official isoseq3 collapse, so equivalent Cupcake scripts may be phased out in newer versions.
- Rules filtering after QC will change the number of retained isoforms; counts must be compared to the reported filtered values, not raw classification totals.

## Evidence

- [results] SQANTI3 QC and filtering retained 29,392 isoforms for sample s1 and 31,793 for sample s2: [evidence span withheld — anchor status: not_found]
- [discussion] SQANTI3 used for characterization, QC and rules filter of nonredundant isoforms: "SQANTI3 (https://github.com/ConesaLab/SQANTI3)25 was used for the characterization, quality control, and rules filter of nonredundant isoforms"
- [methods] collapse_isoforms_by_sam.py produces the collapsed isoforms: "we using `cDNA_Cupcake`(https://github.com/Magdoll/cDNA_Cupcake) python script `collapse_isoforms_by_sam.py`"
- [readme] Cupcake scripts superseded by official isoseq software: [evidence span withheld — anchor status: external_doc]
