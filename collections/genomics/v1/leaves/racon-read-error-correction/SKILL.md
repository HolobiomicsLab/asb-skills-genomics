---
name: racon-read-error-correction
description: Use when you have an assembled Nanopore long-amplicon consensus (e.g.
  an rDNA consensus built with Allele Wrangler from MiniBar-demultiplexed MinION reads)
  and need to lower its residual error rate against an Illumina reference.
when_to_use_negative:
- The input is already an Illumina short-read consensus (reference quality) rather
  than a noisy Nanopore draft consensus — polishing targets raw-read errors in long-amplicon
  ONT data
- Only raw MinION reads are available with no assembled draft consensus — Racon requires
  a pre-assembled consensus to correct
- The goal is bulk-sample metabarcoding abundance analysis — the article states further
  research into the observed bias is required before long rDNA amplicon sequencing
  can be used reliably for bulk samples
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: Racon
  role: Error-corrects (polishes) each per-specimen Nanopore rDNA consensus to reduce
    residual raw-read errors before reference comparison
- name: MiniBar
  role: Demultiplexes dual-indexed MinION reads and trims barcodes/primers (-T) into
    per-sample FASTA inputs for consensus assembly
  repo: https://github.com/calacademy-research/minibar
- name: Edlib
  role: Edit-distance library module required by minibar.py for barcode/primer matching
    during demultiplexing
  repo: https://github.com/Martinsos/edlib
- name: Allele Wrangler
  role: Assembles the per-specimen rDNA consensus from trimmed reads with a minimum
    coverage of 30 reads per position; this consensus is the Racon input
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giz006
    title: Nanopore sequencing of long ribosomal DNA amplicons enables portable and
      simple biodiversity assessments with high phylogenetic resolution across broad
      taxonomic scale
  source_content_hash: sha256:8e03255be665023419bff98cc0cf0ead42d6d15c454b247085c1e3b1302e3f0b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/calacademy-research/minibar
  tools:
  - Allele Wrangler
  - Edlib
  - MiniBar
  - Python 2.7
  - Racon
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 20
---

# racon-read-error-correction

## Summary

Polishes per-specimen Nanopore rDNA consensus sequences with Racon to reduce residual raw-read errors before comparing them against Illumina-based consensus references, as in the Peperomia long-amplicon MinION barcoding workflow.

## When to use

Use when you have an assembled Nanopore long-amplicon consensus (e.g. an rDNA consensus built with Allele Wrangler from MiniBar-demultiplexed MinION reads) and need to lower its residual error rate against an Illumina reference.

## When NOT to use

- The input is already an Illumina short-read consensus (reference quality) rather than a noisy Nanopore draft consensus — polishing targets raw-read errors in long-amplicon ONT data
- Only raw MinION reads are available with no assembled draft consensus — Racon requires a pre-assembled consensus to correct
- The goal is bulk-sample metabarcoding abundance analysis — the article states further research into the observed bias is required before long rDNA amplicon sequencing can be used reliably for bulk samples

## Inputs

- Per-specimen draft Nanopore rDNA consensus FASTA assembled from MiniBar-trimmed MinION reads (coverage >= 30 reads per position)
- Trimmed per-sample MinION read FASTA files (e.g. Peperomia_Jun_38/39/40 produced by minibar.py -T -F)
- Illumina-based reference consensus sequences (Peperomia consensus sequences) for distance assessment

## Outputs

- Racon-polished per-specimen Nanopore rDNA consensus sequences
- Per-specimen and average pairwise Nanopore-vs-Illumina consensus distances (target values: 0.165%, 0.187%, 2.072%)

## How to apply

First demultiplex MinION reads with minibar.py using the dual-index barcode file (e.g. IndexCombinationPeperomonia.txt) with the -T trim option so barcodes and primers are removed, and bin records into per-sample FASTA files (e.g. Peperomia_Jun_38/39/40) with -F and a prefix such as Peperomia_. Align the trimmed reads of each specimen and assemble a per-specimen rDNA consensus with Allele Wrangler, enforcing the minimum coverage threshold of 30 reads per position before polishing. Run Racon error correction on each draft consensus to reduce residual raw-read errors inherited from the noisy MinION reads. Judge success by computing pairwise genetic distances between each polished Nanopore rDNA consensus and the corresponding Illumina-based Peperomia consensus; in the source study this reproduced averages of 0.165%, 0.187%, and 2.072% for MiniBar index edit distances 2, 3, and 4 respectively, with edit distance two showing the smallest final consensus error rate.

## Related tools

- **Racon** (Error-corrects (polishes) each per-specimen Nanopore rDNA consensus to reduce residual raw-read errors before reference comparison)
- **MiniBar** (Demultiplexes dual-indexed MinION reads and trims barcodes/primers (-T) into per-sample FASTA inputs for consensus assembly) — https://github.com/calacademy-research/minibar
- **Edlib** (Edit-distance library module required by minibar.py for barcode/primer matching during demultiplexing) — https://github.com/Martinsos/edlib
- **Allele Wrangler** (Assembles the per-specimen rDNA consensus from trimmed reads with a minimum coverage of 30 reads per position; this consensus is the Racon input)

## Examples

```
racon trimmed_minion_reads.fastq overlaps.paf allele_wranger_consensus.fasta > racon_polished_consensus.fasta
```

## Evaluation signals

- Every specimen consensus position meets the minimum coverage threshold of 30 reads before polishing
- Average pairwise Nanopore-vs-Illumina consensus distances reproduce the reported 0.165%, 0.187%, and 2.072% within rounding tolerance
- Edit distance 2 (MiniBar index) specimens show the smallest final consensus error rate, and edit distance 4 remains the worst (~2.072%)
- Each polished Nanopore consensus has a lower pairwise distance to its Illumina-based counterpart than the pre-polish draft
- Demultiplexing run summary shows reads assigned to expected sample IDs (e.g. Jun_38, Jun_39, Jun_40) with Mult_IDs 0

## Limitations

- Consensus quality depends critically on the MiniBar demultiplexing index edit distance — index edit distance 4 yielded ~2.072% average distance to Illumina consensus versus ~0.165% at distance 2, so mis-binned reads propagate into polished consensus
- Long rDNA amplicon metabarcoding recovers highly skewed diversity estimates; polishing does not resolve the taxonomic bias in bulk samples, whose sources remain undetermined
- minibar.py requires the Edlib module and Python 2.7 (Python 3 compatible), an environmental prerequisite upstream of polishing

## Evidence

- [other] Polish each Nanopore consensus with RACON error correction to reduce residual raw-read errors.: [evidence span withheld — anchor status: not_found]
- [other] assemble a per-specimen rDNA consensus with Allele Wrangler, enforcing the minimum coverage threshold of 30 reads per position: [evidence span withheld — anchor status: not_found]
- [results] the computed average pairwise Nanopore-vs-Illumina distances must reproduce the reported values 0.165%, 0.187%, and 2.072% within rounding tolerance: [evidence span withheld — anchor status: not_found]
- [intro] We compile a workflow from PCR amplification, to library preparation, to demultiplexing and consensus calling: "We compile a workflow from PCR amplification, to library preparation, to demultiplexing and consensus calling"
- [readme] ./minibar.py -T -F Demultiplex.txt my_sequences.fastq: "./minibar.py -T -F Demultiplex.txt my_sequences.fastq"
