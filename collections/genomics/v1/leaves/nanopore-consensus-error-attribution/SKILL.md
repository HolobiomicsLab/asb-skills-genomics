---
name: nanopore-consensus-error-attribution
description: Use when FASTA consensus reads from a nanopore amplicon workflow (INC-Seq
  / chopSeq / nanoClust) must be error-profiled against 16S references, e.g. to test
  whether remaining errors are homopolymer deletions.
when_to_use_negative:
- When the input is raw noisy nanopore reads rather than corrected consensus sequences
  (INC-Seq, chopSeq, or nanoClust output) — this skill characterizes residual consensus
  error, not raw-read error rates
- When a reference database is unavailable or the goal is fully de novo consensus
  accuracy assessment, since the attribution depends on blastn alignment to 16S references
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0654
tools:
- name: blastn
  role: Aligns reads at each of the three workflow stages (INC-Seq, chopSeq, nanoClust
    OTU consensus) against reference 16S sequences to obtain alignment lengths and
    indel/mismatch patterns for ratio computation
  repo: github.com/umerijaz/nanopore
- name: chopSeq
  role: 'Produces one of the analyzed intermediate inputs: fragmentation and read
    orientation correction of INC-Seq consensus reads'
  repo: github.com/umerijaz/nanopore
- name: nanoClust
  role: Produces the final nanoClust OTU-consensus sequences whose residual errors
    are the target of the attribution analysis
  repo: github.com/umerijaz/nanopore
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/gigascience/giy140
    title: 'NanoAmpli-Seq: a workflow for amplicon sequencing for mixed microbial
      communities on the nanopore sequencing platform'
  source_content_hash: sha256:d21e8cb25d5de3a22ff65180f66b6f8f085a77933652ffb2815f3dd7d5e0a717
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - blastn
  - chopSeq
  - nanoClust
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# nanopore-consensus-error-attribution

## Summary

Attributes residual errors in Nanopore amplicon consensus reads (INC-Seq, chopSeq, nanoClust stages) by comparing read-to-reference blastn alignment length ratios and mapping mismatches/indels to homopolymer tracts. This distinguishes deletion-dominated base-calling error from other error classes in ONT MinION data.

## When to use

Use when FASTA consensus reads from a nanopore amplicon workflow (INC-Seq / chopSeq / nanoClust) must be error-profiled against 16S references, e.g. to test whether remaining errors are homopolymer deletions.

## When NOT to use

- When the input is raw noisy nanopore reads rather than corrected consensus sequences (INC-Seq, chopSeq, or nanoClust output) — this skill characterizes residual consensus error, not raw-read error rates
- When a reference database is unavailable or the goal is fully de novo consensus accuracy assessment, since the attribution depends on blastn alignment to 16S references

## Inputs

- FASTA files of INC-Seq consensus reads, chopSeq-fragmented reads, and nanoClust OTU-consensus sequences (from github.com/umerijaz/nanopore)
- Reference 16S rRNA gene FASTA sequences (mock-community references / SILVA-derived references)
- blastn tabular output (alignment lengths, aligned reference intervals, mismatch/indel patterns)

## Outputs

- Per-read alignment length ratios (alignment length / reference length) tabulated per workflow stage using pandas
- Distribution of alignment ratios showing whether ratios are predominantly <1 (deletion-dominated)
- Counts of single base-pair deletions located within homopolymer tracts >4 bp

## How to apply

Retrieve the deposited FASTA files (INC-Seq consensus reads, chopSeq-fragmented reads, and nanoClust OTU-consensus sequences) from github.com/umerijaz/nanopore and run blastn against the reference 16S sequences (mock-community / SILVA-derived references), recording alignment lengths and aligned reference intervals. Parse the tabular blastn output with Biopython and pandas to compute, per read, the alignment length ratio (alignment length divided by reference length). Tabulate the ratio distributions per stage (INC-Seq, chopSeq, OTU consensus) and check that ratios are predominantly <1, which indicates deletions dominate over insertions. Then extract mismatch/insertion/deletion patterns from the alignments and count single base-pair deletions within homopolymer tracts longer than 4 bp to attribute residual errors to homopolymer-associated deletion errors of the ONT MinION base caller.

## Related tools

- **blastn** (Aligns reads at each of the three workflow stages (INC-Seq, chopSeq, nanoClust OTU consensus) against reference 16S sequences to obtain alignment lengths and indel/mismatch patterns for ratio computation) — github.com/umerijaz/nanopore
- **chopSeq** (Produces one of the analyzed intermediate inputs: fragmentation and read orientation correction of INC-Seq consensus reads) — github.com/umerijaz/nanopore
- **nanoClust** (Produces the final nanoClust OTU-consensus sequences whose residual errors are the target of the attribution analysis) — github.com/umerijaz/nanopore

## Examples

```
blastn -query nanoclust_otu_consensus.fasta -subject silva_16S_refs.fasta -outfmt '6 qseqid sseqid length qstart qend mismatch gaps' > alignments.tsv
```

## Evaluation signals

- Alignment length ratios are predominantly <1 at each stage, indicating deletions dominate over insertions
- Residual consensus errors are concentrated as single base-pair deletions within homopolymer tracts longer than 4 bp
- No detectable insertions in the final NanoAmpli-Seq OTU consensus reads
- Consensus accuracy of the nanoClust OTU reads is consistent with the reported 99.5% average for 2D and 1D2 sequencing on mock communities
- Reproduced tabulations for all three stages (INC-Seq, chopSeq, OTU consensus) match the reported attribution pattern

## Limitations

- Attribution is specific to the homopolymer-deletion error profile of nanopore base calling; residual errors may be of other classes on different platforms or chemistries
- The analysis is reference-dependent: incorrect mapping to a reference database is likely due to high error rates of raw nanopore reads, and organisms absent from the reference cannot be assessed
- Fixing these errors requires homopolymer-aware base calling or error correction; the skill only attributes errors, it does not correct them

## Evidence

- [abstract] Nearly all residual errors in NanoAmpli-Seq sequences originate from deletions in homopolymer regions: "Nearly all residual errors in NanoAmpli-Seq sequences originate from deletions in homopolymer regions"
- [abstract] average consensus sequence accuracy of 99.5% for 2D and 1D2 sequencing: "NanoAmpli-Seq accurately estimates the diversity of tested mock communities with average consensus sequence accuracy of 99.5% for 2D and 1D2 sequencing on the nanopore sequencing platform"
- [intro] mapping to SILVA reference databases: "mapping raw, noisy reads to custom or publicly available reference databases (i.e., SILVA) [25]"
- [readme] Scripts for processing short amplicon reads from Oxford Nanopore.: "Scripts for processing short amplicon reads from Oxford Nanopore. For details, please see:"
