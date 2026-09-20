---
name: multiple-sequence-alignment-consensus-generation
description: Use when self-correcting long reads (PacBio or ONT) by deriving per-window
  consensus from overlapping read piles, or when ultra-long reads (up to 1.5 Mbp)
  break alignment-based correctors.
when_to_use_negative:
- The task is assembly polishing rather than read self-correction — use CONSENT-polish
  (--contigs/--reads) instead of CONSENT-correct
- 'When correcting repeated regions that grow very long: the strategy of selecting
  only the N highest-identity overlaps per alignment pile may be inefficient in that
  regime'
- 'When disk space is limited: the overlaps file alone requires ~5 TB of external
  storage for a full human ONT dataset'
edam_operation: http://edamontology.org/operation_3184
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0769
tools:
- name: CONSENT
  role: 'Primary tool: computes overlaps, segments alignment piles into windows, builds
    window MSAs, and polishes consensus with local de Bruijn graphs via CONSENT-correct'
  repo: https://github.com/morispi/CONSENT
- name: minimap2
  role: Required dependency (must be on PATH) used to compute overlaps between long
    reads before MSA/consensus generation; index split every 500M bases by default
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1038/s41598-020-80757-5
    title: Scalable long read self-correction and assembly polishing with multiple
      sequence alignment
  source_content_hash: sha256:78bf0bfc887d0d8f00d58a5418f20cc35ec441470c27ded10e53ff8208bd81d0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/morispi/CONSENT
  tools:
  - CONSENT
  - Canu
  - Daccord
  - ELECTOR
  - FLAS
  - MECAT
  - ONT
  - minimap2
  edam_operation: http://edamontology.org/operation_3184
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 22
---

# multiple-sequence-alignment-consensus-generation

## Summary

Generate corrected long-read consensus sequences by computing multiple sequence alignments (MSA) over windows of overlapping PacBio/ONT reads and polishing each window consensus with a local de Bruijn graph, as implemented in CONSENT.

## When to use

Use when self-correcting long reads (PacBio or ONT) by deriving per-window consensus from overlapping read piles, or when ultra-long reads (up to 1.5 Mbp) break alignment-based correctors.

## When NOT to use

- The task is assembly polishing rather than read self-correction — use CONSENT-polish (--contigs/--reads) instead of CONSENT-correct
- When correcting repeated regions that grow very long: the strategy of selecting only the N highest-identity overlaps per alignment pile may be inefficient in that regime
- When disk space is limited: the overlaps file alone requires ~5 TB of external storage for a full human ONT dataset

## Inputs

- FASTA or FASTQ file of long reads (PacBio --type PB or ONT --type ONT)
- Optionally a reference genome (e.g. E. coli, S. cerevisiae, C. elegans, human) for evaluation
- Reference FASTA of the target genome if polishing assemblies (contigs FASTA + long reads)

## Outputs

- FASTA file of corrected long reads (consensus sequences per read)
- Per-window MSA-derived consensus sequences polished with local de Bruijn graphs
- Temporary overlaps file in PAF format (Alignments_dateTimeStamp.paf, ~5 TB for a full human dataset)

## How to apply

First compute overlaps between the long reads with minimap2, select up to maxSupport (default 150) highest-identity overlaps per read to form an alignment pile, and split each pile into windows of windowSize 500 (windowOverlap 50). For each window, use a k-mer chaining strategy (merSize 9, anchorSupport 8, minAnchors 2) to build an MSA of the supporting reads and derive a consensus sequence, keeping windows with minSupport 4. Polish each window consensus with a local de Bruijn graph (solid k-mer threshold 4) to remove remaining errors, then locally realign the polished consensus back to the read to output the corrected sequence. Evaluate the corrected reads against a reference (e.g. with ELECTOR for simulated PacBio data, minimap2 for ONT human data) to confirm error rates and identity in expected ranges.

## Related tools

- **CONSENT** (Primary tool: computes overlaps, segments alignment piles into windows, builds window MSAs, and polishes consensus with local de Bruijn graphs via CONSENT-correct) — https://github.com/morispi/CONSENT
- **minimap2** (Required dependency (must be on PATH) used to compute overlaps between long reads before MSA/consensus generation; index split every 500M bases by default)

## Examples

```
./CONSENT-correct --in longReads.fastq --out correctedReads.fasta --type ONT --tmpdir /external/storage --nproc 16
```

## Evaluation signals

- On the simulated PacBio example (10x, 4 cores), the run should take ~2 min and use up to ~750 MB RAM
- Reproduced post-correction error rates should match published Table 1 values: ~0.3111% (E. coli 30x), ~0.1833% (E. coli 60x), ~0.3858% (C. elegans 60x), with recall/precision in reported ranges
- On the full human ONT dataset, expect ~10 days runtime, <100 GB RAM, corrected-read N50 ~12,880 bp, ~98.4% aligned fraction, and ~93.86% alignment identity via minimap2
- Corrected output must be a valid FASTA in which each corrected read is locally realigned from the polished window consensus
- Default parameters should produce the published results: windowSize 500, minSupport 4, maxSupport 150, merSize 9, solid 4, anchorSupport 8, minAnchors 2

## Limitations

- CONSENT's correction step runtime tends to be higher than other state-of-the-art methods (2–38x speed advantage applies to polishing, not correction)
- Selecting only the N highest-identity overlaps may prove inefficient when repeated regions grow longer
- Window size and minimum number of anchors significantly impact runtime; an optimal runtime/quality compromise has not been established
- The overlaps file for a full human dataset requires ~5 TB of disk, so --tmpdir must point to external storage
- Daccord's alignment-based overlapping (an alternative approach) consumed 3x-11x more memory than CONSENT or MECAT and failed to scale to C. elegans; CONSENT is the only method able to efficiently scale to ONT ultra-long reads

## Evidence

- [discussion] MSA/consensus per window: "It then divides the overlapping regions into smaller windows, in order to compute MSA, and consensus sequences of each window independently."
- [intro] local de Bruijn graph polishing: "These consensus sequences are then polished with local de Bruijn graphs, in order to correct remaining errors, and further reduce the final error rate."
- [methods] k-mer chaining parameter defaults: "--anchorSupport INT, -c INT:   Minimum number of sequences supporting (Ai) - (Ai+1) to keep the two anchors in the chaining. (default: 8)"
- [readme] README correction invocation: "./CONSENT-correct --in example/reads.fasta --out example/correctedReads.fasta --type PB"
- [discussion] realign polished consensus: "Finally, the polished consensus is locally realigned to the read, in order to correct it."
