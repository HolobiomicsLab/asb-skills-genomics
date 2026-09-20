---
name: tandem-repeat-consensus-calling
description: Use when processing isoCirc RCA nanopore reads (FASTA/FASTQ) where each
  read contains tandem copies of a circular RNA template and a per-read consensus
  sequence must be derived before BSJ and full-length isoform calling.
when_to_use_negative:
- Input reads are not RCA-derived (e.g. direct RNA or linear cDNA long reads) — the
  tandem-repeat consensus model assumes repeated copies of the same circular template
  within each read
- CircRNAs have already been consensus-called and you only need downstream BSJ/isoform
  filtering — applying TRF again is redundant
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0080
tools:
- name: isoCirc
  role: Python pipeline that orchestrates TRF consensus calling, minimap2 filtering/mapping,
    and high-confidence circRNA isoform identification from RCA nanopore reads
  repo: https://github.com/Xinglab/isoCirc
- name: Tandem Repeats Finder (TRF)
  role: Calls consensus sequences from tandem-repeat structure of RCA long reads (match
    2, mismatch 7, indel 7, max period 2000, min score 100)
- name: minimap2
  role: Filters and maps consensus sequences to the reference genome; requires version
    >= 2.11
  repo: https://github.com/lh3/minimap2
- name: LoRDEC
  role: Optional hybrid error correction of long reads with short-read data prior
    to consensus calling
- name: bedtools
  role: Dependency of the isoCirc pipeline; requires version >= 2.27.0
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-20459-8
    title: isoCirc catalogs full-length circular RNA isoforms in human transcriptomes
  source_content_hash: sha256:fda18efc1cbc4efd9d55dd6ea71cc9a051036b941034f2804e3d39888e1205d3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Xinglab/isoCirc
  tools:
  - LoRDEC
  - Python
  - Tandem Repeats Finder (TRF)
  - bedtools
  - isoCirc
  - minimap2
  - pip
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# tandem-repeat-consensus-calling

## Summary

Builds consensus sequences from rolling-circle-amplified (RCA) isoCirc nanopore reads by treating each read's repeated circRNA copies as tandem repeats and calling a consensus with Tandem Repeats Finder, then filtering and mapping the consensus with minimap2. This converts noisy multi-copy long reads into full-length circRNA isoform sequences suitable for BSJ identification.

## When to use

Use when processing isoCirc RCA nanopore reads (FASTA/FASTQ) where each read contains tandem copies of a circular RNA template and a per-read consensus sequence must be derived before BSJ and full-length isoform calling.

## When NOT to use

- Input reads are not RCA-derived (e.g. direct RNA or linear cDNA long reads) — the tandem-repeat consensus model assumes repeated copies of the same circular template within each read
- CircRNAs have already been consensus-called and you only need downstream BSJ/isoform filtering — applying TRF again is redundant

## Inputs

- isoCirc RCA nanopore long-read sequences (FASTA/FASTQ)
- Reference genome FASTA
- Gene annotation GTF
- circRNA annotation BED/GTF
- Optional short-read FASTA/FASTQ for LoRDEC hybrid error correction

## Outputs

- Consensus sequences of full-length circRNA isoforms derived from tandem-repeat RCA reads
- isocirc.out (35-column table including readCount per isoform)
- isocirc.bed (BSJ coordinates)
- isocirc_stats.out (run statistics)

## How to apply

Run the isoCirc pipeline, which invokes Tandem Repeats Finder (TRF) on the RCA long reads with defaults match 2, mismatch 7, indel 7, max period 2000, min score 100 to call a consensus from the tandem copies of the circular template; this reduces nanopore error rates by leveraging the multiple copies of the same circRNA molecule within a read. Consensus sequences shorter than 30 nt or representing fewer than 2.0 copies (min copy number 2.0) are discarded. Filtered consensus sequences are then aligned to the reference genome with minimap2 (>= 2.11) to support downstream identification of high-confidence back-splice junctions and full-length circRNA isoforms. Hybrid error correction with short reads via LoRDEC can optionally be applied before consensus calling. Validation mirrors the toy example: run the pipeline on `test_data` (read_toy.fa against chr16_toy.fa/gtf/bed) and confirm isocirc.out, isocirc.bed, and isocirc_stats.out are produced.

## Related tools

- **isoCirc** (Python pipeline that orchestrates TRF consensus calling, minimap2 filtering/mapping, and high-confidence circRNA isoform identification from RCA nanopore reads) — https://github.com/Xinglab/isoCirc
- **Tandem Repeats Finder (TRF)** (Calls consensus sequences from tandem-repeat structure of RCA long reads (match 2, mismatch 7, indel 7, max period 2000, min score 100))
- **minimap2** (Filters and maps consensus sequences to the reference genome; requires version >= 2.11) — https://github.com/lh3/minimap2
- **LoRDEC** (Optional hybrid error correction of long reads with short-read data prior to consensus calling)
- **bedtools** (Dependency of the isoCirc pipeline; requires version >= 2.27.0)

## Examples

```
isocirc -t 1 read_toy.fa chr16_toy.fa chr16_toy.gtf chr16_circRNA_toy.bed output
```

## Evaluation signals

- TRF parameters used match the published defaults: match 2, mismatch 7, indel 7, max period 2000, min score 100
- Consensus sequences pass minimap2 filters of min consensus length 30 and min copy number 2.0
- Toy run on isoCirc test_data (read_toy.fa, chr16_toy.fa, chr16_toy.gtf, chr16_circRNA_toy.bed) completes and emits isocirc.out, isocirc.bed, isocirc_stats.out
- Downstream BSJ calling on the consensus passes canonical GT/AG motif filtering with ≤1 mismatch/indel in the 20-bp exonic flank (0 in the 4-bp core) and min genomic back-splice distance 150
- On the 12-tissue dataset, per-tissue full-length isoform counts at read >= 1 reproduce reported values (testis 115,348; prostate 18,312; skeletal muscle 19,261)

## Limitations

- Nanopore raw-read errors can introduce false BSJs; consensus calling followed by stringent filtering is required to remove potential false positives, especially for BSJs involving novel splice sites
- Very short amplification products (consensus length < 30 nt or copy number < 2.0) cannot yield reliable consensus calls
- At current sequencing depth isoCirc is far from saturation, so additional consensus/isoforms may appear with deeper sequencing

## Evidence

- [methods] Consensus calling with Tandem Repeats Finder (TRF): "Consensus calling with Tandem Repeats Finder (TRF)):"
- [methods] Filtering and mapping of consensus sequences with minimap2: "Filtering and mapping of consensus sequences (minimap2):"
- [discussion] Stringent post-consensus criteria: "After calling consensus sequences from isoCirc raw reads, we use stringent criteria to remove potential false-positive detection"
- [readme] Dependency versions: [evidence span withheld — anchor status: not_found]
- [readme] Toy example invocation: "isocirc -t 1 read_toy.fa chr16_toy.fa chr16_toy.gtf chr16_circRNA_toy.bed output"
