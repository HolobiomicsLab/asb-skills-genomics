---
name: homopolymer-deletion-error-characterization
description: Use when you have consensus reads from a nanopore amplicon workflow (FASTA/FASTQ)
  plus reference 16S sequences and need to quantify whether residual errors are deletions
  versus insertions and whether they localize to homopolymer tracts.
when_to_use_negative:
- Input is raw uncorrected nanopore reads with very high error rates, where alignment
  ratios are too noisy to attribute errors to homopolymers reliably
- The goal is consensus generation or clustering itself (use chopSeq/nanoClust workflow
  steps) rather than post-hoc error attribution
- The reference database lacks the organism's 16S sequence, so alignments are unavailable
  or incorrect mapping would confound ratio interpretation
edam_operation: http://edamontology.org/operation_3187
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: blastn
  role: Aligns consensus reads at each pipeline stage to reference 16S sequences to
    obtain alignment lengths and mismatch/insertion/deletion patterns
- name: chopSeq
  role: Produces one of the intermediate read sets (fragmented, orientation-corrected
    INC-Seq consensus reads) whose error profile is compared
  repo: github.com/umerijaz/nanopore
- name: nanoClust
  role: Produces the OTU-consensus reads whose residual errors are the final attribution
    target
  repo: github.com/umerijaz/nanopore
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/gigascience/giy140
    title: 'NanoAmpli-Seq: a workflow for amplicon sequencing for mixed microbial
      communities on the nanopore sequencing platform'
  source_content_hash: sha256:5ca10c2174188993b2d23e7067479e0c0778dcf22e0389d67180a40e454a1c65
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - blastn
  - chopSeq
  - nanoClust
  edam_operation: http://edamontology.org/operation_3187
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 4
---

# homopolymer-deletion-error-characterization

## Summary

Attributes residual consensus errors in nanopore amplicon pipelines (INC-Seq, chopSeq, nanoClust) to homopolymer deletions by computing read-to-reference alignment length ratios with blastn and tabulating mismatch/insertion/deletion patterns.

## When to use

Use when you have consensus reads from a nanopore amplicon workflow (FASTA/FASTQ) plus reference 16S sequences and need to quantify whether residual errors are deletions versus insertions and whether they localize to homopolymer tracts.

## When NOT to use

- Input is raw uncorrected nanopore reads with very high error rates, where alignment ratios are too noisy to attribute errors to homopolymers reliably
- The goal is consensus generation or clustering itself (use chopSeq/nanoClust workflow steps) rather than post-hoc error attribution
- The reference database lacks the organism's 16S sequence, so alignments are unavailable or incorrect mapping would confound ratio interpretation

## Inputs

- FASTA files of consensus reads at each pipeline stage (INC-Seq consensus reads, chopSeq-fragmented reads, nanoClust OTU-consensus sequences) from github.com/umerijaz/nanopore
- Reference 16S rRNA gene sequences (mock-community references / SILVA-derived references)

## Outputs

- Per-read alignment length ratio distribution (alignment length / reference length) per pipeline stage
- Counts of single base-pair deletions located in homopolymer tracts >4 bp
- Attribution statement of residual error mode (deletions vs insertions) supporting consensus accuracy estimates such as 99.5%

## How to apply

Retrieve the deposited FASTA files (INC-Seq consensus reads, chopSeq-fragmented reads, nanoClust OTU-consensus sequences) and run blastn against the reference 16S sequences (mock-community or SILVA-derived references), recording alignment lengths and aligned reference intervals. Parse the tabular blastn output with Biopython and pandas to compute, per read, the alignment length ratio (alignment length divided by reference length); ratios predominantly <1 indicate that deletions dominate over insertions. Then extract mismatch/insertion/deletion patterns from the alignments and count single base-pair deletions falling within homopolymer tracts longer than 4 bp. Tabulate the ratio distributions per pipeline stage (INC-Seq, chopSeq, OTU consensus) to see where errors arise and whether downstream consensus calling removes or retains them.

## Related tools

- **blastn** (Aligns consensus reads at each pipeline stage to reference 16S sequences to obtain alignment lengths and mismatch/insertion/deletion patterns)
- **chopSeq** (Produces one of the intermediate read sets (fragmented, orientation-corrected INC-Seq consensus reads) whose error profile is compared) — github.com/umerijaz/nanopore
- **nanoClust** (Produces the OTU-consensus reads whose residual errors are the final attribution target) — github.com/umerijaz/nanopore

## Examples

```
blastn -query otu_consensus.fasta -subject references.fasta -outfmt 6 -out alignments.tsv  # then parse with Biopython/pandas to compute alignment_length/reference_length ratios
```

## Evaluation signals

- Alignment length ratios (alignment length / reference length) are predominantly <1 at each stage, confirming deletions dominate over insertions
- Residual consensus errors concentrate as single base-pair deletions within homopolymer tracts longer than 4 bp, with no detectable insertions
- The reproduced output matches the reported attribution consistent with the paper's 99.5% average consensus accuracy for 2D and 1D2 sequencing
- Ratio distributions are tabulated consistently across all three stages (INC-Seq, chopSeq, nanoClust OTU consensus) for comparison

## Limitations

- Residual errors concentrate in a few homopolymer regions (>4 bp), so error attribution is specific to homopolymer-associated deletions and may not capture other error modes
- The analysis assumes correct reference mapping; incorrect mapping to a reference database is likely due to high error rates of raw nanopore reads
- Homopolymer-aware base calling or error correction would be needed to remove these errors; this skill characterizes but does not correct them

## Evidence

- [abstract] Nearly all residual errors in NanoAmpli-Seq sequences originate from deletions in homopolymer regions: "Nearly all residual errors in NanoAmpli-Seq sequences originate from deletions in homopolymer regions"
- [results] homopolymer >4bp finding: [evidence span withheld — anchor status: not_found]
- [abstract] homopolymer-aware correction: "indicating that homopolymer aware base calling or error correction may allow for sequencing accuracy comparable to short-read sequencing platforms"
- [abstract] consensus accuracy: "NanoAmpli-Seq accurately estimates the diversity of tested mock communities with average consensus sequence accuracy of 99.5% for 2D and 1D2 sequencing on the nanopore sequencing platform"
- [readme] repository README: "Scripts for processing short amplicon reads from Oxford Nanopore. For details, please see:"
