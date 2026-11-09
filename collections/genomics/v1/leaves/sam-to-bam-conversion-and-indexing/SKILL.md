---
name: sam-to-bam-conversion-and-indexing
description: Use when long ONT/PacBio reads have been aligned with Minimap2 (e.g.
  -ax map-ont --secondary=no) and the downstream tool (AgileStructure) requires a
  sorted, indexed BAM input.
when_to_use_negative:
- The alignment is still in PAF format from minimap2 without CIGAR/SAM output — convert
  or re-run with -a first
- The downstream tool accepts SAM or CRAM directly, making sorting/indexing redundant
  overhead
- Short-read data is being processed for SNV/small-indel calling, where this long-read
  SV workflow does not apply
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0091
tools:
- name: minimap2
  role: Generates the SAM alignments from ONT/PacBio long reads against hg19, configured
    with -ax map-ont and --secondary=no so primary alignments feed SV detection
  repo: https://github.com/lh3/minimap2
- name: AgileStructure
  role: Consumes the sorted, indexed BAM to display split-read alignments and annotate
    breakpoints of large-scale structural rearrangements
  repo: https://github.com/msjimc/AgileStructure
- name: Samtools
  role: Converts SAM to BAM, sorts by genomic position, and indexes the BAM for AgileStructure
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btag294
    title: Utilization of long-read sequencing for the detection of structural rearrangements
      with AgileStructure
  source_content_hash: sha256:9bd37dcaaec96436ead4d180ce17f8a9a349ff59fc5fa821892b46e1311f44b7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - AgileStructure
  - Samtools
  - minimap2
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# SAM to BAM conversion and indexing

## Summary

Converts Minimap2 long-read SAM alignments to coordinate-sorted, indexed BAM so that desktop SV callers such as AgileStructure can load and browse breakpoint-spanning reads for structural rearrangement annotation.

## When to use

Use when long ONT/PacBio reads have been aligned with Minimap2 (e.g. -ax map-ont --secondary=no) and the downstream tool (AgileStructure) requires a sorted, indexed BAM input.

## When NOT to use

- The alignment is still in PAF format from minimap2 without CIGAR/SAM output — convert or re-run with -a first
- The downstream tool accepts SAM or CRAM directly, making sorting/indexing redundant overhead
- Short-read data is being processed for SNV/small-indel calling, where this long-read SV workflow does not apply

## Inputs

- SAM file of long-read alignments produced by Minimap2 (e.g. with -ax map-ont --secondary=no)
- Optionally the raw long-read FASTQ/FASTA and reference genome FASTA (hg19) upstream of the SAM

## Outputs

- Coordinate-sorted BAM file of primary long-read alignments
- BAM index (.bai/.csi) enabling random access by AgileStructure
- Downstream: translocation annotation (e.g. translocation_annotation.txt) with breakpoint coordinates and supporting read information

## How to apply

After aligning long reads (e.g. SRA sample SRR9982132) to the hg19 reference with minimap2 using '-ax map-ont' and '--secondary=no' so only primary alignments feed breakpoint calling, convert the resulting SAM to BAM, sort the BAM by genomic coordinate, and build the index with Samtools. Coordinate sorting plus the .bai index is what allows AgileStructure to rapidly query split-read alignments around candidate breakpoints such as chr6:167,281,717 and chr8:113,696,089. Keep only primary alignments (suppress secondaries at the Minimap2 stage) because the AgileStructure breakpoint rule — 2 or more split reads with junction positions within 250 nucleotides — assumes clean primary evidence. The indexed BAM then becomes the direct input for the Variant determination menu, where per-read annotations are compiled into a consensus translocation call.

## Related tools

- **minimap2** (Generates the SAM alignments from ONT/PacBio long reads against hg19, configured with -ax map-ont and --secondary=no so primary alignments feed SV detection) — https://github.com/lh3/minimap2
- **AgileStructure** (Consumes the sorted, indexed BAM to display split-read alignments and annotate breakpoints of large-scale structural rearrangements) — https://github.com/msjimc/AgileStructure
- **Samtools** (Converts SAM to BAM, sorts by genomic position, and indexes the BAM for AgileStructure)

## Examples

```
minimap2 -ax map-ont --secondary=no hg19.fa reads.fq > aln.sam && samtools sort -o aln.sorted.bam aln.sam && samtools index aln.sorted.bam
```

## Evaluation signals

- The BAM is coordinate-sorted and a valid index file (.bai) is produced alongside it (samtools quickcheck / idxstats succeeds)
- Alignment records correspond to primary alignments only, consistent with minimap2 --secondary=no
- Split-read junctions cluster around the expected breakpoint loci (e.g. chr6:167,281,717 and chr8:113,696,089 for SRR9982132)
- AgileStructure can load the BAM and display breakpoint-spanning reads without coordinate errors
- Final annotation offsets match the published call: chr6 within -1/+2 bp and chr8 within +9/+11 bp of the published positions

## Limitations

- Unsorted or unindexed BAMs cannot be efficiently queried by AgileStructure, so skipping the sort/index step breaks breakpoint inspection
- Alignment accuracy depends on the long-read aligner choice (minimap2 or lra) and reference build; coordinates are reference-specific (hg19 in the exemplar)
- WINE is needed to run the Windows-only AgileStructure binary on POSIX systems, adding setup overhead

## Evidence

- [intro] the data is typically aligned to the appropriate genome using a long-read-specific aligner such as minimap2: "the data is typically aligned to the appropriate genome using a long-read-specific aligner such as minimap2"
- [abstract] We developed AgileStructure, a desktop application for locating and annotating large-scale genomic rearrangements using aligned long-read data.: "We developed AgileStructure, a desktop application for locating and annotating large‑scale genomic rearrangements using aligned long‑read data."
- [other] Align the reads to hg19 with Minimap2 using the parameters -ax map-ont and --secondary=no. Convert the resulting SAM file to BAM, sort by genomic position, and index it with Samtools.: [evidence span withheld — anchor status: not_found]
- [readme] ./minimap2 -ax map-ont ref.fa ont.fq.gz > aln.sam: [evidence span withheld — anchor status: external_doc]
