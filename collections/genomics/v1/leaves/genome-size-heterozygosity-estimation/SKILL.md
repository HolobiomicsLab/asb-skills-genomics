---
name: genome-size-heterozygosity-estimation
description: Use when starting a de novo assembly of an uncharacterized eukaryote
  genome and you need an estimate of haploid genome size, repeat content, and heterozygosity
  from raw short reads before choosing assembly parameters.
when_to_use_negative:
- When only long nanopore reads are available and no raw Illumina short-read set exists
  for k-mer profiling
- When a high-quality reference genome already exists and genome size can be measured
  directly rather than estimated
- When reads cannot be truncated to a uniform length, making k-mer depth distributions
  incomparable across the read set
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: Jellyfish
  role: Counts k-mer frequencies for each k in the range 19–25 on 76-nt-truncated
    Illumina reads, producing a k-mer histogram file per k
- name: GenomeScope
  role: Fits a k-mer statistical model to each Jellyfish histogram to yield haploid
    genome size, repeat content, and heterozygosity estimates
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-017-07650-6
    title: Rapid de novo assembly of the European eel genome from nanopore sequencing
      reads
  source_content_hash: sha256:2049e80cf36645bc83a8a198d72c7807746308bef0c600ba5fda45672b1386ca
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GenomeScope
  - Jellyfish
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# genome-size-heterozygosity-estimation

## Summary

Estimates haploid genome size, repetitive fraction, and heterozygous fraction from k-mer frequency profiles of raw Illumina reads using Jellyfish counting and GenomeScope model fitting, as benchmarked on the ~860 Mbp European eel (Anguilla anguilla) genome.

## When to use

Use when starting a de novo assembly of an uncharacterized eukaryote genome and you need an estimate of haploid genome size, repeat content, and heterozygosity from raw short reads before choosing assembly parameters.

## When NOT to use

- When only long nanopore reads are available and no raw Illumina short-read set exists for k-mer profiling
- When a high-quality reference genome already exists and genome size can be measured directly rather than estimated
- When reads cannot be truncated to a uniform length, making k-mer depth distributions incomparable across the read set

## Inputs

- Raw Illumina sequencing reads (e.g. SRA accession SRR5235521 for Anguilla anguilla)
- Uniformly truncated FASTQ reads (all reads truncated to 76 nt)
- Choice of k-mer length k (range 19–25)

## Outputs

- Jellyfish k-mer histogram file per k value
- GenomeScope fit parameters: haploid genome size (Mbp), repetitive fraction (%), heterozygous fraction (%)
- Model fit statistics written to genomescope_summary.txt

## How to apply

Download a raw Illumina read set (e.g. SRR5235521 for A. anguilla) and truncate all reads to a uniform length of 76 nt so that k-mer quality is consistent across the read length. Count k-mer frequencies with Jellyfish for a range of k values (k = 19–25), producing one k-mer histogram per k. Fit the GenomeScope model to each Jellyfish histogram to obtain haploid genome size, repeat content, and heterozygosity estimates, and select the fit consistent with prior knowledge (here, the ~860 Mbp genome size from k-mer analysis of the European eel). For A. anguilla the expected result is a haploid genome size of 854.0–866.5 Mbp, a repetitive fraction of 15.5–20.0%, and a heterozygous fraction of 1.48–1.59%. Write the fitted parameters and model fit statistics to a summary file (e.g. genomescope_summary.txt).

## Related tools

- **Jellyfish** (Counts k-mer frequencies for each k in the range 19–25 on 76-nt-truncated Illumina reads, producing a k-mer histogram file per k)
- **GenomeScope** (Fits a k-mer statistical model to each Jellyfish histogram to yield haploid genome size, repeat content, and heterozygosity estimates)

## Examples

```
jellyfish count -m 21 -s 100M -C reads_76nt.fastq -o kmers.jf && jellyfish histo kmers.jf > histo_k21.txt  # then fit GenomeScope to histo_k21.txt
```

## Evaluation signals

- genomescope_summary.txt exists and contains the fitted parameters and model fit statistics
- Estimated haploid genome size falls within the reported 854.0–866.5 Mbp range for A. anguilla
- Estimated repetitive fraction is consistent with the reported 15.5–20.0% range
- Estimated heterozygous fraction is consistent with the reported 1.48–1.59% range
- Result is consistent with the ~860 Mbp genome size from k-mer analysis reported for the European eel genome

## Limitations

- The estimate depends on uniform read length: k-mer quality must be uniform, hence all reads are truncated to 76 nt before counting
- It is uncertain whether inflated short-read assembly sizes or missing genomic fractions explain size discrepancies versus the k-mer-based estimate
- Without a high-quality reference genome it is not straightforward to establish which assembly (or size estimate) is correct

## Evidence

- [discussion] K-mer based genome size estimation: "the genome size predicted from k-mer analysis (~860 Mbp)"
- [discussion] Hybrid assembly size uncertainty: "This can be explained either by the absence of a substantial fraction of the genome from the nanopore data or assembly, or by an artificially inflated genome size for the short-read assemblies"
- [discussion] TULIP eel genome context: "Using our original TULIP methodology, we were able to assemble the 860 Mbp genome of the European eel"
- [discussion] Missing reference caveat: "In the absence of a high-quality reference, it is not straightforward to establish which assembly is correct"
- [other] Workflow target parameters: [evidence span withheld — anchor status: not_found]
