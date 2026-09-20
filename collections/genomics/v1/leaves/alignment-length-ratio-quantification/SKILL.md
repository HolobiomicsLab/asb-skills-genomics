---
name: alignment-length-ratio-quantification
description: Use when you need to attribute residual errors in nanopore consensus
  reads (e.g., NanoAmpli-Seq OTU sequences) to indel classes by computing alignment-length/reference-length
  ratios from blastn output.
when_to_use_negative:
- Reads have not been corrected through the INC-Seq / chopSeq / nanoClust stages (raw
  noisy nanopore reads will not yield interpretable residual-error attribution).
- The goal is general consensus accuracy estimation rather than diagnosing the indel
  class or homopolymer localization of residual errors.
edam_operation: http://edamontology.org/operation_0297
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: blastn
  role: Aligns reads at each NanoAmpli-Seq stage against reference 16S sequences,
    recording alignment lengths and aligned reference intervals
  repo: https://github.com/umerijaz/nanopore
- name: chopSeq
  role: Produces the fragmented, orientation-corrected reads that constitute the intermediate
    stage of the alignment-ratio comparison
  repo: https://github.com/umerijaz/nanopore
- name: nanoClust
  role: Produces the OTU-consensus sequences whose residual errors are attributed
    via alignment length ratios
  repo: https://github.com/umerijaz/nanopore
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/gigascience/giy140
    title: 'NanoAmpli-Seq: a workflow for amplicon sequencing for mixed microbial
      communities on the nanopore sequencing platform'
  source_content_hash: sha256:eb5ed2c83ef990cfc64e663834d2a3fa3e5aed7ca613e8c4dec61bf4b6a6af76
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/umerijaz/nanopore
  tools:
  - blastn
  - chopSeq
  - nanoClust
  edam_operation: http://edamontology.org/operation_0297
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# alignment-length-ratio-quantification

## Summary

Quantifies per-read read-to-reference alignment length ratios from blastn alignments of NanoAmpli-Seq reads (INC-Seq consensus, chopSeq-fragmented, nanoClust OTU consensus) to attribute residual sequencing errors to deletions versus insertions, particularly in homopolymer regions.

## When to use

Use when you need to attribute residual errors in nanopore consensus reads (e.g., NanoAmpli-Seq OTU sequences) to indel classes by computing alignment-length/reference-length ratios from blastn output.

## When NOT to use

- Reads have not been corrected through the INC-Seq / chopSeq / nanoClust stages (raw noisy nanopore reads will not yield interpretable residual-error attribution).
- The goal is general consensus accuracy estimation rather than diagnosing the indel class or homopolymer localization of residual errors.

## Inputs

- FASTA files of INC-Seq consensus reads
- FASTA files of chopSeq-fragmented reads
- nanoClust OTU-consensus FASTA sequences
- Reference 16S rRNA sequences (mock-community references / SILVA-derived references)

## Outputs

- Per-read alignment length ratio table (alignment length / reference length) per processing stage
- Ratio distribution summaries per stage (INC-Seq, chopSeq, OTU consensus)
- Counts of single base-pair deletions within homopolymer tracts >4 bp

## How to apply

Retrieve the deposited FASTA files (INC-Seq consensus reads, chopSeq-fragmented reads, and nanoClust OTU-consensus sequences) from github.com/umerijaz/nanopore and run blastn against the reference 16S sequences (mock-community / SILVA-derived references). Parse the tabular blastn output with Biopython and pandas, computing for each read the alignment length divided by reference length, and tabulate the ratio distributions per processing stage. Ratios predominantly <1 indicate deletions dominate over insertions. Then extract mismatch/insertion/deletion patterns from the alignments and count single base-pair deletions within homopolymer tracts longer than 4 bp to localize the error source.

## Related tools

- **blastn** (Aligns reads at each NanoAmpli-Seq stage against reference 16S sequences, recording alignment lengths and aligned reference intervals) — https://github.com/umerijaz/nanopore
- **chopSeq** (Produces the fragmented, orientation-corrected reads that constitute the intermediate stage of the alignment-ratio comparison) — https://github.com/umerijaz/nanopore
- **nanoClust** (Produces the OTU-consensus sequences whose residual errors are attributed via alignment length ratios) — https://github.com/umerijaz/nanopore

## Examples

```
python -c "import pandas as pd; b=pd.read_csv('hits.tsv',sep='\t',header=None); print((b[3]/b[12]).describe())"  # blastn outfmt with qlen; ratio = aln_len/qlen
```

## Evaluation signals

- Alignment length ratios are predominantly <1 at the OTU-consensus stage, confirming deletions dominate over insertions
- Residual errors concentrate as single base-pair deletions within homopolymer tracts longer than 4 bp
- No detectable insertions in the final nanoClust OTU-consensus alignments
- Consensus accuracy of the analyzed OTU reads is consistent with the reported 99.5% average for 2D and 1D2 sequencing
- Reproduced ratio distributions for all three stages (INC-Seq, chopSeq, OTU consensus) match the reported error attribution

## Limitations

- The homopolymer-deletion attribution was established on tested mock communities with 2D and 1D2 ONT MinION data; results may not transfer directly to other platforms or chemistries
- Attribution depends on the quality and completeness of the reference database (mock-community or SILVA-derived references); incorrect reference mapping can bias indel classification
- Only deletions in homopolymers >4 bp were counted as the residual error class; other error modes outside this pattern will not be captured by this analysis

## Evidence

- [abstract] Nearly all residual errors in NanoAmpli-Seq sequences originate from deletions in homopolymer regions: "Nearly all residual errors in NanoAmpli-Seq sequences originate from deletions in homopolymer regions"
- [abstract] NanoAmpli-Seq consensus accuracy of 99.5%: "NanoAmpli-Seq accurately estimates the diversity of tested mock communities with average consensus sequence accuracy of 99.5%"
- [abstract] chopSeq fragmentation and orientation correction stage: "The developed protocol includes chopSeq software for fragmentation and read orientation correction of INC-Seq consensus reads"
- [readme] Repository README pointing to the deposited scripts and data: "Scripts for processing short amplicon reads from Oxford Nanopore."
