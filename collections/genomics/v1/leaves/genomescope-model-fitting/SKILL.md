---
name: genomescope-model-fitting
description: Use when raw Illumina FASTQ reads are available and an assembly-independent
  estimate of haploid genome size, repeat content, and heterozygosity is needed to
  validate or contextualize a draft genome assembly.
when_to_use_negative:
- Input data are long nanopore reads rather than short Illumina reads — k-mer histograms
  from error-prone long reads will distort the GenomeScope model fit
- A high-quality reference genome already exists for the species — direct reference-based
  estimation is preferable to model fitting
- The goal is assembly itself rather than genome size estimation — use an assembly
  pipeline (e.g. TULIP for uncorrected long reads) instead
edam_operation: http://edamontology.org/operation_0206
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Jellyfish
  role: Counts k-mer frequencies on the truncated reads for each k in the range 19–25,
    producing the histogram files that GenomeScope fits
- name: GenomeScope
  role: Fits the k-mer histogram model to estimate haploid genome size, repetitive
    fraction, and heterozygous fraction
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-017-07650-6
    title: Rapid de novo assembly of the European eel genome from nanopore sequencing
      reads
  source_content_hash: sha256:7250b521bbe1325d7ed2878229d81aedb0d7c91ab4bd46c5f4dde508e60a4094
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GenomeScope
  - Jellyfish
  edam_operation: http://edamontology.org/operation_0206
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# genomescope-model-fitting

## Summary

Estimates haploid genome size, repetitive fraction, and heterozygosity by fitting the GenomeScope model to Jellyfish k-mer frequency histograms derived from raw Illumina reads, providing an assembly-independent genome size (here ~860 Mbp for A. anguilla).

## When to use

Use when raw Illumina FASTQ reads are available and an assembly-independent estimate of haploid genome size, repeat content, and heterozygosity is needed to validate or contextualize a draft genome assembly.

## When NOT to use

- Input data are long nanopore reads rather than short Illumina reads — k-mer histograms from error-prone long reads will distort the GenomeScope model fit
- A high-quality reference genome already exists for the species — direct reference-based estimation is preferable to model fitting
- The goal is assembly itself rather than genome size estimation — use an assembly pipeline (e.g. TULIP for uncorrected long reads) instead

## Inputs

- Raw Illumina paired-end FASTQ reads (e.g. SRA accession SRR5235521)
- Read-length-truncated FASTQ (all reads truncated to 76 nt for uniform k-mer quality)
- Jellyfish k-mer histogram files, one per k in the range 19–25

## Outputs

- GenomeScope fit summary with haploid genome size (854.0–866.5 Mbp)
- Repetitive fraction estimate (15.5–20.0%)
- Heterozygous fraction estimate (1.48–1.59%)
- genomescope_summary.txt containing fitted parameters and model fit statistics

## How to apply

First normalize k-mer quality across the read set by truncating all reads to a uniform length (here 76 nt) so that k-mer accuracy does not vary with position in the read. Count k-mer frequencies with Jellyfish across a range of k values (k = 19–25), producing one histogram file per k. Fit the GenomeScope model to each Jellyfish histogram to obtain haploid genome size, repetitive fraction, and heterozygous fraction estimates, then select the fit whose haploid genome size is consistent with expectations for the organism (854.0–866.5 Mbp for A. anguilla, consistent with the ~860 Mbp k-mer-analysis genome size reported in the article). Writing the fitted parameters and fit statistics to a summary file (e.g. genomescope_summary.txt) makes the result verifiable. Cross-check the estimate against assembly sizes from independent drafts, keeping in mind that short-read assemblies may be artificially inflated.

## Related tools

- **Jellyfish** (Counts k-mer frequencies on the truncated reads for each k in the range 19–25, producing the histogram files that GenomeScope fits)
- **GenomeScope** (Fits the k-mer histogram model to estimate haploid genome size, repetitive fraction, and heterozygous fraction)

## Examples

```
jellyfish count -m 21 -s 100M -C -t 8 -o kmers21.jf reads_truncated76.fastq && jellyfish histo -o kmers21.histo kmers21.jf
```

## Evaluation signals

- genomescope_summary.txt exists and contains fitted model parameters
- Reported haploid genome size falls within the expected 854.0–866.5 Mbp range
- Estimated repetitive fraction lies between 15.5% and 20.0%
- Estimated heterozygous fraction lies between 1.48% and 1.59%
- The estimate is consistent with the ~860 Mbp k-mer-analysis genome size reported for the European eel genome

## Limitations

- Short-read assembly sizes may be artificially inflated, so size discrepancies between k-mer estimates and assemblies can reflect either missing genomic fractions or inflated assembly sizes
- The estimate depends on uniform k-mer quality; variable read lengths require truncation (here to 76 nt) before counting
- Results vary across the k range tested (k = 19–25), so the reported size (854.0–866.5 Mbp), repeat (15.5–20.0%), and heterozygosity (1.48–1.59%) values are ranges rather than single point estimates

## Evidence

- [discussion] the genome size predicted from k-mer analysis (~860 Mbp): "the genome size predicted from k-mer analysis (~860 Mbp)"
- [discussion] inflated short-read assembly sizes vs missing genomic fraction: "This can be explained either by the absence of a substantial fraction of the genome from the nanopore data or assembly, or by an artificially inflated genome size for the short-read assemblies"
