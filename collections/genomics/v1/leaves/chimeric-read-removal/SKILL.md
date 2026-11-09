---
name: chimeric-read-removal
description: Use when running DBG2OLC overlap/layout on raw long (PacBio/3GS) reads
  for hybrid assembly and the long-read set may contain chimeric molecules, especially
  with >10x long-read coverage.
when_to_use_negative:
- Long-read coverage is at or below 10x — the README's suggestion of RemoveChimera
  1 is explicitly conditioned on >10x coverage
- Reads are already error-corrected or filtered by other tools; the README warns that
  combining DBG2OLC with existing assembly/error-correction tools can introduce errors
  and worsen results
- Running the non-hybrid NGS-only assembly mode, where long-read chimeras are not
  the input
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: DBG2OLC
  role: Performs the overlap and layout of compressed long reads and applies RemoveChimera
    (plus ChimeraTh/ContigTh at high coverage) internally to drop chimeric reads
  repo: https://github.com/yechengxi/DBG2OLC
- name: AssemblyUtility
  role: SelectLongestReads builds the 20x PacBio read subset (e.g. Pacbio_20x.fasta)
    that RemoveChimera then cleans during layout
  repo: https://github.com/yechengxi/AssemblyUtility
provenance:
  source_task_ids:
  - task_008
  - task_005
  source_papers:
  - doi: 10.1038/srep31900
    title: 'DBG2OLC: Efficient Assembly of Large Genomes Using Long Erroneous Reads
      of the Third Generation Sequencing Technologies'
  source_content_hash: sha256:86a0ffecb0319e5b71e783d99fa215465e4b713fcdf9f9efaf3d7fe27436dc3e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yechengxi/DBG2OLC
  tools:
  - AssemblyUtility
  - DBG2OLC
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# chimeric-read-removal

## Summary

Filters chimeric long reads during the DBG2OLC overlap-and-layout step via the RemoveChimera flag (and ChimeraTh/ContigTh at high coverage), so chimeric PacBio/3GS reads cannot be compressed and laid out into false backbone links.

## When to use

Use when running DBG2OLC overlap/layout on raw long (PacBio/3GS) reads for hybrid assembly and the long-read set may contain chimeric molecules, especially with >10x long-read coverage.

## When NOT to use

- Long-read coverage is at or below 10x — the README's suggestion of RemoveChimera 1 is explicitly conditioned on >10x coverage
- Reads are already error-corrected or filtered by other tools; the README warns that combining DBG2OLC with existing assembly/error-correction tools can introduce errors and worsen results
- Running the non-hybrid NGS-only assembly mode, where long-read chimeras are not the input

## Inputs

- Raw long reads in FASTA format (e.g. Pacbio_20x.fasta, 20x PacBio subset selected with SelectLongestReads)
- DBG contigs from SparseAssembler in FASTA format (Contigs.txt)
- DBG2OLC parameter set: k, AdaptiveTh, KmerCovTh, MinOverlap, RemoveChimera (and ChimeraTh/ContigTh at ~100x)

## Outputs

- Cleaned compressed reads (ordered contig-identifier lists, loadable with LD 1)
- Overlap-layout backbone backbone_raw.fasta
- DBG2OLC_Consensus_info.txt mapping file for the consensus step

## How to apply

Enable removal inside the DBG2OLC invocation by setting RemoveChimera 1 alongside the core parameters (k 17, AdaptiveTh, KmerCovTh, MinOverlap) rather than pre-filtering reads with a separate tool. The README suggests RemoveChimera 1 when coverage exceeds 10x, as in the S.cer w303 test run (20x PacBio, KmerCovTh 2, MinOverlap 20, AdaptiveTh 0.0001). For ~100x coverage, additionally raise ChimeraTh and ContigTh from their default of 1 to 2; these act in the multiple-alignment stage to drop problematic reads and false contig anchors. Because the tool itself decides which compressed reads are chimeric, the practical decision point is the N50 of the resulting backbone_raw.fasta — if it is unsatisfactory, retune KmerCovTh/MinOverlap/AdaptiveTh (e.g. MinOverlap 10-30 for 10x-20x PacBio) before proceeding to consensus.

## Related tools

- **DBG2OLC** (Performs the overlap and layout of compressed long reads and applies RemoveChimera (plus ChimeraTh/ContigTh at high coverage) internally to drop chimeric reads) — https://github.com/yechengxi/DBG2OLC
- **AssemblyUtility** (SelectLongestReads builds the 20x PacBio read subset (e.g. Pacbio_20x.fasta) that RemoveChimera then cleans during layout) — https://github.com/yechengxi/AssemblyUtility

## Examples

```
./DBG2OLC k 17 AdaptiveTh 0.0001 KmerCovTh 2 MinOverlap 20 RemoveChimera 1 Contigs Contigs.txt f ../Pacbio_data/Pacbio_20x.fasta
```

## Evaluation signals

- backbone_raw.fasta is produced with positive node/edge counts consistent with the cleaned-read set
- N50 of backbone_raw.fasta is acceptable — 583 kbp in the reference S.cer w303 run with 20x PacBio reads; if unsatisfactory, do not proceed to consensus
- The number of surviving compressed reads is consistent with the non-chimeric fraction of the input long-read FASTA (check via the ReadsInfoFrom_* output)
- A subsequent DBG2OLC run with LD 1 reloads the same compressed reads without regenerating them, confirming the filter stage completed
- Downstream consensus (split_and_run_sparc.sh on backbone_raw.fasta + DBG2OLC_Consensus_info.txt + ctg_pb.fasta) completes without unmatched backbone regions

## Limitations

- RemoveChimera 1 is only suggested when long-read coverage exceeds 10x; with sparse coverage the filter risks discarding true reads
- ChimeraTh/ContigTh adjustments (default 1, set to 2 at ~100x) are needed for high-coverage data, otherwise the default stringency may leave false contig anchors
- The manual instructs following the prescribed pipeline steps before improvising, since external assembly/error-correction tools can introduce errors that chimeric filtering does not compensate for

## Evidence

- [readme] RemoveChimera: remove chimeric reads in the dataset, suggest 1 if you have >10x coverage.: "RemoveChimera: remove chimeric reads in the dataset, suggest 1 if you have >10x coverage."
- [readme] ./DBG2OLC k 17 AdaptiveTh 0.0001 KmerCovTh 2 MinOverlap 20 RemoveChimera 1 Contigs Contigs.txt f ../Pacbio_data/Pacbio_20x.fasta: "./DBG2OLC k 17 AdaptiveTh 0.0001 KmerCovTh 2 MinOverlap 20 RemoveChimera 1 Contigs Contigs.txt f ../Pacbio_data/Pacbio_20x.fasta"
- [readme] ChimeraTh: default: 1, set to 2 if coverage is ~100x.: "ChimeraTh: default: 1, set to 2 if coverage is ~100x."
- [readme] When we have high coverage, some more stringent conditions shall be applied as with the suggested parameters.: "When we have high coverage, some more stringent conditions shall be applied as with the suggested parameters."
- [intro] It is important to follow the steps before you invent your own pipeline.: "It is important to follow the steps before you invent your own pipeline."
