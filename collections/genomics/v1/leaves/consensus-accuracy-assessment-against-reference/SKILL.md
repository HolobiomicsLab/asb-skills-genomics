---
name: consensus-accuracy-assessment-against-reference
description: Use when you have consensus sequences from a de novo long-read amplicon
  workflow applied to a mock community with known reference 16S rRNA sequences, and
  need to quantify accuracy and characterize residual error types.
when_to_use_negative:
- Input reads are raw noisy MinION reads that have not yet passed INC-Seq, chopSeq,
  and nanoClust consensus generation — accuracy values are only meaningful for called
  consensus sequences
- Community composition is unknown (no reference genomes/16S sequences available),
  so a reference-based accuracy comparison cannot be computed
- The target is short-read (e.g., Illumina MiSeq) amplicon data, where the nanopore
  homopolymer deletion error mode being benchmarked does not apply
edam_operation: http://edamontology.org/operation_3180
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3174
tools:
- name: chopSeq
  role: Fragments and orientation-corrects INC-Seq consensus reads upstream of the
    consensus sequences being assessed
  repo: github.com/umerijaz/nanopore
- name: nanoClust
  role: Performs read partitioning-based de novo clustering and within-cluster consensus
    calling to produce the per-OTU consensus sequences under evaluation
  repo: github.com/umerijaz/nanopore
- name: MAFFT
  role: G-INS-i alignment of ~50 reads per OTU used to call and validate consensus
    against the reference
- name: INC-Seq
  role: Rolling-circle alignment consensus step (3-concatemer threshold) that generates
    the circular consensus reads feeding the assessed workflow
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giy140
    title: 'NanoAmpli-Seq: a workflow for amplicon sequencing for mixed microbial
      communities on the nanopore sequencing platform'
  source_content_hash: sha256:19a4884eca151bf536a7f66a41ca2f2e695bf89fb3be4023b6cbb2fb5820dcd3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - INC-Seq
  - MAFFT
  - chopSeq
  - nanoClust
  edam_operation: http://edamontology.org/operation_3180
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# consensus-accuracy-assessment-against-reference

## Summary

Benchmarking skill for validating de novo consensus sequences (e.g., NanoAmpli-Seq OTU consensus 16S rRNA reads) by aligning each called consensus against known reference sequences and computing per-OTU and average accuracy, with error profiling of residual deletions in homopolymer regions.

## When to use

Use when you have consensus sequences from a de novo long-read amplicon workflow applied to a mock community with known reference 16S rRNA sequences, and need to quantify accuracy and characterize residual error types.

## When NOT to use

- Input reads are raw noisy MinION reads that have not yet passed INC-Seq, chopSeq, and nanoClust consensus generation — accuracy values are only meaningful for called consensus sequences
- Community composition is unknown (no reference genomes/16S sequences available), so a reference-based accuracy comparison cannot be computed
- The target is short-read (e.g., Illumina MiSeq) amplicon data, where the nanopore homopolymer deletion error mode being benchmarked does not apply

## Inputs

- Per-OTU consensus FASTA of full-length 16S rRNA gene sequences (nanoampliseq_consensus.fasta)
- Reference 16S rRNA gene sequences of the mock community organisms
- Within-cluster INC-Seq consensus reads (chopped, orientation-corrected) for per-OTU realignment

## Outputs

- Per-OTU accuracy values with expected range 99.2%–100%
- Average consensus accuracy across OTUs (expected ~99.5%)
- Error-type profile showing residual errors dominated by deletions in homopolymer regions
- Consensus composition record for both mock communities

## How to apply

After running the full NanoAmpli-Seq workflow (INC-Seq with a 3-concatemer threshold, chopSeq fragmentation/orientation correction, nanoClust partitioned de novo clustering with within-cluster consensus calling), align each called per-OTU consensus sequence from nanoampliseq_consensus.fasta against the reference 16S rRNA sequences of the mock community organisms. Use MAFFT with the G-INS-i iterative refinement option on ~50 reads per OTU when regenerating or validating within-cluster consensus, checking that the entire read aligns to the reference. Compute per-OTU accuracy and the average across OTUs; on the single- and 10-organism mock communities from the umerijaz/nanopore deposit, expect ~99.5% average accuracy with a per-OTU range of 99.2%–100%. Finally, tabulate error types to confirm residual errors are dominated by deletions in homopolymer regions.

## Related tools

- **chopSeq** (Fragments and orientation-corrects INC-Seq consensus reads upstream of the consensus sequences being assessed) — github.com/umerijaz/nanopore
- **nanoClust** (Performs read partitioning-based de novo clustering and within-cluster consensus calling to produce the per-OTU consensus sequences under evaluation) — github.com/umerijaz/nanopore
- **MAFFT** (G-INS-i alignment of ~50 reads per OTU used to call and validate consensus against the reference)
- **INC-Seq** (Rolling-circle alignment consensus step (3-concatemer threshold) that generates the circular consensus reads feeding the assessed workflow)

## Examples

```
mafft --ginsi --quiet otu_cluster.fasta > otu_consensus.aln; python compute_accuracy.py --consensus nanoampliseq_consensus.fasta --reference mock_refs.fasta
```

## Evaluation signals

- Average consensus accuracy across OTUs reaches ~99.5%
- Per-OTU accuracy falls within 99.2%–100%
- Each consensus read aligns entirely to its reference 16S rRNA sequence (full-length alignment, no truncation)
- Residual error counts are dominated by deletions concentrated in homopolymer regions
- Consensus composition of both mock communities (single-organism and 10-organism) matches the expected organism mixture

## Limitations

- Accuracy estimates require a mock community with known reference 16S rRNA sequences; de novo accuracy cannot be computed on natural communities without references
- Nearly all residual errors originate from deletions in homopolymer regions, an intrinsic nanopore base-calling limitation; homopolymer-aware base calling or error correction would be needed to reach short-read-platform accuracy
- Reads supporting fewer than 3 INC-Seq concatemers are discarded, so accuracy applies only to consensus calls meeting the 3-concatemer threshold

## Evidence

- [abstract] NanoAmpli-Seq accurately estimates the diversity of tested mock communities with average consensus sequence accuracy of 99.5% for 2D and 1D2 sequencing: "NanoAmpli-Seq accurately estimates the diversity of tested mock communities with average consensus sequence accuracy of 99.5% for 2D and 1D2 sequencing"
- [abstract] Nearly all residual errors in NanoAmpli-Seq sequences originate from deletions in homopolymer regions: "Nearly all residual errors in NanoAmpli-Seq sequences originate from deletions in homopolymer regions"
- [intro] generation of a consensus sequence without the use of any reference: "The more robust alternative towards high accuracy consensus sequence generation would be a completely de novo approach, i.e., generation of a consensus sequence without the use of any reference"
- [readme] Scripts for processing short amplicon reads from Oxford Nanopore: "Scripts for processing short amplicon reads from Oxford Nanopore. For details, please see:"
- [other] MAFFT-G-INS-i alignment of 50 reads per OTU followed by consensus calling yielded consensus reads with the entire read aligned: [evidence span withheld — anchor status: not_found]
