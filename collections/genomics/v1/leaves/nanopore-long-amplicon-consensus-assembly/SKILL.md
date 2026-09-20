---
name: nanopore-long-amplicon-consensus-assembly
description: Use when demultiplexed MinION long-amplicon (e.g. rDNA) reads must be
  converted into a per-specimen consensus sequence of sufficient accuracy to compare
  against an Illumina short-read reference, with a minimum per-position coverage cutoff.
when_to_use_negative:
- Input is already assembled consensus sequences — this skill covers read-level demultiplexing,
  assembly, and polishing, not consensus-vs-consensus comparison alone
- Reads are Illumina short reads; the workflow is built around Nanopore long rDNA
  amplicons with dual barcode indexes and primer trimming
- A specimen lacks ≥30 reads per position in the alignment; the coverage threshold
  cannot be enforced and the consensus would not be validated
edam_operation: http://edamontology.org/operation_3184
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0622
tools:
- name: MiniBar
  role: Demultiplexes dual-indexed MinION reads by barcode/primer matching, trims
    barcodes and primers (-T), and bins records into per-sample FASTA files (-F -P)
  repo: https://github.com/calacademy-research/minibar
- name: Edlib
  role: Python library module required by minibar.py for edit-distance-based barcode
    and primer matching
  repo: https://github.com/Martinsos/edlib
- name: Allele Wrangler
  role: Aligns trimmed reads per specimen and assembles the per-specimen rDNA consensus
    with the 30-reads-per-position coverage threshold
- name: Racon
  role: Polishes the assembled Nanopore consensus to reduce residual raw-read errors
    before benchmarking
- name: Oxford Nanopore MinION
  role: Instrument producing the long dual-indexed rDNA amplicon reads used as input
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giz006
    title: Nanopore sequencing of long ribosomal DNA amplicons enables portable and
      simple biodiversity assessments with high phylogenetic resolution across broad
      taxonomic scale
  source_content_hash: sha256:c13935108678f7c8f5d750d5340c9a756aadfc6278c3b4faf3d0a0f485ef1b64
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/calacademy-research/minibar
  tools:
  - Allele Wrangler
  - Edlib
  - MiniBar
  - Oxford Nanopore MinION
  - Python 2.7
  - Racon
  edam_operation: http://edamontology.org/operation_3184
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# nanopore-long-amplicon-consensus-assembly

## Summary

Assembles accurate per-specimen consensus sequences from dual-indexed Nanopore long rDNA amplicon reads by demultiplexing with MiniBar, building a consensus with a minimum 30-reads-per-position coverage threshold, and polishing with Racon so the Nanopore consensus can be benchmarked against Illumina-based consensus sequences.

## When to use

Use when demultiplexed MinION long-amplicon (e.g. rDNA) reads must be converted into a per-specimen consensus sequence of sufficient accuracy to compare against an Illumina short-read reference, with a minimum per-position coverage cutoff.

## When NOT to use

- Input is already assembled consensus sequences — this skill covers read-level demultiplexing, assembly, and polishing, not consensus-vs-consensus comparison alone
- Reads are Illumina short reads; the workflow is built around Nanopore long rDNA amplicons with dual barcode indexes and primer trimming
- A specimen lacks ≥30 reads per position in the alignment; the coverage threshold cannot be enforced and the consensus would not be validated

## Inputs

- Dual-indexed Nanopore MinION read set (FASTA or FASTQ, plain or gzipped), e.g. PeperomiaTestSet.fasta
- Barcode/index combination file mapping sample IDs to fwd/rev indexes and primers, e.g. IndexCombinationPeperomonia.txt (MinIndexCombinationPeperomonia.txt)
- Corresponding Illumina-based consensus sequences to serve as reference for distance benchmarking

## Outputs

- Per-sample demultiplexed, barcode- and primer-trimmed FASTA files (e.g. Peperomia_Jun_38, Peperomia_Jun_39, Peperomia_Jun_40, unk) when using -F with a prefix such as Peperomia_
- Polished per-specimen rDNA consensus sequence meeting the 30-reads-per-position coverage threshold
- Per-specimen Nanopore-vs-Illumina pairwise distances and summary averages (e.g. 0.165%, 0.187%, 2.072%)

## How to apply

First demultiplex dual-indexed Nanopore reads with minibar.py using the barcode-combination file (e.g. IndexCombinationPeperomonia.txt) with the -T trim and -F -P <prefix> options so barcodes and primers are trimmed and records are binned into per-sample FASTA files (e.g. Peperomia_Jun_38/39/40 plus unk). Next, align the trimmed reads of each specimen and assemble a per-specimen rDNA consensus with Allele Wrangler, enforcing the minimum coverage threshold of 30 reads per position. Then polish each Nanopore consensus with RACON error correction to reduce residual raw-read errors. Finally, compute pairwise genetic distances between each Nanopore rDNA consensus and the corresponding Illumina-based consensus using pairwise sequence alignment, and tabulate per-specimen distances and their averages. Decision point: MiniBar index edit distance affects consensus accuracy — edit distances 2 and 3 yielded the smallest final consensus error rates (0.165% and 0.187% average distance to Illumina-based consensus) versus 2.072% at edit distance 4, so assign the tightest reliable barcode matching settings.

## Related tools

- **MiniBar** (Demultiplexes dual-indexed MinION reads by barcode/primer matching, trims barcodes and primers (-T), and bins records into per-sample FASTA files (-F -P)) — https://github.com/calacademy-research/minibar
- **Edlib** (Python library module required by minibar.py for edit-distance-based barcode and primer matching) — https://github.com/Martinsos/edlib
- **Allele Wrangler** (Aligns trimmed reads per specimen and assembles the per-specimen rDNA consensus with the 30-reads-per-position coverage threshold)
- **Racon** (Polishes the assembled Nanopore consensus to reduce residual raw-read errors before benchmarking)
- **Oxford Nanopore MinION** (Instrument producing the long dual-indexed rDNA amplicon reads used as input)

## Examples

```
minibar.py IndexCombinationPeperomonia.txt PeperomiaTestSet.fasta -T -F -P Peperomia_
```

## Evaluation signals

- Computed average pairwise Nanopore-vs-Illumina distances reproduce the reported values 0.165%, 0.187%, and 2.072% within rounding tolerance
- Every specimen consensus meets the minimum coverage threshold of 30 reads per position
- MiniBar run summary line reports high identification counts with no ambiguous sample IDs (e.g. test set: 750 seqs: H 750 HH 679 Hh 62 hh 0 IDs 741 Mult_IDs 0)
- Demultiplexed records have barcodes and primers trimmed from both sequence ends when the -T option is supplied
- Polished consensus error rate against the Illumina reference is markedly lower for index edit distances 2–3 than for edit distance 4

## Limitations

- Consensus accuracy depends strongly on MiniBar index edit distance: edit distance 4 gave 2.072% average distance to the Illumina-based consensus, an order of magnitude worse than distances 2 (0.165%) and 3 (0.187%)
- Records whose reverse primer is not matched within the maximum 11 edit distances are flagged with lowercase 'h' and -1 in the hit info and are counted separately (Hh) in the run summary
- Coverage below 30 reads per position means the consensus does not meet the enforced threshold and should not be reported

## Evidence

- [results] Consensus accuracy significantly increased for MiniBar index edit distances of two (0.165% average distance to Illumina-based consensus) and three (0.187% average distance), with edit distance two showing the smallest final consensus error rate.: [evidence span withheld — anchor status: not_found]
- [methods] Demultiplex and bin per-sample reads with trim.: "$ minibar.py IndexCombinationPeperomonia.txt PeperomiaTestSet.fasta >PeperomiaTestSet_SampleIDs.fa"
- [readme] Trim option removes barcodes and primers for downstream processing.: "For downstream processing, you'll probably want to trim off the barcode and primer part of the sequence. This is done for you if you supply the -T trim option"
- [readme] Bin records by sample with the -F option and a custom prefix.: "Typically you'll want to group your records by identified sample. This can be done by using the -F option"
- [readme] Primer edit distance tolerance default of 11 explains Hh-record flagging.: "-E <int> primer edit distance value (11)"
