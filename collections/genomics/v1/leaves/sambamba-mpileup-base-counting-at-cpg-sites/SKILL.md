---
name: sambamba-mpileup-base-counting-at-cpg-sites
description: Use when you need base-level pileup counts at CpG positions from base-converted
  nanopore BAM alignments (CT and GA strands) as input for CpG methylation frequency
  calling.
when_to_use_negative:
- Native (unconverted) nanopore reads where methylation should be called directly
  from signal, e.g. with nanopolish call-methylation — pileup-based counting of converted
  bases does not apply.
- Short-read EMseq/bisulfite data processed with standard short-read aligners; the
  nanoEM pileup step depends on long-read minimap2 map-ont alignments of converted
  reads.
- Whole-genome variant calling at all positions — this setup is deliberately restricted
  to CpG positions via cpg_sites.bed, not genome-wide.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_0622
tools:
- name: sambamba
  role: Runs mpileup over the CpG-restricted BED regions on the CT and GA best-alignment
    BAMs to produce pileup tables
  repo: https://github.com/yos-sk/nanoEM
- name: samtools
  role: Indexes and sorts the BAM files fed to sambamba mpileup
  repo: https://github.com/yos-sk/nanoEM
- name: minimap2
  role: Aligns the C-to-T / G-to-A converted reads (map-ont, --eqx) to the converted
    reference to create the sorted BAMs pileup is computed from
  repo: https://github.com/yos-sk/nanoEM
- name: call_methylation.py
  role: Combines pileup_CT.tsv and pileup_GA.tsv, balancing the two strands to resolve
    per-CpG methylation frequencies
  repo: https://github.com/yos-sk/nanoEM
- name: nanopore sequencing (PromethION)
  role: Sequenced the enzymatically base-converted DNA from which the converted reads
    originate
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkab397
    title: Long-read whole-genome methylation patterning using enzymatic base conversion
      and nanopore sequencing
  source_content_hash: sha256:7d7d637c93d158117a75c8c807d06b1a009f8e65a04e7cfce591a9e6259709d1
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/yos-sk/nanoEM
  repo_url: https://github.com/yos-sk/nanoEM
  tools:
  - Python3
  - call_methylation.py
  - minimap2
  - nanopore sequencing (PromethION)
  - pysam
  - sambamba
  - samtools
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# sambamba-mpileup-base-counting-at-cpg-sites

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Restricts sambamba mpileup (v0.7.1, default parameters) to reference-genome CpG sites via a BED file to count bases from the C-to-T and G-to-A converted, best-aligned nanoEM reads, producing per-CpG pileup tables for downstream methylation calling.

## When to use

Use when you need base-level pileup counts at CpG positions from base-converted nanopore BAM alignments (CT and GA strands) as input for CpG methylation frequency calling.

## When NOT to use

- Native (unconverted) nanopore reads where methylation should be called directly from signal, e.g. with nanopolish call-methylation — pileup-based counting of converted bases does not apply.
- Short-read EMseq/bisulfite data processed with standard short-read aligners; the nanoEM pileup step depends on long-read minimap2 map-ont alignments of converted reads.
- Whole-genome variant calling at all positions — this setup is deliberately restricted to CpG positions via cpg_sites.bed, not genome-wide.

## Inputs

- output_CT.sorted.bam (indexed best unique alignment BAM of C-to-T converted reads)
- output_GA.sorted.bam (indexed best unique alignment BAM of G-to-A converted reads)
- cpg_sites.bed (CpG sites extracted from the reference genome)
- unconverted reference genome FASTA (passed via -f ref)

## Outputs

- pileup_CT.tsv (mpileup over CpG sites from CT-converted reads)
- pileup_GA.tsv (mpileup over CpG sites from GA-converted reads)
- frequency_methylation.tsv (per-CpG methylation frequencies after call_methylation.py)

## How to apply

Extract CpG sites from the unconverted reference genome into a BED file (cpg_sites.bed) so the pileup is restricted to CpG positions. Run sambamba mpileup (v0.7.1, default parameters) with 8 threads on the CT best-alignment BAM using -L cpg_sites.bed, supplying the unconverted reference genome with --samtools -f ref, and write pileup_CT.tsv; repeat identically on the GA best-alignment BAM to write pileup_GA.tsv. Using the best unique alignments (from best_align.py) is essential because the mostly C–T converted, error-prone reads can map ambiguously. The resulting pileup tables are combined by src/call_methylation.py, which balances the two strands to resolve the methylation state per CpG.

## Related tools

- **sambamba** (Runs mpileup over the CpG-restricted BED regions on the CT and GA best-alignment BAMs to produce pileup tables) — https://github.com/yos-sk/nanoEM
- **samtools** (Indexes and sorts the BAM files fed to sambamba mpileup) — https://github.com/yos-sk/nanoEM
- **minimap2** (Aligns the C-to-T / G-to-A converted reads (map-ont, --eqx) to the converted reference to create the sorted BAMs pileup is computed from) — https://github.com/yos-sk/nanoEM
- **call_methylation.py** (Combines pileup_CT.tsv and pileup_GA.tsv, balancing the two strands to resolve per-CpG methylation frequencies) — https://github.com/yos-sk/nanoEM
- **nanopore sequencing (PromethION)** (Sequenced the enzymatically base-converted DNA from which the converted reads originate)

## Examples

```
sambamba mpileup output_CT.sorted.bam -L cpg_sites.bed -o pileup_CT.tsv -t 8 --samtools -f ref; sambamba mpileup output_GA.sorted.bam -L cpg_sites.bed -o pileup_GA.tsv -t 8 --samtools -f ref
```

## Evaluation signals

- pileup_CT.tsv and pileup_GA.tsv exist and are tab-separated pileup tables restricted to positions listed in cpg_sites.bed
- frequency_methylation.tsv exists and contains one row per CpG site from cpg_sites.bed
- All methylation frequency values in frequency_methylation.tsv fall within [0, 1]
- Both CT and GA pileups were produced with identical parameters (8 threads, same BED, same reference) so strand balancing in call_methylation.py is valid
- mpileup was run on the best unique alignment BAMs (output_CT.sorted.bam / output_GA.sorted.bam), not on the raw minimap2 BAMs

## Limitations

- nanoEM cannot distinguish between 5-methylcytosine (mC) and 5-hydroxymethylcytosine (hmC) on the same DNA molecule, unlike direct nanopore WGS calling.
- The mpileup is restricted to CpG sites; methylation in non-CpG contexts is not captured.
- No software version numbers are stated for the pipeline tools in the article; sambamba v0.7.1 is known only from the source card, and counts depend on the quality of C–T converted long-read alignments at error-prone positions.

## Evidence
