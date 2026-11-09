---
name: genome-assembly-quality-comparison
description: Use when you have multiple assembly versions (e.g. uncorrected long-read,
  consensus-corrected, polished, and an Illumina draft) and need to quantify sequence-composition
  convergence or expose platform-specific k-mer biases.
when_to_use_negative:
- You need positional or structural accuracy metrics (N50, misassemblies, optical-map
  agreement) rather than sequence-composition comparison — this skill compares global
  6-mer frequencies only and cannot judge scaffold contiguity or structural correctness.
- 'You need reference-guided base-level validation: the article notes that ''In the
  absence of a high-quality reference, it is not straightforward to establish which
  assembly is correct'' at individual error sites; k-mer composition can only reveal
  systematic biases, not site-specific correctness.'
- The assemblies differ so much in size or coverage that normalized 6-mer frequencies
  are no longer comparable (sparse profiles from very low-coverage assemblies will
  dominate correlation results).
edam_operation: http://edamontology.org/operation_0286
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: Jellyfish
  role: Counts canonical 6-mer histograms for each assembly FASTA; counts are then
    normalized to relative frequencies before correlation.
- name: Racon
  role: Produces one of the comparison stages — the Racon-corrected nanopore scaffold
    set whose 6-mer profile is compared against the draft.
  repo: https://github.com/isovic/racon
- name: pilon
  role: Produces the final polished scaffold set (Illumina-based polishing) whose
    6-mer profile should correlate near-unity with the Illumina draft AZBK01.
  repo: https://github.com/broadinstitute/pilon/wiki
- name: TULIP
  role: Produces the uncorrected long-read scaffold assembly whose 6-mer composition
    is the baseline for measuring improvement from correction.
  repo: https://github.com/Generade-nl/TULIP
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-017-07650-6
    title: Rapid de novo assembly of the European eel genome from nanopore sequencing
      reads
  source_content_hash: sha256:4b2b02a4e2b7ed9c4e65a4ecf873cd93b084a2a4774b7c4cc6424529755ba2f8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/isovic/racon
  tools:
  - Jellyfish
  - Racon
  - TULIP
  - pilon
  edam_operation: http://edamontology.org/operation_0286
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# genome-assembly-quality-comparison

## Summary

Quantifies how correction stages (TULIP → Racon → Pilon) move a long-read assembly toward a reference-quality draft by comparing canonical 6-mer frequency profiles with Jellyfish and pairwise Pearson correlations, revealing platform-specific biases such as homopolymer underrepresentation in nanopore data and high-CG underrepresentation in Illumina data.

## When to use

Use when you have multiple assembly versions (e.g. uncorrected long-read, consensus-corrected, polished, and an Illumina draft) and need to quantify sequence-composition convergence or expose platform-specific k-mer biases.

## When NOT to use

- You need positional or structural accuracy metrics (N50, misassemblies, optical-map agreement) rather than sequence-composition comparison — this skill compares global 6-mer frequencies only and cannot judge scaffold contiguity or structural correctness.
- You need reference-guided base-level validation: the article notes that 'In the absence of a high-quality reference, it is not straightforward to establish which assembly is correct' at individual error sites; k-mer composition can only reveal systematic biases, not site-specific correctness.
- The assemblies differ so much in size or coverage that normalized 6-mer frequencies are no longer comparable (sparse profiles from very low-coverage assemblies will dominate correlation results).

## Inputs

- Assembly FASTA files at each correction stage: uncorrected TULIP scaffold FASTA, Racon-corrected scaffolds, Pilon-polished scaffolds
- Reference-quality comparison draft assembly in FASTA (e.g. NCBI WGS accession AZBK01, Illumina-based)
- Jellyfish (k-mer counting, k=6 canonical)

## Outputs

- fig6_kmer_correlation_matrix.csv — 4x4 Pearson correlation matrix over normalized 6-mer frequency profiles
- fig6_kmer_correlation_heatmap.png — heatmap rendering of the matrix including self-correlations on the diagonal
- Interpretation of platform-specific biases: homopolymer underrepresentation (nanopore assemblies) and high-CG underrepresentation (Illumina-based assemblies)

## How to apply

Run Jellyfish with k=6 on each assembly — here the NCBI WGS Illumina draft AZBK01, the uncorrected TULIP scaffolds, the Racon-corrected scaffolds, and the Pilon-polished scaffolds — to obtain canonical 6-mer count histograms. Normalize each histogram to relative frequencies so assemblies of different total sizes are comparable. Compute the full 4x4 pairwise Pearson correlation matrix over the 6-mer frequency profiles (pandas/scipy), write it as fig6_kmer_correlation_matrix.csv, and render it as a heatmap with matplotlib/seaborn (fig6_kmer_correlation_heatmap.png) including the diagonal. Interpret the off-diagonal structure: near-unity correlation between the Pilon assembly and the draft indicates that Racon-then-Pilon correction removed most nanopore-specific sequence error, while systematic k-mer deficits identify homopolymer underrepresentation in TULIP/Racon and slight high-CG underrepresentation in the Illumina-based draft and Pilon sequences.

## Related tools

- **Jellyfish** (Counts canonical 6-mer histograms for each assembly FASTA; counts are then normalized to relative frequencies before correlation.)
- **Racon** (Produces one of the comparison stages — the Racon-corrected nanopore scaffold set whose 6-mer profile is compared against the draft.) — https://github.com/isovic/racon
- **pilon** (Produces the final polished scaffold set (Illumina-based polishing) whose 6-mer profile should correlate near-unity with the Illumina draft AZBK01.) — https://github.com/broadinstitute/pilon/wiki
- **TULIP** (Produces the uncorrected long-read scaffold assembly whose 6-mer composition is the baseline for measuring improvement from correction.) — https://github.com/Generade-nl/TULIP

## Examples

```
jellyfish count -k 6 -C -s 100M -t 4 assembly.fasta; python -c "import pandas, scipy.stats; r = scipy.stats.pearsonr(freqs_a, freqs_b)"
```

## Evaluation signals

- The recovered correlation matrix must reproduce the qualitative Fig. 6 pattern: highest correlations in the Racon-vs-Pilon and Pilon-vs-draft comparisons, with the Pilon assembly near-unity against the Illumina draft AZBK01.
- Reduced correlation must appear for the uncorrected TULIP and Racon assemblies, attributable to homopolymer underrepresentation in nanopore-derived sequence.
- A slight but detectable deficit of high-CG 6-mers should appear in the Illumina-based draft and Pilon profiles.
- Diagonal of the correlation matrix must be exactly 1 (self-correlations) and the matrix must be symmetric — a sanity check that normalization and pairing were done correctly.
- 6-mer frequency profiles must sum to 1 for each assembly after normalization, ensuring assemblies of different total sizes (e.g. the ~860 Mbp nanopore assembly vs the 969 Mbp draft) are directly comparable.

## Limitations

- k-mer composition similarity does not establish base-level correctness: 'In the absence of a high-quality reference, it is not straightforward to establish which assembly is correct' at individual error sites.
- The comparison assemblies derive from a hybrid of two sequencing technologies, three nanopore chemistries, and two different animals ('in theory this generates a consensus between up to four different haplotypes'), which confounds interpretation of composition differences.
- Genome-size discrepancies between short-read and nanopore assemblies ('This can be explained either by the absence of a substantial fraction of the genome from the nanopore data or assembly, or by an artificially inflated genome size for the short-read assemblies') mean frequency-normalized comparison may be confounded by differing genomic completeness.
- 6-mer comparison reveals only systematic biases (homopolymers, high-CG content); it cannot detect structural errors such as missing repetitive or (sub)telomeric regions.

## Evidence

- [results] The correction steps bring the sequence similarity of the nanopore-based assembly closer to the Illumina-based draft, with the final corrected assembly having a high correlation to the draft: [evidence span withheld — anchor status: not_found]
- [results] 6-mer frequencies also show homopolymer underrepresentation in TULIP/Racon and slight high-CG underrepresentation in the Illumina-based draft and Pilon sequence: [evidence span withheld — anchor status: not_found]
- [discussion] In the absence of a high-quality reference, it is not straightforward to establish which assembly is correct: "In the absence of a high-quality reference, it is not straightforward to establish which assembly is correct"
- [discussion] The genome assembly generated here is a hybrid, incorporating two different sequencing technologies, three generations of nanopore sequencing, and two different animals: "The genome assembly generated here is a hybrid, incorporating two different sequencing technologies, three generations of nanopore sequencing, and two different animals"
- [discussion] This can be explained either by the absence of a substantial fraction of the genome from the nanopore data or assembly, or by an artificially inflated genome size for the short-read assemblies: "This can be explained either by the absence of a substantial fraction of the genome from the nanopore data or assembly, or by an artificially inflated genome size for the short-read assemblies"
