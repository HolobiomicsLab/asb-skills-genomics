---
name: line1-source-element-locus-annotation
description: Use when paired tumor/control long-read (ONT or PacBio) data has been
  processed with nanomonsv's mobile element insertion workflow and you need to classify
  insertions as LINE1-mediated and resolve their 3' transduction source sites to define
  source element loci.
when_to_use_negative:
- When both breakpoints of an SV lie in poorly characterized highly repetitive regions
  — the Single breakend SV module cannot detect SVs where both breakpoints are in
  areas where reference genomes are not well-characterized
- When the event is a long repeat expansion (VNTR/microsatellite) rather than a LINE1
  insertion — the reference-genome-based framework may not capture long repeat expansion
  events
- When only short-read tumor/control data is available and full-length inserted sequences
  cannot be recovered, since tracing 3' transductions to source sites requires full-length
  insert sequence
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: nanomonsv
  role: Detects somatic insertions with single-base resolution from paired tumor/control
    long-read data and its mobile element insertion classification workflow resolves
    LINE1 3' transduction source sites from full-length inserted sequences
  repo: https://github.com/friend1ws/nanomonsv
- name: Oxford Nanopore Technologies (ONT)
  role: Long-read sequencing platform supplying the raw reads that enable full-length
    LINE1 insertion recovery and source-site resolution
- name: nanopolish
  role: Signal-level methylation calls (nanopolish index / call-methylation) evaluated
    at the annotated LINE1 source element loci in the downstream methylation analysis
  repo: https://github.com/jts/nanopolish
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkad526
    title: Precise characterization of somatic complex structural variations from
      tumor/control paired long-read sequencing data with nanomonsv
  source_content_hash: sha256:430431d6a8b9f29ceef44898e74736eac46f6501bb843d160e398b048a080709
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/friend1ws/nanomonsv
  tools:
  - Oxford Nanopore Technologies (ONT)
  - harmonicmeanp
  - nanomonsv
  - nanopolish
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# line1-source-element-locus-annotation

## Summary

Identifies and annotates the genomic source elements of LINE1 3' transductions from somatic mobile element insertions detected by nanomonsv, producing the reference set of LINE1 source loci (e.g. the 23 elements) that downstream methylation and rearrangement analyses are anchored to.

## When to use

Use when paired tumor/control long-read (ONT or PacBio) data has been processed with nanomonsv's mobile element insertion workflow and you need to classify insertions as LINE1-mediated and resolve their 3' transduction source sites to define source element loci.

## When NOT to use

- When both breakpoints of an SV lie in poorly characterized highly repetitive regions — the Single breakend SV module cannot detect SVs where both breakpoints are in areas where reference genomes are not well-characterized
- When the event is a long repeat expansion (VNTR/microsatellite) rather than a LINE1 insertion — the reference-genome-based framework may not capture long repeat expansion events
- When only short-read tumor/control data is available and full-length inserted sequences cannot be recovered, since tracing 3' transductions to source sites requires full-length insert sequence

## Inputs

- nanomonsv somatic SV/insertion calls (single-base resolution) from paired tumor and matched control long-read data
- Full-length inserted sequences of mobile element insertions extracted by nanomonsv
- Reference genome for mapping resolved source sites to coordinates

## Outputs

- Classified mobile element insertions annotated with 5' truncations, internal inversions, and LINE1 3' transduction source sites
- A reference list of LINE1 source element loci (e.g. 23 source elements) with genomic coordinates for downstream locus-level analyses
- Per-locus annotation records suitable as input to locus-wise methylation testing (e.g. methylation_pvalues.tsv rows)

## How to apply

Run the nanomonsv mobile element insertion classification workflow on somatic insertions detected with single-base resolution from tumor and matched control long-read data, using the full-length inserted sequences recovered by nanomonsv to characterize insertion properties. From the classified insertions, extract the source sites for 3' transduction mediated by LINE1 (the genomic elements that donated transduced sequence), together with in-depth properties such as 5' truncations and internal inversions that distinguish intact vs. rearranged source elements. Each resolved source site becomes a LINE1 source element locus (e.g. the 23 reference source elements in the study) that can be tabulated as a BED-like locus list and used as the input coordinate set for downstream analyses such as signal-level methylation testing at those loci. Rationale: full-length inserted sequence is required to trace a 3' transduction back to its source element, which short-read approaches cannot do reliably in repetitive regions.

## Related tools

- **nanomonsv** (Detects somatic insertions with single-base resolution from paired tumor/control long-read data and its mobile element insertion classification workflow resolves LINE1 3' transduction source sites from full-length inserted sequences) — https://github.com/friend1ws/nanomonsv
- **Oxford Nanopore Technologies (ONT)** (Long-read sequencing platform supplying the raw reads that enable full-length LINE1 insertion recovery and source-site resolution)
- **nanopolish** (Signal-level methylation calls (nanopolish index / call-methylation) evaluated at the annotated LINE1 source element loci in the downstream methylation analysis) — https://github.com/jts/nanopolish

## Examples

```
nanopolish index -d <fast5_dir> <sorted.bam>; nanopolish call-methylation -r <reads.fastq> -g <ref.fa> <sorted.bam> > methylation.tsv
```

## Evaluation signals

- Each classified 3' transduction has a resolvable source-site coordinate, and source elements are recovered consistently between tumor and matched control (source elements are germline loci; only the methylation/insertion activity differs)
- The number of independent LINE1 source element loci matches the reference set size used in the paper (23 source elements for the methylation analysis)
- Per-locus annotation includes the expected properties: 5' truncations, internal inversions, and target-site information for 3' transductions
- Downstream validation: at all 23 reference source elements, per-locus one-sided Fisher's exact tests plus harmonicmeanp combined P-values show significantly lower tumor methylation ratios (reproducing Figure 6A, Supplementary Figure S15)
- Breakpoints/source-site coordinates are single-nucleotide resolution rather than cluster-level calls

## Limitations

- Detection of subclonal MEIs present in a minority of cells is challenging at modest sequencing depth
- Interpretation of the detailed structure and properties of complex SVs (including transduction source assignments) is not fully automated and much of the work is done manually
- The Single breakend SV module cannot detect SVs where both breakpoints lie in poorly characterized highly repetitive regions
- The reference-genome-based framework may not capture long repeat expansion events

## Evidence

- [abstract] workflow to classify MEIs and resolve LINE1 source sites: "we have developed a workflow to classify mobile element insertions while elucidating their in-depth properties, such as 5' truncations, internal inversions, as well as source sites for 3' transduction"
- [intro] full-length inserted sequences enable characterization: "the full-length inserted sequences obtained by the nanomonsv allowed us to characterize their genetic properties"
- [abstract] single-base resolution from paired data: "nanomonsv, for detecting somatic structural variations (SVs) using tumor and matched control long-read sequencing data with a single-base resolution"
- [abstract] single breakend module for LINE1-mediated rearrangements: "Single breakend SV module enables the detection of complex SVs that can only be identified by long-reads, such as SVs involving highly-repetitive centromeric sequences, and LINE1- and virus-mediated"
- [readme] README description of nanomonsv purpose: "nanomonsv is a software for detecting somatic structural variations from paired (tumor and matched control) cancer genome sequence data"
