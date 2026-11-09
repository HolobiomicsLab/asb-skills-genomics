---
name: corrected-reads-reference-alignment
description: Use when you have FASTA files of corrected long reads from self-correction
  tools and need alignment-based statistics (identity, genome coverage, N50) against
  a reference genome for benchmarking.
when_to_use_negative:
- Input is raw (uncorrected) long reads — raw read error rates make identity/coverage
  statistics meaningless for correction quality comparison
- No reference genome exists for the organism, since all statistics depend on reference
  alignment
- Goal is evaluating assembly polishing rather than read correction — use ELECTOR's
  assembly-evaluation mode instead of reference alignment of reads
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3573
tools:
- name: ELECTOR
  role: Reference-alignment module (built on minimap2) used to align each tool's corrected
    reads back to the reference genome and derive Table 2 statistics
- name: minimap2
  role: Underlying aligner inside ELECTOR's reference-alignment module; index split
    every 500M bases by default
- name: CONSENT
  role: One of the self-correction tools whose corrected-reads FASTA output is aligned
    and benchmarked; also dependency of the CONSENT repo which requires minimap2 on
    the path
  repo: https://github.com/morispi/CONSENT
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-020-80757-5
    title: Scalable long read self-correction and assembly polishing with multiple
      sequence alignment
  source_content_hash: sha256:f7a485ea483dd586d746e5ab3b1b03bbe14fc2b32ca9d1c7a9f245eae4827f10
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/morispi/CONSENT
  tools:
  - CONSENT
  - ELECTOR
  - minimap2
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3573
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# corrected-reads-reference-alignment

## Summary

Aligns corrected long reads back to a reference genome with ELECTOR's reference-alignment module (built on minimap2) to extract benchmark statistics — corrected reads, bases, N50, aligned-reads proportion, alignment identity, and genome coverage — used to compare self-correction tools like CONSENT, Canu, Daccord, FLAS, and MECAT.

## When to use

Use when you have FASTA files of corrected long reads from self-correction tools and need alignment-based statistics (identity, genome coverage, N50) against a reference genome for benchmarking.

## When NOT to use

- Input is raw (uncorrected) long reads — raw read error rates make identity/coverage statistics meaningless for correction quality comparison
- No reference genome exists for the organism, since all statistics depend on reference alignment
- Goal is evaluating assembly polishing rather than read correction — use ELECTOR's assembly-evaluation mode instead of reference alignment of reads

## Inputs

- Corrected long reads in FASTA format, one file per self-correction tool (e.g. CONSENT, Canu, Daccord, FLAS, MECAT outputs)
- Reference genome for each dataset (e.g. S. cerevisiae, D. melanogaster, H. sapiens chr 1)

## Outputs

- Minimap2-based alignments of corrected reads to the reference genome (from ELECTOR's reference-alignment module)
- Per-tool benchmark metrics: number of corrected reads, total bases, N50, proportion of aligned reads, alignment identity, genome coverage
- Compiled benchmark table (reproducing Table 2's columns) across all tools and datasets

## How to apply

After running each self-correction tool (e.g. CONSENT-correct, Canu, FLAS, MECAT) on the same long-read dataset, feed each tool's corrected-reads FASTA into ELECTOR's reference-alignment module, which aligns reads to the corresponding reference genome using minimap2 with its index split every 500M bases by default. From the alignment output, extract per-tool statistics: number of corrected reads, total bases, N50, proportion of aligned reads, alignment identity, and genome coverage. Compile these across all tools and datasets (e.g. S. cerevisiae PacBio SRR9617898, D. melanogaster ONT SRX3676783, H. sapiens chr 1 ONT PRJEB23027) into a benchmark table reproducing the paper's Table 2 columns. Judge correctness by verifying every metric is computable from the produced corrected-reads files and the ELECTOR alignment output, and that all tools are evaluated on identical inputs.

## Related tools

- **ELECTOR** (Reference-alignment module (built on minimap2) used to align each tool's corrected reads back to the reference genome and derive Table 2 statistics)
- **minimap2** (Underlying aligner inside ELECTOR's reference-alignment module; index split every 500M bases by default)
- **CONSENT** (One of the self-correction tools whose corrected-reads FASTA output is aligned and benchmarked; also dependency of the CONSENT repo which requires minimap2 on the path) — https://github.com/morispi/CONSENT

## Examples

```
./CONSENT-correct --in longReads.fastq --out correctedReads.fasta --type ONT  # then align correctedReads.fasta to the reference with ELECTOR's minimap2-based module
```

## Evaluation signals

- All eight Table 2 metrics (reads-corrected, bases, N50, aligned-reads proportion, alignment identity, genome coverage, runtime, memory) are reported for every tool on every dataset
- Each reported value is recomputable from the corrected-reads FASTA files and the ELECTOR/minimap2 alignment output
- Proportion of aligned reads and alignment identity fall in plausible ranges (0–1 and 0–100% respectively), with corrected reads expected to show higher identity than raw reads
- All tools are aligned against the same reference genome per dataset, ensuring cross-tool comparability
- Expected qualitative pattern: CONSENT yields the largest number of corrected reads, largest base output, and largest genome coverage across the S. cerevisiae, D. melanogaster, and H. sapiens chr 1 real datasets

## Limitations

- Requires a reference genome, so the evaluation is biased toward known regions and may not reflect correction of novel sequence
- Runtime and peak memory of the correction step itself must be recorded separately during tool execution; this skill only derives alignment-based metrics
- CONSENT's correction step runtime tends to be higher than that of other state-of-the-art methods, so runtime comparisons should not be inferred from alignment statistics

## Evidence

- [other] ELECTOR's reference-alignment module built on minimap2, with index split every 500M bases by default: [evidence span withheld — anchor status: not_found]
- [other] CONSENT corrected the largest number of reads, output the largest number of bases, and reached the largest genome coverage: [evidence span withheld — anchor status: not_found]
- [abstract] CONSENT compares well to the state-of-the-art, and performs better on real Oxford Nanopore data: "CONSENT compares well to the state-of-the-art, and performs better on real Oxford Nanopore data."
- [readme] Minimap2 available through you path: "Minimap2 available through you path."
