---
name: bidirectional-dfs-chromosome-class-resolution
description: Use when HiFi reads exit linear reference alignment (e.g., minimap2 map-hifi
  vs CHM13) with 'None' or non-'chr' classes such as 'NA'/'HG', and a pangenome graph
  alignment (GAF) is available to assign chromosome classes via graph traversal.
when_to_use_negative:
- Reads already carry a confident 'chr'-prefixed chromosome class from linear reference
  alignment — DFS resolution is only for 'NA'/'HG'/unclassified reads
- No pangenome graph or GAF alignment is available for the unaligned reads
- The task is de novo class correction after initial classification (the Z-drop-inspired
  ALG heuristic), not initial reference-guided class assignment
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: minigraph
  role: Realigns reads that failed linear-reference alignment to the pangenome graph
    with '-cx lr', producing the GAF alignments that anchor the DFS starting positions
    in the PLG
  repo: https://github.com/lh3/minigraph
- name: HiFiCCL
  role: Framework that builds the Pangenome Label Graph from pangenome graph paths
    and runs the bidirectional DFS to standardize 'NA'/'HG' classes
  repo: https://github.com/zjjbuqi/HiFiCCL
- name: pysam
  role: Python library (v0.21.0) used to parse per-read alignment records when aggregating
    chromosome classes
  repo: https://github.com/pysam-developers/pysam
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/advs.202515308
    title: Reference‐Guided Chromosome‐by‐Chromosome <i>de novo</i> Assembly at Scale
      Using Low‐Coverage High‐Fidelity Long‐Reads with HiFiCCL
  source_content_hash: sha256:9796c0f6e0b85ff377f4950b897d23238bf5f964b34502cefb3b0178ccf0b349
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minigraph
  tools:
  - HiFiCCL
  - minigraph
  - minimap2
  - pysam
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# bidirectional-dfs-chromosome-class-resolution

## Summary

Resolves chromosome classes of HiFi reads that fail linear-reference alignment by running a bidirectional depth-first search from the read's alignment position in a Pangenome Label Graph (PLG) until 'chr'-prefixed classes are reached, standardizing classes that begin with 'NA' or 'HG'.

## When to use

Use when HiFi reads exit linear reference alignment (e.g., minimap2 map-hifi vs CHM13) with 'None' or non-'chr' classes such as 'NA'/'HG', and a pangenome graph alignment (GAF) is available to assign chromosome classes via graph traversal.

## When NOT to use

- Reads already carry a confident 'chr'-prefixed chromosome class from linear reference alignment — DFS resolution is only for 'NA'/'HG'/unclassified reads
- No pangenome graph or GAF alignment is available for the unaligned reads
- The task is de novo class correction after initial classification (the Z-drop-inspired ALG heuristic), not initial reference-guided class assignment

## Inputs

- HiFi reads (FASTQ/FASTA) that failed linear-reference alignment or carry 'NA'/'HG' chromosome classes
- Pangenome graph in GFA format (from minigraph)
- Per-read alignments to the pangenome graph in GAF format (*prefix*.gaf)
- Linear reference genome (e.g., CHM13 v2.0) alignments in SAM (*prefix*.map_to_reference.sam)

## Outputs

- Standardized per-read chromosome classes with 'chr'-prefixed labels replacing 'NA'/'HG'/'None' classes where the PLG traversal succeeds
- Pangenome Label Graph (PLG) with nodes tagged by pangenome node IDs and bidirectional path edges

## How to apply

First realign reads that failed linear-reference alignment to the pangenome graph using minigraph with the '-cx lr' parameter. Build the Pangenome Label Graph (PLG) from the pangenome graph paths, tagging nodes with pangenome node IDs and establishing bidirectional edges along the paths. For reads whose classes begin with 'NA' or 'HG', start a bidirectional depth-first search from the read's alignment position in the PLG and traverse until chromosome classes starting with 'chr' are encountered; that class determines and standardizes the read's chromosome label. This complements linear-reference classification so that every read exits the classification stage with a non-'None' 'chr'-prefixed label wherever reference or pangenome information supports it, before any de novo class correction (ALG heuristic) is applied.

## Related tools

- **minigraph** (Realigns reads that failed linear-reference alignment to the pangenome graph with '-cx lr', producing the GAF alignments that anchor the DFS starting positions in the PLG) — https://github.com/lh3/minigraph
- **HiFiCCL** (Framework that builds the Pangenome Label Graph from pangenome graph paths and runs the bidirectional DFS to standardize 'NA'/'HG' classes) — https://github.com/zjjbuqi/HiFiCCL
- **pysam** (Python library (v0.21.0) used to parse per-read alignment records when aggregating chromosome classes) — https://github.com/pysam-developers/pysam

## Examples

```
minigraph -cx lr pangenome.gfa unaligned_reads.fq > prefix.gaf
```

## Evaluation signals

- Each read that had reference or pangenome alignment support exits classification with a non-'None', 'chr'-prefixed chromosome label
- Read-level standardized classes match the read-level ground-truth chromosome classes
- Every DFS traversal for an 'NA'/'HG'-classed read terminates at a node/position whose class starts with 'chr'
- The per-read pangenome alignment file (*prefix*.gaf) and reference alignment file (*prefix*.map_to_reference.sam) are both populated for the expected read sets
- PLG nodes carry valid pangenome node IDs and bidirectional edges consistent with the pangenome graph paths

## Limitations

- Resolution depends on the DFS reaching a 'chr'-prefixed class from the read's graph position; reads whose neighborhood in the PLG never reaches a linear-reference-anchored position remain unresolved
- This is an initial-classification aid only — it does not perform de novo or translocation class correction, which are separate HiFiCCL modules
- Pangenome-guided mode is optional; it only applies to reads that failed linear reference alignment or received non-'chr' classes

## Evidence

- [methods] These reads are processed through bidirectional DFS within the constructed PLG until chromosomal classes starting with "chr" are encountered: [evidence span withheld — anchor status: not_found]
- [methods] a PLG is constructed to harmonize the chromosomal classes: "a PLG is constructed to harmonize the chromosomal classes"
- [methods] Construction Of Pangenome Label Graph: "Construction Of Pangenome Label Graph"
- [readme] Map sequence to graph ./minigraph test/MT.gfa test/MT-orangA.fa > out.gaf: "Map sequence to graph
./minigraph test/MT.gfa test/MT-orangA.fa > out.gaf"
