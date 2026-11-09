---
name: single-cell-isoform-count-verification
description: Use when a collapsed long-read single-cell isoform FASTA (e.g. all.collapsed.rep.fa
  from cDNA_Cupcake) must be classified with SQANTI3 and the resulting per-sample
  retained isoform counts and structural-category distribution (FSM/ISM/NIC/NNC) need
  validation against reported values.
when_to_use_negative:
- Input is raw CCS/FLNC reads that have not yet been collapsed to nonredundant isoforms
  — run collapse_isoforms_by_sam.py first
- The goal is gene-level single-cell quantification (scGene_matrix / scIsoform_matrix
  UMI counting), not transcript structural classification
- Data derive from short-read scRNA-Seq (e.g. 10x 3' gene expression), where structural
  categories like FSM/NIC are not applicable
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SQANTI3
  role: Characterization, quality control, and rules filter of nonredundant isoforms;
    produces TEST_classification.txt used for count verification
  repo: https://github.com/ConesaLab/SQANTI3
- name: cDNA_Cupcake
  role: Upstream collapse_isoforms_by_sam.py produces the collapsed representative
    isoform FASTA that SQANTI3 QC consumes
  repo: https://github.com/Magdoll/cDNA_Cupcake
- name: scISA-Tools
  role: Upstream demultiplexing of HIT-scISOseq concatenated reads into single-cell
    cDNA reads feeding the collapsed isoform set
  repo: https://github.com/shizhuoxing/scISA-Tools
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-38324-9
    title: High-throughput and high-accuracy single-cell RNA isoform analysis using
      PacBio circular consensus sequencing
  source_content_hash: sha256:399784645e719fbba355d640fcc69f5b6d39e35abf45c874ff4a1762d92af4f4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ConesaLab/SQANTI3
  tools:
  - SQANTI3
  - cDNA_Cupcake
  - scISA-Tools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# single-cell-isoform-count-verification

## Summary

Verifies the characterization and filtering step of the HIT-scISOseq single-cell isoform pipeline by running SQANTI3 QC on collapsed isoforms and checking that retained isoform counts and the dominant isoform class reproduce the reported results (29,392 isoforms for sample s1, 31,793 for s2, FSM most abundant).

## When to use

Use when a collapsed long-read single-cell isoform FASTA (e.g. all.collapsed.rep.fa from cDNA_Cupcake) must be classified with SQANTI3 and the resulting per-sample retained isoform counts and structural-category distribution (FSM/ISM/NIC/NNC) need validation against reported values.

## When NOT to use

- Input is raw CCS/FLNC reads that have not yet been collapsed to nonredundant isoforms — run collapse_isoforms_by_sam.py first
- The goal is gene-level single-cell quantification (scGene_matrix / scIsoform_matrix UMI counting), not transcript structural classification
- Data derive from short-read scRNA-Seq (e.g. 10x 3' gene expression), where structural categories like FSM/NIC are not applicable

## Inputs

- Collapsed representative isoform FASTA (all.collapsed.rep.fa) from cDNA_Cupcake collapse_isoforms_by_sam.py
- Reference genome FASTA (ref.genome.fa)
- Reference annotation GTF (ref.genes.gtf)

## Outputs

- SQANTI3 classification file TEST_classification.txt with per-isoform structural categories
- Per-sample retained isoform counts (29,392 for s1; 31,793 for s2)
- Isoform structural category frequency summary (FSM, ISM, NIC, NNC, antisense, genic) with FSM as top class

## How to apply

Take the collapsed representative isoform FASTA produced upstream with cDNA_Cupcake's collapse_isoforms_by_sam.py and run SQANTI3 QC (sqanti3_qc.py) against the reference annotation (ref.genes.gtf) and genome (ref.genome.fa) with --isoAnnotLite and 30 threads (-t 30), writing outputs to ./TEST. Parse TEST_classification.txt to tally isoform structural categories (FSM, ISM, NIC, NNC, antisense, genic). Acceptance requires 29,392 retained isoforms for sample s1 and 31,793 for sample s2, and that full splice match (FSM) is the most abundant class at the single-cell level, consistent with the reported class distribution (Fig. 4c). Write the class distribution summary alongside TEST_classification.txt so both artifacts travel together.

## Related tools

- **SQANTI3** (Characterization, quality control, and rules filter of nonredundant isoforms; produces TEST_classification.txt used for count verification) — https://github.com/ConesaLab/SQANTI3
- **cDNA_Cupcake** (Upstream collapse_isoforms_by_sam.py produces the collapsed representative isoform FASTA that SQANTI3 QC consumes) — https://github.com/Magdoll/cDNA_Cupcake
- **scISA-Tools** (Upstream demultiplexing of HIT-scISOseq concatenated reads into single-cell cDNA reads feeding the collapsed isoform set) — https://github.com/shizhuoxing/scISA-Tools

## Examples

```
sqanti3_qc.py ./all.collapsed.rep.fa ref.genes.gtf ref.genome.fa -o TEST --isoAnnotLite -t 30
```

## Evaluation signals

- TEST_classification.txt exists in ./TEST and assigns a structural category to every input isoform
- Retained isoform count equals 29,392 for sample s1
- Retained isoform count equals 31,793 for sample s2
- FSM (full splice match) is the most frequent structural category in both samples, matching the reported class distribution (Fig. 4c)
- Class distribution summary file is written alongside TEST_classification.txt

## Limitations

- cDNA_Cupcake's collapse_isoforms_by_sam.py is deprecated in favor of isoseq3 collapse as of the 08/23/2022 README update, so reproducing the exact collapsed input may require older tagged versions
- Counts depend on the exact genome/annotation versions (Ensembl Macaca_fascicularis_5.0.99 for the corneal limbus data); changed references will shift FSM/NIC/NNC frequencies
- The article applies filtering rules and reports counts for only two samples (one animal each); counts are not guaranteed to generalize to other datasets

## Evidence

- [discussion] SQANTI3 was used for characterization, QC and filtering of nonredundant isoforms: "SQANTI3 (https://github.com/ConesaLab/SQANTI3)25 was used for the characterization, quality control, and rules filter of nonredundant isoforms"
- [methods] SQANTI3 used in methods for isoform characterization: "we using `SQANTI3`(https://github.com/ConesaLab/SQANTI3) to characterization non-redundant isoforms"
- [methods] cDNA_Cupcake collapse script feeds the pipeline: "we using `cDNA_Cupcake`(https://github.com/Magdoll/cDNA_Cupcake) python script `collapse_isoforms_by_sam.py`"
- [readme] Cupcake deprecation of collapse_isoforms_by_sam.py: [evidence span withheld — anchor status: not_found]
