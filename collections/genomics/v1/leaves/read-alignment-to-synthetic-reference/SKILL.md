---
name: read-alignment-to-synthetic-reference
description: Use when basecalled nanopore dRNA reads must be mapped onto synthetic
  RNA reference sequences (random-ligation or splint-ligation oligo constructs) before
  per-read m6A prediction, and mapq-filtered BAMs are required.
when_to_use_negative:
- Input is already a sorted, indexed, quality-filtered BAM from a prior alignment
  run
- Reads are raw fast5 signal files that still need RODAN basecalling before any alignment
  is possible
- The task is ensemble-level quantification against a transcriptome where ambiguous
  multi-mapping is acceptable rather than to be excluded
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3308
tools:
- name: RODAN
  role: Fully convolutional basecaller whose basecall.py produces the rodan.fasta
    read sequences that are the alignment input
  repo: https://github.com/biodlab/RODAN
- name: minimap2.22
  role: Splice-aware, primary-only aligner (-ax splice, --secondary=no, -k14) used
    to map basecalled reads to the synthetic reference
- name: samtools
  role: Filters aligned reads to mapping quality 50 (samtools view -bST -q50), then
    sorts and indexes the BAM
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-47661-2
    title: Detecting m6A at single-molecular resolution via direct RNA sequencing
      and realistic training data
  source_content_hash: sha256:bf4b0e3ad79f70f426d3de41d05f4afebd2d0629673c3ef39ddebc37b77d0a45
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biodlab/RODAN
  tools:
  - CHEUI
  - RODAN
  - m6Anet
  - mAFiA
  - minimap2.22
  - samtools
  - sklearn
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# read-alignment-to-synthetic-reference

## Summary

Align RODAN-basecalled direct RNA reads to synthetic oligo reference sequences (e.g., the TEST1 heteropolymer references containing all possible random combinations of constituent oligos) with minimap2.22 under splice-aware, primary-only settings, then filter to high mapping quality with samtools so downstream m6A detection (mAFiA) operates on confidently placed reads.

## When to use

Use when basecalled nanopore dRNA reads must be mapped onto synthetic RNA reference sequences (random-ligation or splint-ligation oligo constructs) before per-read m6A prediction, and mapq-filtered BAMs are required.

## When NOT to use

- Input is already a sorted, indexed, quality-filtered BAM from a prior alignment run
- Reads are raw fast5 signal files that still need RODAN basecalling before any alignment is possible
- The task is ensemble-level quantification against a transcriptome where ambiguous multi-mapping is acceptable rather than to be excluded

## Inputs

- Basecalled reads as FASTA (rodan.fasta from RODAN basecall.py)
- Synthetic reference sequences FASTA (e.g., TEST1 references containing all possible random combinations of constituent oligos, or GRCh38_96.X.fa)
- Reference index files (.fai) for samtools view -bST

## Outputs

- Sorted, mapq>=50-filtered, indexed BAM (e.g., minimap.q50.bam)
- Optionally a minimap2 --cs tabular/CS alignment file for downstream inspection

## How to apply

After RODAN basecalling produces a FASTA (rodan.fasta), align it against the synthetic reference FASTA with minimap2.22 using splice-aware, primary-only settings: --secondary=no -ax splice -uf -k14 -t 36 --cs. Pipe the output through samtools view -bST ${ref} -q50 to drop reads with mapping quality below 50, then samtools sort and samtools index to produce a sorted, indexed BAM. The rationale is that the synthetic references contain all possible random combinations of constituent oligos, so strict primary-only mapping and a q50 cutoff prevent ambiguous placements from contaminating read-level modification probabilities. For discontinuous alignments, the section of the read with the longest continuous mapping is the one used downstream.

## Related tools

- **RODAN** (Fully convolutional basecaller whose basecall.py produces the rodan.fasta read sequences that are the alignment input) — https://github.com/biodlab/RODAN
- **minimap2.22** (Splice-aware, primary-only aligner (-ax splice, --secondary=no, -k14) used to map basecalled reads to the synthetic reference)
- **samtools** (Filters aligned reads to mapping quality 50 (samtools view -bST -q50), then sorts and indexes the BAM)

## Examples

```
minimap2 --secondary=no -ax splice -uf -k14 -t 36 --cs ${ref} ${basecall} | samtools view -bST ${ref} -q50 - | samtools sort - > ${bam}; samtools index ${bam}
```

## Evaluation signals

- Output BAM contains only primary alignments (run with --secondary=no; no secondary/supplementary ambiguity)
- All retained records have MAPQ >= 50 after the samtools view -q50 filter
- BAM is sorted and an .bai index exists (samtools sort + samtools index complete without error)
- Read names in the BAM match the basecalled rodan.fasta headers, confirming the correct FASTA was aligned
- Downstream mAFiA run (--min_coverage 50) finds sufficient coverage per site, implying the q50-filtered BAM still retains usable depth

## Limitations

- Reads with mapq below 50 are discarded, so highly ambiguous reads against the oligo-combinatorial references reduce effective coverage
- In case of discontinuous alignment to the reference, only the section of the read with the longest continuous mapping is chosen, discarding the rest of the read
- Splice settings (-ax splice -uf -k14) are tuned for nanopore direct RNA; DNA or short-read inputs would need different minimap2 presets
- Results depend on the reference containing all possible random combinations of constituent oligos; an incomplete reference will misplace reads

## Evidence

- [results] basecalled reads are aligned to reference sequences containing all possible random combinations of constituent oligos using minimap2.22: "basecalled reads are aligned to reference sequences containing all possible random combinations of constituent oligos using minimap2.22"
- [results] Reads with mapq below 50 are filtered out.: "Reads with mapq below 50 are filtered out."
- [readme] minimap2 --secondary=no -ax splice -uf -k14 -t 36 --cs ${ref} ${basecall}: "minimap2 --secondary=no -ax splice -uf -k14 -t 36 --cs ${ref} ${basecall} | samtools view -bST ${ref} -q50 -"
- [results] In case of discontinuous alignment to the reference, the section of the read with the longest continuous mapping is chosen.: "In case of discontinuous alignment to the reference, the section of the read with the longest continuous mapping is chosen."
