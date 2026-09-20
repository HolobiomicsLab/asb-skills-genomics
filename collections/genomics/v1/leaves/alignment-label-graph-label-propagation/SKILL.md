---
name: alignment-label-graph-label-propagation
description: Use when reference-based clustering of low-coverage (~5x) PacBio HiFi
  reads has left reads with missing (NA) or misassigned chromosome classes and overlap-based
  label propagation is needed to correct them before per-chromosome de novo assembly.
when_to_use_negative:
- Input reads are noisy long reads (ONT or PacBio CLR) rather than high-fidelity HiFi
  reads — the framework is designed for HiFi at ~5x coverage
- Hi-C or nanopore data integration is required — the article states this is left
  to future efforts and not yet implemented
- Reads already carry consistent, correct 'chr' chromosome classes and the assembly
  is run at sufficient coverage, so overlap-graph class correction adds no value
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: HiFiCCL
  role: Implements the ALG construction, containment collapsing, Z-drop-inspired heuristic
    search and majority-vote label propagation
  repo: https://github.com/zjjbuqi/HiFiCCL
- name: minimap2
  role: Produces the map-hifi alignments whose inter-sequence comparison information
    defines the ALG edges
  repo: https://github.com/lh3/minimap2
- name: pysam
  role: Parses per-read split alignment records into 6-tuples used to label ALG nodes
  repo: https://github.com/pysam-developers/pysam
- name: hifiasm
  role: Downstream de novo assembler run on each label-corrected chromosomal read
    bin
  repo: https://github.com/chhylp123/hifiasm
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/advs.202515308
    title: Reference‐Guided Chromosome‐by‐Chromosome <i>de novo</i> Assembly at Scale
      Using Low‐Coverage High‐Fidelity Long‐Reads with HiFiCCL
  source_content_hash: sha256:102a914e4f2a2899add5170b80568ef97b890f08c038bceafc68b548d6cf964f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zjjbuqi/HiFiCCL
  tools:
  - CuteSV
  - HiFiCCL
  - hifiasm
  - minimap2
  - pysam
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# alignment-label-graph-label-propagation

## Summary

De novo chromosome class correction in HiFiCCL: a bidirectional Alignment Label Graph (ALG) is built from inter-sequence comparison of minimap2-aligned low-coverage HiFi reads, contained shorter nodes are collapsed, and a Z-drop-inspired heuristic traversal propagates chromosomal classes by majority vote with a minimum support threshold.

## When to use

Use when reference-based clustering of low-coverage (~5x) PacBio HiFi reads has left reads with missing (NA) or misassigned chromosome classes and overlap-based label propagation is needed to correct them before per-chromosome de novo assembly.

## When NOT to use

- Input reads are noisy long reads (ONT or PacBio CLR) rather than high-fidelity HiFi reads — the framework is designed for HiFi at ~5x coverage
- Hi-C or nanopore data integration is required — the article states this is left to future efforts and not yet implemented
- Reads already carry consistent, correct 'chr' chromosome classes and the assembly is run at sufficient coverage, so overlap-graph class correction adds no value

## Inputs

- PacBio HiFi reads (FASTQ/FASTA) at low coverage (~5x)
- minimap2 '-ax map-hifi' alignment records (SAM/BAM) of reads against the linear reference genome (e.g., GRCh38, CHM13)
- Per-read split alignment tuples (Read_s, Read_e, Ref_s, Ref_e, Chr, Strand) parsed with pysam/HiFiCCL logic
- Initial per-read reference-derived chromosome classes (including NA/ambiguous labels)

## Outputs

- Corrected per-read chromosome classes ('chr'-prefixed labels) for reads in the ALG paths
- Chromosomal bins of reads based on their final assigned classes
- Class-corrected read sets consumed by per-chromosome de novo assembly (hifiasm)

## How to apply

Align HiFi reads to the linear reference (e.g., GRCh38 or CHM13) with minimap2 using the map-hifi preset, then build a bidirectional Alignment Label Graph from the inter-sequence comparison information; if two sequences exhibit a containment relationship, collapse the node of the shorter sequence into the node of the longer sequence. Run the heuristic Z-drop-inspired search over the ALG to correct sequence classes: traverse paths and update each node's chromosomal class by majority vote among the nodes in the path, applying a minimum support threshold so weakly supported votes are rejected. For reads that remain NA/unresolved, perform bidirectional DFS within the constructed Pangenome Label Graph until chromosomal classes starting with 'chr' are encountered, harmonizing the classes. The rationale is that de novo assembly for each chromosomal cluster depends on correct overlap-derived labels, so misclassified reads would otherwise seed wrong chromosome bins. The corrected per-read classes are written out and reads are clustered into chromosomal bins for chromosome-by-chromosome hifiasm assembly.

## Related tools

- **HiFiCCL** (Implements the ALG construction, containment collapsing, Z-drop-inspired heuristic search and majority-vote label propagation) — https://github.com/zjjbuqi/HiFiCCL
- **minimap2** (Produces the map-hifi alignments whose inter-sequence comparison information defines the ALG edges) — https://github.com/lh3/minimap2
- **pysam** (Parses per-read split alignment records into 6-tuples used to label ALG nodes) — https://github.com/pysam-developers/pysam
- **hifiasm** (Downstream de novo assembler run on each label-corrected chromosomal read bin) — https://github.com/chhylp123/hifiasm

## Examples

```
./minimap2 -ax map-hifi ref.fa pacbio-ccs.fq.gz > aln.sam
```

## Evaluation signals

- The class-corrected output contains one row per read with read-ID and 'chr'-prefixed chromosome-class fields consistent with reference-derived labels
- Containment relationships in the ALG are resolved: every shorter contained sequence node is collapsed into its longer sequence's node
- Majority-vote class updates only take effect for paths meeting the minimum support threshold; low-support paths leave classes unchanged
- After bidirectional DFS in the PLG, no read should retain an NA/unresolved class if a 'chr'-prefixed class is reachable in the graph
- Downstream validation: combining corrected bins with hifiasm reduces misassembled contig length relative to hifiasm alone (average 21.19%, up to 38.58% on 45 human datasets at ~5x)

## Limitations

- At ~5x coverage many genomic regions remain missing, so the ALG may lack support for reads in unsequenced gaps and completeness gaps are unresolved
- Reference-guided correction can propagate reference-consistent but biologically wrong labels; the translocation correction module is needed for breakpoint-spanning reads that reference-based clustering would misassign
- No version numbers are stated for HiFiCCL or hifiasm in the source, so reproducibility of the graph-construction behavior across versions is uncertain

## Evidence

- [methods] Based on the inter‐sequence comparison information, a bidirectional graph is constructed: "Based on the inter‐sequence comparison information, a bidirectional graph is constructed"
- [methods] Z-drop heuristic ALG search: "Inspired by the Z‐drop algorithm, we propose a heuristic algorithm to search the ALG and correct sequence classes"
- [methods] Containment collapsing rule: "if two sequences exhibit a containment relationship, the node representing the shorter sequence is collapsed into the node of the longer sequence"
- [methods] Majority-vote propagation rule: "the chromosomal class is updated based on the majority vote among the nodes in the path"
- [intro] Overlap-based per-chromosome assembly rationale: "de novo assembly is performed for reads within each chromosomal cluster based on their overlap relationships"
- [methods] PLG bidirectional DFS to chr classes: [evidence span withheld — anchor status: not_found]
- [readme] minimap2 map-hifi invocation (README): [evidence span withheld — anchor status: external_doc]
- [discussion] Integration of breakpoint reads via correction modules: "By leveraging the translocation correction module or the de novo correction module, HiFiCCL integrates reads containing translocation breakpoints"
