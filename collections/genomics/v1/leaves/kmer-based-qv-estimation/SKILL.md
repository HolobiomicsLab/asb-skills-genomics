---
name: kmer-based-qv-estimation
description: Reference for assessing assembly consensus accuracy (QV) when no trusted
  reference exists but high-coverage short reads (≥60X recommended) and a HiFi-based
  assembly are available, or when auditing where residual erroneous k-mers reside.
when_to_use_negative:
- When a high-confidence truth reference or GIAB benchmark set exists — variant-based
  evaluation (e.g. DeepVariant, paftools.js) is more informative than k-mer QV
- When short-read coverage is too low for reliable k-mer spectra (NextPolish2 recommends
  >=60X short reads), since QV will be dominated by coverage gaps rather than true
  errors
- When the genome is highly heterozygous without a phasing-aware setup, since Merqury's
  k-mer completeness and QV assumptions about haplotypes can be violated
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: Merqury
  role: Computes the QV of polished assemblies from short-read k-mer spectra; one
    of three evaluation approaches in the NextPolish2 benchmarks
- name: yak
  role: Builds k-mer dataset files (k=21 and k=31, -b37 to discard singletons) from
    short reads for QV computation and k-mer error detection
  repo: https://github.com/lh3/yak
- name: meryl
  role: Counts k-mers, including repetitive k=15 k-mers (greater-than distinct=0.9998)
    used to build sorted HiFi BAMs for downstream depth stratification
  repo: https://github.com/marbl/meryl
- name: winnowmap
  role: Maps HiFi reads to the assembly with a repetitive k-mer mask to produce the
    sorted BAM used for depth/identity stratification of erroneous k-mers
  repo: https://github.com/marbl/Winnowmap
- name: minimap2
  role: Alternative long-read mapper (map-pb/map-hifi presets) for producing HiFi
    alignments against the assembly
  repo: https://github.com/lh3/minimap2/
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/gpbjnl/qzad009
    title: 'NextPolish2: A Repeat-aware Polishing Tool for Genomes Assembled Using
      HiFi Long Reads'
  source_content_hash: sha256:aa911824b0c924bd0fd89ddd552c0d9642dcf9ea27cb25e37b7917284d884041
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/yak
  tools:
  - Merqury
  - NextPolish2
  - fastp
  - meryl
  - minimap2
  - winnowmap
  - yak
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# kmer-based-qv-estimation

## Summary

Estimate consensus base quality (QV) of genome assemblies by comparing assembly k-mers against short-read k-mer spectra with Merqury/yak, without needing a reference genome. Used here to quantify QV gains from polishing and to stratify residual erroneous k-mers by short-read mapping depth and spanning-read identity.

## When to use

Reference for assessing assembly consensus accuracy (QV) when no trusted reference exists but high-coverage short reads (≥60X recommended) and a HiFi-based assembly are available, or when auditing where residual erroneous k-mers reside.

## When NOT to use

- When a high-confidence truth reference or GIAB benchmark set exists — variant-based evaluation (e.g. DeepVariant, paftools.js) is more informative than k-mer QV
- When short-read coverage is too low for reliable k-mer spectra (NextPolish2 recommends >=60X short reads), since QV will be dominated by coverage gaps rather than true errors
- When the genome is highly heterozygous without a phasing-aware setup, since Merqury's k-mer completeness and QV assumptions about haplotypes can be violated

## Inputs

- Polished or unpolished genome assembly FASTA (e.g. HiFiasm primary assembly, NextPolish2 output asm.np2.fa)
- High-coverage Illumina short reads (FASTQ, paired-end; >=60X recommended)
- PacBio HiFi reads FASTQ (for mapping-depth and spanning-read identity stratification)
- yak k-mer dataset files (k21.yak, k31.yak)
- Sorted HiFi BAM alignments against the assembly

## Outputs

- Assembly QV estimate (e.g. asm-sr.qv.txt from yak qv or Merqury QV table)
- QV-versus-short-read-coverage table across downsampled coverage levels
- Set of erroneous k-mers (in assembly but absent from yak spectrum) annotated with local HiFi depth and spanning-read identity
- Per-genome fraction of erroneous k-mers in depth ≤3 regions and mean spanning-read identity

## How to apply

Build k-mer datasets from the short reads with yak (e.g. yak count -b37 at k=21 and k=31 to discard singletons) and run Merqury on the polished assembly to obtain the QV; NextPolish2's benchmark uses this to show QV rising with increasing short-read coverage (e.g. HG005 QV improved from 53.8544 to 55.6257 after polishing). To localize residual errors, treat k-mers present in the assembly but absent from the yak spectrum as erroneous, map them back to the polished assembly, and stratify by HiFi mapping depth (fraction in regions of depth ≤3) and mean spanning-read identity. Decision point: if a large fraction of erroneous k-mers falls in low-depth regions (paper values: Col-XJTU 34.52% at avg depth 104.82; HG002 76.29% at avg depth 32.75), the QV estimate is coverage-limited rather than reflecting true residual base errors. yak qv can be used directly for an assembly-vs-short-reads QV estimate as a lightweight alternative to the full Merqury pipeline.

## Related tools

- **Merqury** (Computes the QV of polished assemblies from short-read k-mer spectra; one of three evaluation approaches in the NextPolish2 benchmarks)
- **yak** (Builds k-mer dataset files (k=21 and k=31, -b37 to discard singletons) from short reads for QV computation and k-mer error detection) — https://github.com/lh3/yak
- **meryl** (Counts k-mers, including repetitive k=15 k-mers (greater-than distinct=0.9998) used to build sorted HiFi BAMs for downstream depth stratification) — https://github.com/marbl/meryl
- **winnowmap** (Maps HiFi reads to the assembly with a repetitive k-mer mask to produce the sorted BAM used for depth/identity stratification of erroneous k-mers) — https://github.com/marbl/Winnowmap
- **minimap2** (Alternative long-read mapper (map-pb/map-hifi presets) for producing HiFi alignments against the assembly) — https://github.com/lh3/minimap2/

## Examples

```
./yak count -b37 -t32 -o sr.yak <(zcat sr*.fq.gz) <(zcat sr*.fq.gz); ./yak qv -t32 -p -K3.2g -l100k sr.yak asm.np2.fa > asm-sr.qv.txt
```

## Evaluation signals

- QV should increase monotonically for the identical assembly as short-read coverage is downsampled upward (Col-XJTU trend reproduced across coverage series)
- Polished QV must exceed the unpolished Hifiasm QV (e.g. HG005: 53.8544 → 55.6257 after NextPolish2)
- Stratification should reproduce paper-reported fractions of erroneous k-mers in low-depth regions: Col-XJTU 34.52% at avg depth 104.82 and HG002 76.29% at avg depth 32.75
- Mean spanning-read identity of erroneous k-mer regions should match reported values (Col-XJTU 98.31%, HG002 95.37%)
- yak k-mer datasets must be built with -b37 singleton filtering; omitting it inflates error rates from singleton sequencing noise

## Limitations

- NextPolish2's performance (and thus the k-mer error calls) relies heavily on short reads to check whether a K-mer contains errors; regions with insufficient short-read coverage cannot be validated
- High-coverage short reads are needed to minimize uncorrected errors caused by biases inherent in short-read sequencing technologies; no specific coverage threshold is stated beyond the >=60X recommendation
- QV estimated at low coverage conflates true assembly errors with unvalidated low-depth regions (34.52%–76.29% of erroneous k-mers can lie at depth ≤3)

## Evidence

- [methods] we employed three approaches to evaluate the genome's accuracy before and after polishing:

1. **Merqury:** To assess the quality value (QV).: "we employed three approaches to evaluate the genome's accuracy before and after polishing:

1. **Merqury:** To assess the quality value (QV)."
- [methods] Prepare k-mer dataset files ([yak](https://github.com/lh3/yak)) (We recommend using >=60X short reads).: "Prepare k-mer dataset files ([yak](https://github.com/lh3/yak)) (We recommend using >=60X short reads)."
- [methods] `NextPolish2` improved the QV of the assembled genome from 53.8544 to 55.6257: "`NextPolish2` improved the QV of the assembled genome from 53.8544 to 55.6257"
- [discussion] The performance of NextPolish2 relies heavily on short reads to check whether a K-mer contains errors.: "The performance of NextPolish2 relies heavily on short reads to check whether a K-mer contains errors."
- [readme] ./yak qv -t32 -p -K3.2g -l100k sr.yak asm.fa.gz > asm-sr.qv.txt: [evidence span withheld — anchor status: external_doc]
