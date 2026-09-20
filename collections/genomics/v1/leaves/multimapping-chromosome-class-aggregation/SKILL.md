---
name: multimapping-chromosome-class-aggregation
description: Use when assigning per-read chromosome classes to low-coverage (~5x)
  HiFi reads via reference alignment, where multi-mapping reads must aggregate every
  hit chromosome and unmapped reads must be retained as 'None' rather than discarded.
when_to_use_negative:
- Reads are high-coverage enough for fully de novo assembly without reference guidance,
  or no reference genome exists for the organism
- Input is short or noisy long reads — minimap2's map-hifi preset targets PacBio HiFi/CCS
  reads, so short-read or ONT data would need a different preset/skill
- The goal is de novo class correction of already-classified reads (the ALG-based
  heuristic or translocation correction steps), not initial reference-derived class
  assignment
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0196
tools:
- name: minimap2
  role: Aligns all HiFi reads to the linear reference genome with the '-ax map-hifi'
    preset to derive per-read chromosome classes and multi-mapping class aggregation
  repo: https://github.com/lh3/minimap2
- name: minigraph
  role: 'Optional: realigns reads that failed linear-reference alignment to the pangenome
    graph using ''-cx lr'' for DFS-based class resolution'
  repo: https://github.com/lh3/minigraph
- name: HiFiCCL
  role: Framework implementing the aggregation of multi-mapping chromosome classes,
    'None' labeling, PLG construction, and bidirectional DFS standardization of NA/HG
    classes
  repo: https://github.com/zjjbuqi/HiFiCCL
- name: pysam
  role: Python library used to parse the SAM alignments and attach chromosome classes
    per read
  repo: https://github.com/pysam-developers/pysam
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/advs.202515308
    title: Reference‐Guided Chromosome‐by‐Chromosome <i>de novo</i> Assembly at Scale
      Using Low‐Coverage High‐Fidelity Long‐Reads with HiFiCCL
  source_content_hash: sha256:48cac1c89f499558e0484a625e9349c7140bdced3dc9a410867f57d8afbad8cc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - HiFiCCL
  - minigraph
  - minimap2
  - pysam
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# multimapping-chromosome-class-aggregation

## Summary

HiFiCCL's reference-guided chromosome classification stage: every low-coverage PacBio HiFi read is aligned to a linear reference (e.g., CHM13 v2.0) with minimap2 '-ax map-hifi', and each read's class is set by aggregating all chromosome classes when it maps to multiple chromosomes, with unaligned reads labeled 'None' (optionally resolved later via pangenome-graph DFS).

## When to use

Use when assigning per-read chromosome classes to low-coverage (~5x) HiFi reads via reference alignment, where multi-mapping reads must aggregate every hit chromosome and unmapped reads must be retained as 'None' rather than discarded.

## When NOT to use

- Reads are high-coverage enough for fully de novo assembly without reference guidance, or no reference genome exists for the organism
- Input is short or noisy long reads — minimap2's map-hifi preset targets PacBio HiFi/CCS reads, so short-read or ONT data would need a different preset/skill
- The goal is de novo class correction of already-classified reads (the ALG-based heuristic or translocation correction steps), not initial reference-derived class assignment

## Inputs

- PacBio HiFi/CCS reads in FASTQ/FASTA (low-coverage, e.g., ~5x)
- Linear reference genome (e.g., CHM13 v2.0) in FASTA
- Optional: pangenome graph in GFA for the pangenome-guided mode

## Outputs

- Per-read chromosome classes (multi-mapping reads carry aggregated classes; unaligned reads labeled 'None')
- *prefix*.map_to_reference.sam — per-read alignment to the linear reference
- *prefix*.gaf — pangenome alignment (when pangenome-guided mode is enabled)

## How to apply

Align all HiFi reads to the linear reference genome (e.g., CHM13 v2.0) with minimap2 using the '-ax map-hifi' preset, parsing the resulting SAM (e.g., with pysam 0.21.0) to record each read's chromosome class. When a read maps to multiple chromosomes, aggregate every chromosome class into the read's label instead of keeping a single arbitrary hit; label reads with no alignment as 'None' so they can be recovered downstream rather than silently dropped. Optionally, realign reads that failed linear-reference alignment to a pangenome graph with minigraph using '-cx lr', build the Pangenome Label Graph (PLG) from the graph paths with nodes tagged by pangenome node IDs, and run bidirectional DFS for reads whose classes begin with 'NA' or 'HG' until 'chr'-prefixed classes are reached. Write the per-read reference alignment to *prefix*.map_to_reference.sam and the pangenome alignment to *prefix*.gaf. These initial classes then feed chromosome binning and any subsequent de novo class correction; this stage applies only the reference-derived (or pangenome-resolved) labels.

## Related tools

- **minimap2** (Aligns all HiFi reads to the linear reference genome with the '-ax map-hifi' preset to derive per-read chromosome classes and multi-mapping class aggregation) — https://github.com/lh3/minimap2
- **minigraph** (Optional: realigns reads that failed linear-reference alignment to the pangenome graph using '-cx lr' for DFS-based class resolution) — https://github.com/lh3/minigraph
- **HiFiCCL** (Framework implementing the aggregation of multi-mapping chromosome classes, 'None' labeling, PLG construction, and bidirectional DFS standardization of NA/HG classes) — https://github.com/zjjbuqi/HiFiCCL
- **pysam** (Python library used to parse the SAM alignments and attach chromosome classes per read) — https://github.com/pysam-developers/pysam

## Examples

```
minimap2 -ax map-hifi CHM13v2.0.fa hifi_reads.fq.gz > prefix.map_to_reference.sam
```

## Evaluation signals

- Each read that has supporting reference or pangenome alignment exits the classification stage with a non-'None', 'chr'-prefixed chromosome label
- Multi-mapping reads carry every chromosome class they map to (aggregated), not a single arbitrarily chosen hit
- Output files *prefix*.map_to_reference.sam and (optionally) *prefix*.gaf exist and contain per-read alignment records
- Read-level chromosome labels match the read-level ground-truth chromosome classes
- Reads with 'NA'/'HG'-prefixed classes are standardized to 'chr'-prefixed classes after the PLG bidirectional DFS

## Limitations

- Reads with no linear-reference alignment are labeled 'None' and depend on the optional pangenome-guided mode (minigraph) to be resolved; without it they remain unclassified
- At ~5x coverage many genomic regions remain missing, so some reads may lack sufficient alignment information for classification
- Classes beginning with 'NA' or 'HG' (e.g., unplaced/alt reference contigs) require PLG bidirectional DFS standardization before binning

## Evidence

- [methods] Align all HiFi reads to the linear reference genome (e.g. CHM13 v2.0) with minimap2 using the '-ax map-hifi' preset and record the chromosome class of each read, aggregating every chromosome class when a read maps to multiple chromosomes and labeling reads with no alignment as 'None'.: [evidence span withheld — anchor status: not_found]
- [methods] For reads carrying classes that begin with 'NA' or 'HG', run bidirectional depth-first search within the PLG until chromosome classes starting with 'chr' are reached, and standardize those reads' chromosome classes.: [evidence span withheld — anchor status: not_found]
- [methods] Write the per-read reference alignment to *prefix*.map_to_reference.sam and the pangenome alignment to *prefix*.gaf.: [evidence span withheld — anchor status: not_found]
- [results] Reads that cannot be classified from the linear reference are resolved via the pangenome graph: starting from the read's alignment position in the graph, a bidirectional DFS is run until a position on the linear reference genome is reached, which determines the read's chromosome class.: [evidence span withheld — anchor status: not_found]
- [readme] ./minimap2 -ax map-hifi ref.fa pacbio-ccs.fq.gz > aln.sam # PacBio HiFi/CCS genomic reads (v2.19+): [evidence span withheld — anchor status: external_doc]
- [readme] # Map sequence to graph
./minigraph test/MT.gfa test/MT-orangA.fa > out.gaf: [evidence span withheld — anchor status: external_doc]
