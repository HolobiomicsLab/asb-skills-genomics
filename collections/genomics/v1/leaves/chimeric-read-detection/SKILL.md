---
name: chimeric-read-detection
description: Use when running DBG2OLC overlap/layout on long erroneous PacBio or Nanopore
  reads with >10x coverage, where chimeric reads and spurious contig identifiers would
  corrupt the backbone layout.
when_to_use_negative:
- Dataset coverage is <=10x long-read coverage, where the README only suggests RemoveChimera
  1 for >10x coverage
- Input contigs come from short-read assemblies that used repeat-resolving heuristics
  such as gap closing or scaffolding, which the README warns introduces errors and
  gives poor final results
- The task is base-level consensus polishing (Sparc/BLASR step), not structural/chimeric
  read filtering
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: DBG2OLC
  role: Runs the overlap/layout step with RemoveChimera 1, aligning compressed reads
    in multiple sequence alignment to remove chimeric reads and false contig anchors
  repo: https://github.com/yechengxi/DBG2OLC
- name: SparseAssembler
  role: Supplies the accurate raw DBG contigs (Contigs.txt) that serve as the identifiers
    compressed into long reads and filtered for false anchors
  repo: https://github.com/yechengxi/SparseAssembler
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1038/srep31900
    title: 'DBG2OLC: Efficient Assembly of Large Genomes Using Long Erroneous Reads
      of the Third Generation Sequencing Technologies'
  source_content_hash: sha256:30e0eb1eea685334ac0721179c8e821604e3d34a8da5603be320d4eea9fb7478
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yechengxi/DBG2OLC
  tools:
  - DBG2OLC
  - SparseAssembler
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# chimeric-read-detection

## Summary

Chimera-cleaning step of DBG2OLC that detects chimeric long reads and false contig anchors via multiple sequence alignment of compressed reads during overlap-layout, controlled by the RemoveChimera flag and related thresholds.

## When to use

Use when running DBG2OLC overlap/layout on long erroneous PacBio or Nanopore reads with >10x coverage, where chimeric reads and spurious contig identifiers would corrupt the backbone layout.

## When NOT to use

- Dataset coverage is <=10x long-read coverage, where the README only suggests RemoveChimera 1 for >10x coverage
- Input contigs come from short-read assemblies that used repeat-resolving heuristics such as gap closing or scaffolding, which the README warns introduces errors and gives poor final results
- The task is base-level consensus polishing (Sparc/BLASR step), not structural/chimeric read filtering

## Inputs

- SparseAssembler DBG contigs in FASTA format (Contigs.txt)
- Long erroneous 3GS reads in FASTA format (e.g. Pacbio_20x.fasta)
- Compressed long reads (contig-identifier sequences) and candidate overlap scores from the DBG2OLC overlap step

## Outputs

- Cleaned compressed-read set with chimeric reads and false contig anchors removed
- backbone_raw.fasta layout produced from chimera-filtered reads
- DBG2OLC_Consensus_info.txt used by the downstream Sparc/pbdagcon consensus step

## How to apply

Run DBG2OLC on the SparseAssembler Contigs.txt and the long-read FASTA with RemoveChimera 1, using the compressed-read representation (k 17) in which each long read is a sequence of contig identifiers. Candidate overlaps are first scored with AdaptiveTh (e.g. 0.0001-0.01) and KmerCovTh (e.g. 2) anchoring thresholds and a MinOverlap cutoff (e.g. 20 for 20x PacBio data); then each compressed read is aligned against its candidate reads in multiple sequence alignment to flag chimeric reads and false contig anchors, which are removed before layout. At ~100x coverage, raise the auxiliary parameters ChimeraTh and ContigTh from their default of 1 to 2 so more stringent conditions are applied. Validate by checking the N50 of backbone_raw.fasta before proceeding to consensus, retuning parameters if unsatisfied.

## Related tools

- **DBG2OLC** (Runs the overlap/layout step with RemoveChimera 1, aligning compressed reads in multiple sequence alignment to remove chimeric reads and false contig anchors) — https://github.com/yechengxi/DBG2OLC
- **SparseAssembler** (Supplies the accurate raw DBG contigs (Contigs.txt) that serve as the identifiers compressed into long reads and filtered for false anchors) — https://github.com/yechengxi/SparseAssembler

## Examples

```
./DBG2OLC k 17 AdaptiveTh 0.0001 KmerCovTh 2 MinOverlap 20 RemoveChimera 1 Contigs Contigs.txt f ../Pacbio_data/Pacbio_20x.fasta
```

## Evaluation signals

- RemoveChimera is set to 1 (2 at ~100x coverage) in the DBG2OLC command line
- Every read flagged as chimeric is absent from the cleaned compressed-read set, and every rejected contig anchor is absent from subsequent alignments
- All retained read pairs satisfy the MinOverlap score threshold used (10-30 for 10x/20x PacBio data, 50-150 for 50x-100x data)
- All anchored contigs pass both M >= AdaptiveTh * Contig_Length and M >= KmerCovTh thresholds
- N50 of the produced backbone_raw.fasta (e.g. 583 kbp in the S.cer w303 20x PacBio test) is checked and acceptable before proceeding to consensus

## Limitations

- Chimera removal is only suggested when coverage exceeds 10x; at low coverage the filter may not be recommended
- Default ChimeraTh and ContigTh values (1) are insufficient at ~100x coverage, where they must be raised to 2 for stringent filtering
- As with parameter tuning generally, the first run may not give the best result; RemoveChimera and the matching thresholds may need to be fine-tuned together, and combining DBG2OLC with other tools may introduce errors

## Evidence

- [readme] RemoveChimera: remove chimeric reads in the dataset, suggest 1 if you have >10x coverage.: "RemoveChimera: remove chimeric reads in the dataset, suggest 1 if you have >10x coverage."
- [readme] ChimeraTh/ContigTh used in multiple alignment: "These two are used in multiple alignment to remove problematic reads and false contig anchors."
- [abstract] Structural errors must be detected and corrected: "Base-level errors can be skipped; structural errors need to be detected and corrected."
