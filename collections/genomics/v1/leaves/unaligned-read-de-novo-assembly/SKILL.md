---
name: unaligned-read-de-novo-assembly
description: Use when a linked-read (10x/TELL-Seq/stLFR) BAM contains reads with unmapped
  ends or poor alignments and the goal is to discover non-reference insertions >300
  bp without whole-genome de novo assembly.
when_to_use_negative:
- The goal is whole-genome de novo assembly or variant classes other than long (>300
  bp) non-reference insertions — Novel-X processes only the small fraction of informative
  linked-reads and is tuned for NRS insertion discovery.
- 'Insertions are small: Novel-X needs a substantial number of distinct barcodes per
  insertion, and for small insertions the number of associated barcodes is usually
  too small for sufficient coverage.'
- Input is not barcode-linked short-read data (e.g. plain short reads or long Nanopore/PacBio
  reads) — the barcode collection and reassembly stages would have no barcode information
  to exploit.
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Novel-X
  role: Snakemake pipeline orchestrating the unaligned-read extraction and Velvet
    orphan-contig assembly stage
  repo: https://github.com/1dayac/Novel-X
- name: Velvet
  role: de novo assembler used on read set U to produce orphan contigs (k=63, MAXKMERLENGTH=63,
    commit 9adf09f)
  repo: https://github.com/dzerbino/velvet
- name: BLASTN
  role: optional --nt flag filters non-human orphan contigs (contamination removal,
    similar to Pamir)
- name: Samtools
  role: BAM handling during extraction of unaligned/poorly-aligned paired-end reads
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkac653
    title: Efficient detection and assembly of non-reference DNA sequences with synthetic
      long reads
  source_content_hash: sha256:fe822730ba7cd3f0ac9aa83c974c0067d896590887538556507a3333665954c0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/1dayac/Novel-X
  tools:
  - BLASTN
  - Novel-X
  - SPAdes
  - Samtools
  - Velvet
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# unaligned-read-de-novo-assembly

## Summary

Assembles paired-end reads extracted from a LongRanger-aligned BAM that are unaligned, low mapping quality (<10), or heavily soft-clipped (>20%), keeping only reads with average base quality > 20, into 'orphan contigs' with Velvet (k-mer length 63 compiled via MAXKMERLENGTH=63 for 10x data) as the first stage of Novel-X's non-reference sequence (NRS) insertion discovery pipeline.

## When to use

Use when a linked-read (10x/TELL-Seq/stLFR) BAM contains reads with unmapped ends or poor alignments and the goal is to discover non-reference insertions >300 bp without whole-genome de novo assembly.

## When NOT to use

- The goal is whole-genome de novo assembly or variant classes other than long (>300 bp) non-reference insertions — Novel-X processes only the small fraction of informative linked-reads and is tuned for NRS insertion discovery.
- Insertions are small: Novel-X needs a substantial number of distinct barcodes per insertion, and for small insertions the number of associated barcodes is usually too small for sufficient coverage.
- Input is not barcode-linked short-read data (e.g. plain short reads or long Nanopore/PacBio reads) — the barcode collection and reassembly stages would have no barcode information to exploit.

## Inputs

- LongRanger-aligned BAM of paired-end synthetic long reads (10x Chromium, TELL-Seq, or stLFR) with barcode tags
- Reference genome (e.g. GRCh38)

## Outputs

- Set of orphan contigs (FASTA) assembled from the unaligned read set U
- Contig-to-read alignments used downstream to collect barcode lists B(c)

## How to apply

Extract the paired-end read set U from the LongRanger-aligned BAM by selecting reads with at least one unaligned end, mapping quality below 10, or >20% soft-clipped bases, retaining only reads whose average base quality exceeds 20. Assemble U with Velvet using k-mer length 63 (the assembler must be compiled with MAXKMERLENGTH=63 because 63 exceeds Velvet's default maximum k for 10x data). Optionally pass --nt to BLASTN-filter non-human orphan contigs as a contamination-removal step analogous to Pamir. The rationale is that a whole-genome assembly is computationally wasteful: only the small fraction of informative unaligned/poorly-aligned reads carries the NRS signal, so local assembly of U yields candidate insertion contigs for downstream barcode-list extraction and reassembly with SPAdes (k=77, cov-cutoff=3).

## Related tools

- **Novel-X** (Snakemake pipeline orchestrating the unaligned-read extraction and Velvet orphan-contig assembly stage) — https://github.com/1dayac/Novel-X
- **Velvet** (de novo assembler used on read set U to produce orphan contigs (k=63, MAXKMERLENGTH=63, commit 9adf09f)) — https://github.com/dzerbino/velvet
- **BLASTN** (optional --nt flag filters non-human orphan contigs (contamination removal, similar to Pamir))
- **Samtools** (BAM handling during extraction of unaligned/poorly-aligned paired-end reads)

## Examples

```
git clone --recursive git@github.com:1dayac/Novel-X.git && cd Novel-X && git submodule update --init --recursive
```

## Evaluation signals

- Every read entering assembly has at least one unaligned end, MAPQ < 10, or >20% soft-clipped bases, and average base quality > 20.
- Velvet was compiled with MAXKMERLENGTH=63 so k-mer length 63 runs without error (default Velvet k max would fail).
- Orphan contigs are non-empty and each surviving contig recruits at least t=5 distinct barcodes B(c); contigs below t are discarded.
- Contigs whose best Minimap2 match is not an existing orphan contig are dropped after reassembly with SPAdes (k=77, cov-cutoff=3).
- On the demo run, the output VCF contains a chr1 record around position 29503 with an insertion sequence present and length/anchor thresholds satisfied.

## Limitations

- For small insertions, the number of associated barcodes is usually small and coverage is insufficient, so calls are missed.
- Non-recursive cloning breaks the pipeline: Novel-X requires 'git clone --recursive' or 'git submodule update --init --recursive' because it depends on submodules.
- A problem exists on the filter_target_contig stage at the moment, per the article's methods.
- Optional BLASTN (--nt) filtering is only a partial contamination-removal procedure; the paper suggests a Pamir-like procedure as an alternative.

## Evidence

- [methods] extraction of unaligned/poorly-aligned paired-end reads: "We choose paired-end reads with at least one end not aligned to the reference genome, or aligned but with a mapping quality below 10, or that have > 20% of soft-clipped bases"
- [methods] average base quality filter: "we choose only reads with the average base quality score > 20"
- [methods] barcode recruitment threshold: [evidence span withheld — anchor status: not_found]
- [intro] rationale for local instead of whole-genome assembly: "our method is computationally more efficient compared with those synthetic long-read approaches that use whole-genome de novo assembly such as (9,20) because we only process a very small fraction of"
- [readme] recursive clone requirement: "If you clone repository non-recursively Novel-X will not work. To fix this run from Novel-X folder:"
