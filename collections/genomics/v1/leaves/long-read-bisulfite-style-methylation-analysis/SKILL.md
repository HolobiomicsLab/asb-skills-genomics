---
name: long-read-bisulfite-style-methylation-analysis
description: Use when you have CT- and GA-converted nanopore reads already mapped
  to a converted reference with alignments selected, and you need per-CpG methylation
  frequencies genome-wide.
when_to_use_negative:
- You want methylation called directly from native (unconverted) nanopore signal data
  — use nanopolish call-methylation instead, which also can distinguish mC from hmC
  whereas nanoEM cannot
- Reads are short-read bisulfite/EM-seq data (e.g. processed with Trim Galore) — the
  pileup-balancing across CT/GA long-read alignments does not apply
- BAMs are raw minimap2 alignments rather than the best unique alignments from best_align.py
  — per-CpG counts would be contaminated by non-best mappings
edam_operation: http://edamontology.org/operation_3207
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_0622
tools:
- name: sambamba
  role: Computes mpileup over the reference CpG sites (restricted by -L cpg_sites.bed,
    8 threads, --samtools -f ref) separately for the CT and GA best-alignment BAMs
  repo: https://github.com/yos-sk/nanoEM
- name: call_methylation.py
  role: Python3/pysam script that balances the CT and GA pileup tables to resolve
    methylation state per CpG and outputs frequency_methylation.tsv
  repo: https://github.com/yos-sk/nanoEM
- name: minimap2
  role: Upstream mapper used to align converted reads (map-ont preset, --eqx) before
    best-align selection; not used in the pileup step itself
- name: nanopolish
  role: Alternative direct signal-level methylation caller from native nanopore WGS,
    contrasted with the conversion-based approach of this skill
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkab397
    title: Long-read whole-genome methylation patterning using enzymatic base conversion
      and nanopore sequencing
  source_content_hash: sha256:96ee37c3dcd35e021f3f6262671e45cfbc9e5c0686567a50a430d54b8f1e2104
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
  - nanopolish
  - pysam
  - sambamba
  edam_operation: http://edamontology.org/operation_3207
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# long-read-bisulfite-style-methylation-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Calls per-CpG methylation frequencies from base-converted (C-to-T / G-to-A) nanopore long-read alignments by running sambamba mpileup over reference CpG sites on both strand-specific best-alignment BAMs and combining the two pileups with call_methylation.py.

## When to use

Use when you have CT- and GA-converted nanopore reads already mapped to a converted reference with best alignments selected, and you need per-CpG methylation frequencies genome-wide.

## When NOT to use

- You want methylation called directly from native (unconverted) nanopore signal data — use nanopolish call-methylation instead, which also can distinguish mC from hmC whereas nanoEM cannot
- Reads are short-read bisulfite/EM-seq data (e.g. processed with Trim Galore) — the pileup-balancing across CT/GA long-read alignments does not apply
- BAMs are raw minimap2 alignments rather than the best unique alignments from best_align.py — per-CpG counts would be contaminated by non-best mappings

## Inputs

- output_CT.sorted.bam (indexed best-alignment BAM of C-to-T converted reads)
- output_GA.sorted.bam (indexed best-alignment BAM of G-to-A converted reads)
- cpg_sites.bed (CpG positions extracted from the reference genome)
- unconverted reference genome FASTA (passed to sambamba mpileup via -f ref)

## Outputs

- pileup_CT.tsv (sambamba mpileup over CpG sites for the CT strand)
- pileup_GA.tsv (sambamba mpileup over CpG sites for the GA strand)
- frequency_methylation.tsv (per-CpG methylation frequency table)

## How to apply

First extract CpG sites from the unconverted reference genome into cpg_sites.bed to restrict pileup to CpG positions. Run sambamba mpileup (v0.7.1, default parameters, 8 threads, restricted to cpg_sites.bed via -L, using the unconverted reference with --samtools -f ref) on the CT best-alignment BAM to produce pileup_CT.tsv, and identically on the GA best-alignment BAM to produce pileup_GA.tsv; the CT pileup reads methylated-state evidence from one strand while the GA pileup covers the complementary strand after G-to-A conversion. Combine the two pileup tables with python src/call_methylation.py, which balances the two strands to resolve the methylation state per CpG, writing per-CpG methylation frequencies to frequency_methylation.tsv. Correct application requires the pileups to come from best unique alignments (output of best_align.py), not raw minimap2 mappings, because converted-read multi-mapping otherwise contaminates CpG-level counts.

## Related tools

- **sambamba** (Computes mpileup over the reference CpG sites (restricted by -L cpg_sites.bed, 8 threads, --samtools -f ref) separately for the CT and GA best-alignment BAMs) — https://github.com/yos-sk/nanoEM
- **call_methylation.py** (Python3/pysam script that balances the CT and GA pileup tables to resolve methylation state per CpG and outputs frequency_methylation.tsv) — https://github.com/yos-sk/nanoEM
- **minimap2** (Upstream mapper used to align converted reads (map-ont preset, --eqx) before best-align selection; not used in the pileup step itself)
- **nanopolish** (Alternative direct signal-level methylation caller from native nanopore WGS, contrasted with the conversion-based approach of this skill)

## Examples

```
sambamba mpileup output_CT.sorted.bam -L cpg_sites.bed -o pileup_CT.tsv -t 8 --samtools -f ref; sambamba mpileup output_GA.sorted.bam -L cpg_sites.bed -o pileup_GA.tsv -t 8 --samtools -f ref; python src/call_methylation.py pileup_CT.tsv pileup_GA.tsv > frequency_methylation.tsv
```

## Evaluation signals

- frequency_methylation.tsv exists and is a tab-separated table with one row per CpG site listed in cpg_sites.bed
- All methylation frequency values fall within the [0, 1] range
- pileup_CT.tsv and pileup_GA.tsv were generated with identical mpileup parameters (-L cpg_sites.bed, -t 8, --samtools, -f ref) so strand balancing is valid
- BAM inputs are the sorted, indexed best-alignment files (output_CT.sorted.bam, output_GA.sorted.bam) produced by best_align.py, not the raw 1.sorted.bam / 2.sorted.bam minimap2 outputs

## Limitations

- nanoEM cannot distinguish between 5-methylcytosine and 5-hydroxymethylcytosine on the same DNA molecule, unlike direct nanopore WGS calling with nanopolish
- No software version numbers are stated for the pipeline tools in the article or README, aside from sambamba v0.7.1 in the source card; reproducibility requires pinning versions manually
- Frequencies depend on the quality of best unique alignments of error-prone, base-converted reads; coverage at CpG islands can be limiting (5.9–20.1 M pass reads, 3.4–7.6 kb N50 per PromethION flow cell)

## Evidence

- [methods] sambamba version and default parameters: [evidence span withheld — anchor status: not_found]
