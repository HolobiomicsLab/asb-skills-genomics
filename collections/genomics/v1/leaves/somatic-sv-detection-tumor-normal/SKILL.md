---
name: somatic-sv-detection-tumor-normal
description: Use when evaluating whether pangenome graphs built from HiFiCCL-Hifiasm
  versus Hifiasm assemblies improve detection of large somatic SVs in tumor HiFi data
  (e.g., COLO829 30x), with or without a matched normal.
when_to_use_negative:
- The goal is germline SV detection on HG002 (use the germline SV skill with the HPRC_graph-derived
  ground truth instead of the COLO829 somatic truth set)
- SVs smaller than 10,000 bp are the target — this pipeline explicitly restricts evaluation
  to large SVs >10,000 bp
- Input data is short-read or ONT tumor data rather than PacBio HiFi alignments produced
  via minimap2/minigraph as required by minisv
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: minisv
  role: Extracts candidate SVs from PAF alignments and performs tumor-only or paired
    tumor-normal SV calling
  repo: https://github.com/lh3/minisv
- name: minimap2
  role: Aligns COLO829 and COLO829BL HiFi reads to linear references and reciprocally
    between tumor and normal
  repo: https://github.com/lh3/minimap2
- name: minigraph
  role: Aligns tumor reads to the constructed pangenome graphs (GAF output) for graph-based
    SV calling
  repo: https://github.com/lh3/minigraph
- name: Truvari
  role: Compares detected somatic SVs to the COLO829 truth set and computes precision,
    recall, and F1 (v3.2.0)
- name: HiFiCCL
  role: Produces the low-coverage assemblies from which the HiFiCCL-Hifiasm pangenome
    graph is built for improved somatic SV detection
  repo: https://github.com/zjjbuqi/HiFiCCL
- name: hifiasm
  role: Assembles chromosomal bins to generate the Hifiasm graph baseline for comparison
  repo: https://github.com/chhylp123/hifiasm
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/advs.202515308
    title: Reference‐Guided Chromosome‐by‐Chromosome <i>de novo</i> Assembly at Scale
      Using Low‐Coverage High‐Fidelity Long‐Reads with HiFiCCL
  source_content_hash: sha256:8462f0cfc592b056c00cfa067c53663ccfd1df1416afadbb212446d030674308
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minisv
  tools:
  - HiFiCCL
  - Truvari
  - hifiasm
  - minigraph
  - minimap2
  - minisv
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# somatic-sv-detection-tumor-normal

## Summary

Detects somatic structural variants (SVs >10,000 bp) in the COLO829 tumor sample by running minisv on graph/linear alignments in tumor-only mode or in tumor-normal paired mode with COLO829BL, then scoring calls against the COLO829 somatic SV truth set with Truvari v3.2.0.

## When to use

Use when evaluating whether pangenome graphs built from HiFiCCL-Hifiasm versus Hifiasm assemblies improve detection of large somatic SVs in tumor HiFi data (e.g., COLO829 30x), with or without a matched normal.

## When NOT to use

- The goal is germline SV detection on HG002 (use the germline SV skill with the HPRC_graph-derived ground truth instead of the COLO829 somatic truth set)
- SVs smaller than 10,000 bp are the target — this pipeline explicitly restricts evaluation to large SVs >10,000 bp
- Input data is short-read or ONT tumor data rather than PacBio HiFi alignments produced via minimap2/minigraph as required by minisv

## Inputs

- COLO829 tumor HiFi reads (30x) as FASTQ/FASTA
- COLO829BL matched normal HiFi reads (30x) for the paired scenario
- Linear reference genomes GRCh38 and CHM13-T2T
- Pangenome graphs in GFA format (Hifiasm_graph, HiFiCCL-Hifiasm_graph, HPRC_graph)
- COLO829 somatic SV benchmark (truth set) for Truvari comparison

## Outputs

- Candidate SV list from minisv (e.g., COLO829T.rsv)
- Filtered somatic SV calls >10,000 bp per alignment/graph
- Precision, recall, and F1 metrics from Truvari v3.2.0 written to sv_detection_evaluation.csv

## How to apply

Align the COLO829 (30x) HiFi reads to linear references (GRCh38, CHM13-T2T) with minimap2 and to the pangenome graphs (Hifiasm_graph, HiFiCCL-Hifiasm_graph, HPRC_graph) with minigraph, producing PAF/GAF alignments. Run minisv in tumor-only mode on the tumor alignment to extract candidate SVs, keeping only SVs larger than 10,000 bp for evaluation. For the tumor-normal paired scenario, supplement the minisv inputs with reciprocal minimap2 alignments between COLO829 and COLO829BL (30x), extracting the normal sample's candidates with `minisv.js extract`. Compare the resulting SV calls to the COLO829 somatic SV truth set using Truvari v3.2.0 to compute precision, recall, and F1, and write all metrics to sv_detection_evaluation.csv. The rationale is that graph-based calling against pangenome graphs should better represent tumor-specific large SVs than linear-reference-only calling.

## Related tools

- **minisv** (Extracts candidate SVs from PAF alignments and performs tumor-only or paired tumor-normal SV calling) — https://github.com/lh3/minisv
- **minimap2** (Aligns COLO829 and COLO829BL HiFi reads to linear references and reciprocally between tumor and normal) — https://github.com/lh3/minimap2
- **minigraph** (Aligns tumor reads to the constructed pangenome graphs (GAF output) for graph-based SV calling) — https://github.com/lh3/minigraph
- **Truvari** (Compares detected somatic SVs to the COLO829 truth set and computes precision, recall, and F1 (v3.2.0))
- **HiFiCCL** (Produces the low-coverage assemblies from which the HiFiCCL-Hifiasm pangenome graph is built for improved somatic SV detection) — https://github.com/zjjbuqi/HiFiCCL
- **hifiasm** (Assembles chromosomal bins to generate the Hifiasm graph baseline for comparison) — https://github.com/chhylp123/hifiasm

## Examples

```
k8 minisv.js extract -n COLO829BL test/COLO829BL.hs38l.paf.gz > COLO829BL.rsv
```

## Evaluation signals

- Truvari v3.2.0-computed tumor-only precision matches ~51.28% with F1 ~41.23 against the COLO829 somatic SV truth set
- Tumor-normal paired mode achieves precision ~95.23% with F1 ~50.63 against the COLO829 somatic SV truth set
- All evaluated SVs are restricted to >10,000 bp; calls below this threshold must be excluded before scoring
- All precision/recall/F1 values for each graph variant are written to sv_detection_evaluation.csv
- minisv must be supplied with PAF alignments (linear via minimap2, graph via minigraph); GAF-only inputs without the required linear alignments will fail the tumor-normal pairing

## Limitations

- Evaluation is restricted to SVs larger than 10,000 bp; smaller somatic SVs are out of scope for this skill
- Version numbers for HiFiCCL, Hifiasm, and RagTag are not stated in the paper, complicating exact reproduction
- In the germline setting RagTag scaffolding overlooks true SVs (only 62.5% of inversion SVs detected), suggesting alignment-based reference bias also affects graph-based SV calling
- The paper does not report version numbers for minisv or the exact Truvari comparison parameters beyond the version

## Evidence

- [readme] Tumor-only and paired calling via minisv on COLO829 PAF alignments: "k8 minisv.js e -c "k8 minisv.js" -n COLO829T test/COLO829T.hs38l.paf.gz 
  test/COLO829T.chm13g.paf.gz | bash > COLO829T.rsv"
- [readme] Extracting matched-normal candidate SVs: [evidence span withheld — anchor status: external_doc]
- [abstract] Article abstract claim on graph-based somatic SV detection: "improve the detection of specific germline and tumor somatic structural variants based on the pangenome graph"
- [intro] Scope of validated applications including large SV detection: "We validate the effectiveness of HiFiCCL across various applications, including contig‐level assembly, large germline SVs detection (larger than 10,000 bp), chromosome‐level scaffolding"
