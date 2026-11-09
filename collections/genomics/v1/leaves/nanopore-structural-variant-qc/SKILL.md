---
name: nanopore-structural-variant-qc
description: Use when a raw NanoSV VCF from nanopore data (e.g. NA12878 chr1) needs
  stringent post-calling filtering before downstream comparison, phasing, or benchmarking.
when_to_use_negative:
- The input VCF has already been repeat- and size-filtered — reapplying the cascade
  would only shrink an already-final callset
- The goal is sensitive SV discovery rather than precision-benchmarked calling — the
  paper notes this stringent filtering lowers estimated sensitivity to ~75%
- The calls come from short-read (Illumina) data rather than long nanopore reads,
  where split-alignment assumptions behind the <40 bp indel rule do not hold
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: NanoSV
  role: Produces the initial split- and gapped-read based structural-variant VCF that
    this QC filtering cascade refines
  repo: mroosmalen/nanosv
- name: sambamba
  role: Converts and sorts LAST-mapped SAM output into the coordinate-sorted BAM that
    NanoSV requires before filtering
  repo: biod/sambamba
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-017-01343-4
    title: Mapping and phasing of structural variation in patient genomes using nanopore
      sequencing
  source_content_hash: sha256:e0a4d8df2a167f1dfbe1b5cb3011b9814ecf2927e161d112016f0e8354bdd061
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mroosmalen/nanosv
  tools:
  - NanoSV
  - sambamba
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# nanopore-structural-variant-qc

## Summary

Post-calling quality control for NanoSV structural-variant call sets derived from MinION nanopore reads, discarding calls whose breakpoints fall in homopolymer/short-tandem-repeat regions and indels smaller than the 40 bp SV size threshold that do not typically produce split alignments.

## When to use

Use when a raw NanoSV VCF from nanopore data (e.g. NA12878 chr1) needs stringent post-calling filtering before downstream comparison, phasing, or benchmarking.

## When NOT to use

- The input VCF has already been repeat- and size-filtered — reapplying the cascade would only shrink an already-final callset
- The goal is sensitive SV discovery rather than precision-benchmarked calling — the paper notes this stringent filtering lowers estimated sensitivity to ~75%
- The calls come from short-read (Illumina) data rather than long nanopore reads, where split-alignment assumptions behind the <40 bp indel rule do not hold

## Inputs

- NanoSV output VCF of structural-variant calls from MinION nanopore reads (e.g. NA12878 chromosome 1, 3957 initial calls)
- Homopolymer region annotations for the reference genome
- Short-tandem-repeat region annotations for the reference genome
- SV size threshold (40 bp) defining the minimum indel size retained

## Outputs

- Intermediate repeat-filtered VCF (657 SVs for NA12878 chr1)
- Final filtered VCF nanosv_na12878_filtered.vcf (654 SVs) in VCF format

## How to apply

Load the NanoSV VCF (e.g. 3957 initial calls for NA12878 chromosome 1) and parse each variant's coordinates, type, and size. Intersect SV breakpoint positions with homopolymer and tandem-repeat annotations and discard every call whose breakpoint lies inside such a repeat region, writing the intermediate callset (expected 657 calls) to an intermediate VCF. From that intermediate set, exclude indel calls shorter than the 40 bp SV threshold, since sub-40bp indels do not typically result in a split alignment and are the least reliable split-read signal. Write the final callset (expected 654 calls) to nanosv_na12878_filtered.vcf in VCF format. This aggressive cascade reflects the paper's design choice of extremely stringent filtering to keep precision above 95% at an estimated sensitivity of ~75%.

## Related tools

- **NanoSV** (Produces the initial split- and gapped-read based structural-variant VCF that this QC filtering cascade refines) — mroosmalen/nanosv
- **sambamba** (Converts and sorts LAST-mapped SAM output into the coordinate-sorted BAM that NanoSV requires before filtering) — biod/sambamba

## Examples

```
NanoSV -t 4 -s sambamba -b human_hg19.bed -o nanosv_na12878.vcf na12878_chr1.sorted.bam
```

## Evaluation signals

- Repeat-filter stage retains exactly 657 calls from the initial 3957 NA12878 chr1 NanoSV calls
- Indel-size filter produces a final callset of exactly 654 SVs, matching the reported filtering cascade
- Final output file nanosv_na12878_filtered.vcf is valid VCF with row counts equal to 654
- No retained call has a breakpoint inside a homopolymer or tandem-repeat annotation
- No retained indel call is smaller than 40 bp

## Limitations

- Extremely stringent filtering deliberately reduces sensitivity (~75%) — genuinely novel variants may be discarded
- The paper notes the conservative SV calling and post-calling filtering steps lower the fraction of novel variants relative to Pacific Biosciences data
- Creating larger SV catalogs in complex repeat regions and segmental duplications remains an unresolved challenge, so repeat filtering may remove true positives there

## Evidence

- [discussion] stringent filtering sensitivity: "we employ extremely stringent filtering that results in a good estimated sensitivity (~75%)"
- [discussion] precision benchmark: [evidence span withheld — anchor status: not_found]
- [readme] NanoSV algorithm description: "NanoSV identifies split- and gapped-aligned reads and clusters the reads according to the orientations and genomic positions of the read segments"
- [full_text] indel filter finding: [evidence span withheld — anchor status: not_found]
