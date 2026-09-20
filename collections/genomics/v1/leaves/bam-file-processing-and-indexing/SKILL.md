---
name: bam-file-processing-and-indexing
description: Use when long-read (ONT/PacBio) SAM alignments from minimap2 must be
  converted, coordinate-sorted and indexed into BAM before breakpoint browsing in
  AgileStructure or IGV, e.g. for tandem-duplication annotation.
when_to_use_negative:
- The input BAM is already coordinate-sorted and indexed (re-running sort/index adds
  nothing).
- The data are short Illumina reads — AgileStructure and the map-ont preset target
  long-read data; short data will not yield the split-read evidence the downstream
  annotation relies on.
- The SAM comes from an aligner other than a long-read-specific one (minimap2/lra),
  so alignment semantics differ from the AgileStructure workflow.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: minimap2
  role: Long-read-specific aligner producing the SAM that is converted to BAM; run
    with '-ax map-ont' and '--secondary=no' for Oxford Nanopore data
  repo: https://github.com/lh3/minimap2
- name: Samtools
  role: Converts minimap2 SAM output to BAM, sorts it by genomic position, and indexes
    it for AgileStructure
- name: AgileStructure
  role: Consumer of the sorted, indexed BAM; used to screen up-to-10-Mb blocks, find
    split-read breakpoints, and annotate structural variants
  repo: https://github.com/msjimc/AgileStructure
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btag294
    title: Utilization of long-read sequencing for the detection of structural rearrangements
      with AgileStructure
  source_content_hash: sha256:50c19b0c501d1e9a0dfdb4ee04f9a89f1354d5987f0c7786667f0c20d82a9d5e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - AgileStructure
  - Samtools
  - minimap2
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# bam-file-processing-and-indexing

## Summary

Prepares long-read alignments for manual structural-variant inspection by converting minimap2 SAM output (using the '-ax map-ont' preset with '--secondary=no') into a coordinate-sorted, indexed BAM via Samtools, so tools like AgileStructure can randomly access and browse breakpoint-spanning reads.

## When to use

Use when long-read (ONT/PacBio) SAM alignments from minimap2 must be converted, coordinate-sorted and indexed into BAM before breakpoint browsing in AgileStructure or IGV, e.g. for tandem-duplication annotation.

## When NOT to use

- The input BAM is already coordinate-sorted and indexed (re-running sort/index adds nothing).
- The data are short Illumina reads — AgileStructure and the map-ont preset target long-read data; short data will not yield the split-read evidence the downstream annotation relies on.
- The SAM comes from an aligner other than a long-read-specific one (minimap2/lra), so alignment semantics differ from the AgileStructure workflow.

## Inputs

- FASTQ of ONT (map-ont) or PacBio long reads, e.g. NCBI SRA sample SRR12783028
- Reference genome FASTA, e.g. mouse mm10 obtained from the UCSC Genome Browser
- SAM file produced by minimap2 '-ax map-ont --secondary=no'

## Outputs

- Coordinate-sorted BAM file produced by Samtools
- BAM index (.bai/.csi) enabling random-access read retrieval
- Sorted/indexed BAM loadable in AgileStructure for split-read breakpoint annotation (e.g. chr12:37,206,133_37,300,424dup)

## How to apply

Align the ONT long reads to the reference (e.g. mouse mm10 downloaded from the UCSC Genome Browser) with minimap2 using the '-ax map-ont' preset and '--secondary=no' to suppress secondary alignment records. Convert the resulting SAM to BAM with Samtools, sort the BAM by genomic position, and index it — indexed BAMs are required because AgileStructure screens large loci (e.g. the chr12 Agmo region) in blocks of up to 10 Mb and needs random access to reads. Load the sorted, indexed BAM in AgileStructure and identify breakpoints by locating two or more split reads whose primary and secondary alignment junction positions fall within 250 nucleotides of each other. Correct indexing is validated when AgileStructure (or IGV) opens the BAM directly and split reads spanning the breakpoint can be selected and annotated (e.g. via Variant determination > Use soft clip data).

## Related tools

- **minimap2** (Long-read-specific aligner producing the SAM that is converted to BAM; run with '-ax map-ont' and '--secondary=no' for Oxford Nanopore data) — https://github.com/lh3/minimap2
- **Samtools** (Converts minimap2 SAM output to BAM, sorts it by genomic position, and indexes it for AgileStructure)
- **AgileStructure** (Consumer of the sorted, indexed BAM; used to screen up-to-10-Mb blocks, find split-read breakpoints, and annotate structural variants) — https://github.com/msjimc/AgileStructure

## Examples

```
minimap2 -ax map-ont --secondary=no mm10.fa SRR12783028.fq.gz > aln.sam && samtools sort -o sorted.bam aln.sam && samtools index sorted.bam
```

## Evaluation signals

- Samtools sort completes with the BAM header reporting coordinate-sorted order, and 'samtools index' produces a companion index file without error.
- '--secondary=no' was used, so downstream breakpoint calls depend on split-read (primary vs secondary alignment junction) positions within 250 nucleotides rather than unmapped secondary records.
- AgileStructure loads the indexed BAM and displays reads when a chr12 block of up to 10 Mb around the Agmo locus is screened.
- Split reads linked to the breakpoint can be selected in the upper and lower panels and per-read annotations merged into a single generic annotation (e.g. a tandem duplication).
- On the SRR12783028 validation case, the saved annotation reports chr12:37,206,133_37,300,424dup, matching the published chr12:37,206,133-37,300,425dup on the 5' side and within -1 bp on the 3' side, with a per-read-annotation-median coordinate recorded.

## Limitations

- AgileStructure is a Windows desktop application; on Linux, BSD or macOS it must be run via WINE on Intel or AMD CPUs.
- With '--secondary=no' the breakpoint evidence comes from split reads' primary and secondary alignment junctions recorded in the alignment data, so alignments lacking such split/supplementary records will not support breakpoint annotation.
- SAM files without genomic sorting and indexing cannot be efficiently screened in AgileStructure's up-to-10-Mb block browsing mode.

## Evidence

- [intro] Long-read alignments are the substrate for BAM preparation: "the data is typically aligned to the appropriate genome using a long-read-specific aligner such as minimap2"
- [readme] minimap2 map-ont preset for ONT reads: [evidence span withheld — anchor status: external_doc]
- [abstract] Downstream consumer of the indexed BAM: "We developed AgileStructure, a desktop application for locating and annotating large‑scale genomic rearrangements using aligned long‑read data."
