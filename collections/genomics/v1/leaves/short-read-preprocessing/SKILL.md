---
name: short-read-preprocessing
description: Use when raw Illumina FASTQ reads have heterogeneous lengths or trailing
  low-quality bases that would distort k-mer spectra, and the downstream goal is k-mer
  histogram generation for genome size, repeat, and heterozygosity estimation.
when_to_use_negative:
- The downstream task is long-read (nanopore) assembly with TULIP/Canu, where reads
  must remain full-length and adaptive trimming of the 3'-end would discard signal
- Input reads are already uniform-length and quality-trimmed, so re-truncation only
  discards data
- Polishing a genome with Pilon using the Illumina reads, where maximum read length
  and full pairing may improve base correction
edam_operation: http://edamontology.org/operation_3232
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: sickle
  role: Sliding-window, adaptive, quality-based trimming of FASTQ reads (single-end
    or paired-end) before k-mer counting
  repo: https://github.com/najoshi/sickle
- name: Jellyfish
  role: K-mer frequency counting on the truncated reads, producing one k-mer histogram
    per k in 19–25
- name: GenomeScope
  role: Model fitting to Jellyfish histograms to estimate haploid genome size, repetitive
    fraction, and heterozygous fraction
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-017-07650-6
    title: Rapid de novo assembly of the European eel genome from nanopore sequencing
      reads
  source_content_hash: sha256:cba1a7042635991f37e7cf560eda2263ce8bbc4edf466635a1f2b4358331ae85
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/najoshi/sickle
  tools:
  - GenomeScope
  - Jellyfish
  - sickle
  edam_operation: http://edamontology.org/operation_3232
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# short-read-preprocessing

## Summary

Uniformize short-read (Illumina) input prior to k-mer-based genome profiling by quality/length trimming or truncation (e.g. truncating SRR5235521 reads to 76 nt), so that k-mer counts computed by Jellyfish across k=19–25 are of consistent quality before GenomeScope model fitting.

## When to use

Use when raw Illumina FASTQ reads have heterogeneous lengths or trailing low-quality bases that would distort k-mer spectra, and the downstream goal is k-mer histogram generation for genome size, repeat, and heterozygosity estimation.

## When NOT to use

- The downstream task is long-read (nanopore) assembly with TULIP/Canu, where reads must remain full-length and adaptive trimming of the 3'-end would discard signal
- Input reads are already uniform-length and quality-trimmed, so re-truncation only discards data
- Polishing a genome with Pilon using the Illumina reads, where maximum read length and full pairing may improve base correction

## Inputs

- Raw Illumina FASTQ read set (e.g. SRA accession SRR5235521)
- Read quality encoding specification (Illumina or Sanger/CASAVA >= 1.8)

## Outputs

- Trimmed/truncated FASTQ file with uniform-length reads (e.g. all reads at 76 nt)
- Jellyfish k-mer histogram files per k (k = 19–25)
- GenomeScope fitted parameters written to genomescope_summary.txt (haploid genome size, repetitive fraction, heterozygous fraction)

## How to apply

Start from the raw Illumina read set (e.g. SRR5235521 for A. anguilla) and trim or truncate all reads to a uniform length (76 nt in the A. anguilla workflow) so k-mer quality is uniform across the read length; alternatively use a sliding-window adaptive trimmer such as Sickle with an explicit quality encoding (sanger/illumina) and window thresholds. Feed the preprocessed FASTQ to Jellyfish for each k in 19–25, then fit GenomeScope to each histogram, selecting the fit whose haploid genome size matches the expected range (854.0–866.5 Mbp for the 860 Mbp European eel genome). The rationale is that mixed read lengths bias k-mer depth distributions toward low counts, corrupting GenomeScope's heterozygous (1.48–1.59%) and repetitive (15.5–20.0%) fraction estimates.

## Related tools

- **sickle** (Sliding-window, adaptive, quality-based trimming of FASTQ reads (single-end or paired-end) before k-mer counting) — https://github.com/najoshi/sickle
- **Jellyfish** (K-mer frequency counting on the truncated reads, producing one k-mer histogram per k in 19–25)
- **GenomeScope** (Model fitting to Jellyfish histograms to estimate haploid genome size, repetitive fraction, and heterozygous fraction)

## Examples

```
sickle se -f input_file.fastq -t illumina -o trimmed_output_file.fastq -q 33 -l 40
```

## Evaluation signals

- genomescope_summary.txt exists and reports a haploid genome size within 854.0–866.5 Mbp, consistent with the ~860 Mbp k-mer-analysis estimate for the European eel genome
- Fitted repetitive fraction falls in the 15.5–20.0% range and heterozygous fraction in the 1.48–1.59% range
- All reads in the preprocessed FASTQ have the target uniform length (76 nt), verifiable with a read-length histogram
- A Jellyfish k-mer histogram is produced for every k value in the 19–25 range with a single dominant depth peak

## Limitations

- Truncating reads to 76 nt discards bases beyond the cut point, reducing total k-mer coverage relative to full-length reads
- Sickle's Solexa quality setting is only an approximation (a non-linear transformation), so quality encoding must be chosen carefully
- Reads shorter than the minimum length threshold after trimming are discarded entirely, which can bias the k-mer spectrum

## Evidence

- [discussion] the genome size predicted from k-mer analysis (~860 Mbp): "the genome size predicted from k-mer analysis (~860 Mbp)"
- [readme] Sickle is a tool that uses sliding windows along with quality and length thresholds: [evidence span withheld — anchor status: external_doc]
- [readme] sickle se example command with quality type and length thresholds: [evidence span withheld — anchor status: external_doc]
