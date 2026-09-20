---
name: l1-regression-line-fitting
description: Use when chaining sequence-alignment anchors (e.g., k-mer LCSk chains
  from noisy long nanopore reads) and you need to remove colinear outliers before
  scoring candidate read placements against a reference genome.
when_to_use_negative:
- Input is raw sequencing reads with no pre-computed seed anchors — L1 line fitting
  is Stage IV, downstream of gapped spaced-seed search and graph-based anchoring
- Reads are short and low-error (e.g., Illumina), where a full five-stage funneling
  approach is unnecessary — GraphMap's default parameters target long, error-prone
  third-generation reads
- Memory-constrained environments for large references — GraphMap's hash index needs
  ~14x reference size (~100 GB peak for human genome), and FM-index reduction is 'currently
  under implementation'
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: GraphMap
  role: Implements the five-stage read-funneling mapper in which L1 regression line
    fitting is Stage IV, following LCSk anchor chaining (Stage III)
  repo: https://github.com/isovic/graphmap
- name: GCC
  role: Compiler toolchain used to build GraphMap from source with make
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/ncomms11307
    title: Fast and sensitive mapping of nanopore sequencing reads with GraphMap
  source_content_hash: sha256:97f4968da655e603af497946f19785b2f502ac8adc01090a704a5e63007fe648
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/isovic/graphmap
  tools:
  - GCC
  - GraphMap
  - make
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# l1-regression-line-fitting

## Summary

GraphMap's Stage IV candidate refinement: fit a 45-degree line to alignment anchors by L1 (least absolute deviation) regression, discard outlier anchors outside a ±3-sigma confidence band, and re-run LCSk chaining on the retained anchors to yield cleaner mapping paths.

## When to use

Use when chaining sequence-alignment anchors (e.g., k-mer LCSk chains from noisy long nanopore reads) and you need to remove colinear outliers before scoring candidate read placements against a reference genome.

## When NOT to use

- Input is raw sequencing reads with no pre-computed seed anchors — L1 line fitting is Stage IV, downstream of gapped spaced-seed search and graph-based anchoring
- Reads are short and low-error (e.g., Illumina), where a full five-stage funneling approach is unnecessary — GraphMap's default parameters target long, error-prone third-generation reads
- Memory-constrained environments for large references — GraphMap's hash index needs ~14x reference size (~100 GB peak for human genome), and FM-index reduction is 'currently under implementation'

## Inputs

- Stage II/III alignment anchor hits (k-mer seed matches) for each nanopore read, from GraphMap's read-funneling pipeline
- Reference genome sequence (e.g., FASTA) the reads are being mapped to

## Outputs

- Refined, colinear anchor chains with outlier anchors removed (residuals within ±3-sigma of the fitted 45-degree line)
- Candidate mapping regions scored with Eqs. 4–7 and ranked for downstream Myers bit-vector alignment

## How to apply

Take the anchor hits produced by GraphMap's Stage II/III for each nanopore read and, for each candidate mapping path, fit a 45-degree diagonal line through the anchor points using L1 (least absolute deviation) regression rather than ordinary least squares, which makes the fit robust to the high-error-rate outliers typical of MinION reads. Compute anchor residuals from the fitted line, estimate a sigma from those residuals, and discard anchors whose residuals fall outside the ±3-sigma confidence band. Re-run the variable-length LCSk (longest common subsequence of k-mers) chaining on the retained anchors to rebuild the path. The resulting refined candidate regions are then scored with GraphMap's region quality scoring formulae (Eqs. 4–7) before final Myers bit-vector alignment. Rationale: progressive refinement — gapped spaced seeds (Stage I), graph-based anchoring (Stage II), LCSk chaining (Stage III), L1 line fitting (Stage IV) — is what lets GraphMap robustly handle high error rates while attaining >95% precision.

## Related tools

- **GraphMap** (Implements the five-stage read-funneling mapper in which L1 regression line fitting is Stage IV, following LCSk anchor chaining (Stage III)) — https://github.com/isovic/graphmap
- **GCC** (Compiler toolchain used to build GraphMap from source with make)

## Examples

```
git clone https://github.com/isovic/graphmap.git; cd graphmap; make modules; make; ./bin/Linux-x64/graphmap align -r reference.fa -d reads.fasta -o output.sam
```

## Evaluation signals

- Residual distribution after L1 fit: retained anchors should lie within the ±3-sigma confidence band of the 45-degree line; outlier count before/after should shrink
- Final SAM output (graphmap_alignments.sam) parses correctly and carries meaningful mapping quality and E-value (ZE:f tag) annotations
- Mapping location precision of the progressive-refinement pipeline exceeds the reported >95% (typically >98%) on processed reads
- GraphMap distinguishes candidate locations that are >94% identical on the human genome after refinement
- Sensitivity check: >90% of reads and ~95% of bases mapped on MinION data, a 10–80% increase in mapped bases versus other mappers

## Limitations

- GraphMap requires large-memory machines for big references (~100 GB for the human genome); the memory index is ~14x reference size, and FM-index reduction was only 'currently under implementation'
- Percentile-based seed filtering (--freq-percentile, default 0.99) slightly reduces sensitivity, though it is critical when mapping to large references
- Index incompatibility halts execution by default in v0.5.1+; use --auto-rebuild-index or --rebuild-index to rebuild deliberately

## Evidence

- [readme] GraphMap then chains anchors using a kmer version of longest common subsequence (LCSk) construction (stage III): [evidence span withheld — anchor status: external_doc]
- [methods] fit a 45-degree line by L1 (least absolute deviation) regression, discard anchors whose residuals fall outside the ±3-sigma confidence band: [evidence span withheld — anchor status: not_found]
- [abstract] progressively refines candidate alignments to robustly handle potentially high-error rates and a fast graph traversal: "progressively refines candidate alignments to robustly handle potentially high-error rates and a fast graph traversal to align long reads with speed and high precision (>95%)"
- [discussion] GraphMap's precision is typically >98% and it is able to distinguish between candidate locations that are >94% identical on the human genome: "GraphMap's precision is typically >98% and it is able to distinguish between candidate locations that are >94% identical on the human genome"
