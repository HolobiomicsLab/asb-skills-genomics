---
name: suffix-array-construction-gsacak
description: Use when you need to extract all maximal exact matches (MEMs) shared
  across multiple ultra-long nucleotide sequences, e.g. to seed segmentation for an
  MSA pipeline on sequences reaching billions of bases.
when_to_use_negative:
- Input is a protein alignment or non-nucleotide sequence data — FMAlign2 Step 1 is
  designed for nucleotide sequences
- Sequences share only very short common substrings — MEMs shorter than the -l threshold
  (default 30) are discarded; the README advises lowering -l if too few MEMs are found,
  but if none exist above any reasonable threshold the MEM-finding module yields nothing
  useful
- You need global chains of common seeds rather than partial-chain MEM segmentation
  — that is FMAlign's approach, which FMAlign2 explicitly redefined
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: FMAlign2
  role: Host pipeline whose Step 1 uses the suffix array to identify MEMs; gSACA-K
    suffix/LCP construction and LCP-interval traversal replicate its MEMs-finding
    module
  repo: https://github.com/malabz/FMAlign2
- name: conda
  role: Recommended channel for installing FMAlign2 and its dependencies so the MEM-finding
    code can be run and validated
  repo: https://github.com/malabz/FMAlign2
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae014
    title: 'FMAlign2: a novel fast multiple nucleotide sequence alignment method for
      ultralong datasets'
  source_content_hash: sha256:1d7e833c10995b0bc418c67385774e611ad836f21a73d566bdc67ed7e30f3765
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/malabz/FMAlign2
  tools:
  - FMAlign2
  - conda
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# suffix-array-construction-gsacak

## Summary

Reconstructs the MEMs-finding module of FMAlign2 Step 1 by concatenating input nucleotide sequences with unique separators, building the suffix and LCP arrays with the linear-time gSACA-K algorithm, and traversing LCP intervals to emit maximal exact matches of length at least lmin (default 30).

## When to use

Use when you need to extract all maximal exact matches (MEMs) shared across multiple ultra-long nucleotide sequences, e.g. to seed segmentation for an MSA pipeline on sequences reaching billions of bases.

## When NOT to use

- Input is a protein alignment or non-nucleotide sequence data — FMAlign2 Step 1 is designed for nucleotide sequences
- Sequences share only very short common substrings — MEMs shorter than the -l threshold (default 30) are discarded; the README advises lowering -l if too few MEMs are found, but if none exist above any reasonable threshold the MEM-finding module yields nothing useful
- You need global chains of common seeds rather than partial-chain MEM segmentation — that is FMAlign's approach, which FMAlign2 explicitly redefined

## Inputs

- Nucleotide FASTA file with multiple input sequences (ultra-long sequences supported)
- Minimum MEM length threshold lmin (default 30, set via -l)
- Separator character verified absent from all input sequences

## Outputs

- List of maximal exact matches, each as a (k+1)-tuple: substring length l plus k starting positions in the concatenated sequence S
- mems.tsv with one MEM record per line (each record length >= lmin)

## How to apply

Load the nucleotide FASTA sequences (e.g. from the FMAlign2 Zenodo deposit 10435770) and validate that the chosen separator character does not occur in any sequence. Concatenate all k input sequences into a single string S using unique separators, following Supplementary Algorithm S1. Build the suffix array and LCP array of S with the gSACA-K linear-time algorithm, then traverse the LCP-interval tree with a two-pointer scan over the LCP array to identify exact matches shared across sequences. Left-extend each LCP interval to enforce left maximality and retain only MEMs with length >= 30 (FMAlign2's default -l parameter, adjustable). Emit each qualifying MEM as a (k+1)-tuple record (substring length l plus k starting positions in S), one MEM per output record.

## Related tools

- **FMAlign2** (Host pipeline whose Step 1 uses the suffix array to identify MEMs; gSACA-K suffix/LCP construction and LCP-interval traversal replicate its MEMs-finding module) — https://github.com/malabz/FMAlign2
- **conda** (Recommended channel for installing FMAlign2 and its dependencies so the MEM-finding code can be run and validated) — https://github.com/malabz/FMAlign2

## Examples

```
FMAlign2 -i /path/to/input.fasta -o /path/to/output.fasta -l 30
```

## Evaluation signals

- Every emitted MEM record is a (k+1)-tuple with substring length l >= 30 (the FMAlign2 default minimum MEM length)
- mems.tsv is non-empty for input sequence sets known to share common substrings
- Suffix and LCP array construction completes in linear time on the concatenated string S without errors from separator collisions (separator must not occur in any input sequence)
- Each reported MEM is left-maximal: left-extending the match breaks the exact match
- Cross-check against running the full pipeline: FMAlign2 -l 30 should segment sequences using these MEMs, and lowering -l below 30 should increase the number of emitted MEMs

## Limitations

- If MEM detection takes too long, the fast filtering mode (-f fast) runs faster but may find fewer MEMs, at a sensitivity cost
- If too few MEMs are found at the default threshold, the README advises lowering -l below 30
- The method depends on a separator character absent from all sequences; separator collisions would corrupt the concatenated string and downstream MEM positions

## Evidence

- [abstract] FMAlign2 leverages the suffix array to identify maximal exact matches: "FMAlign2 leverages the suffix array to identify maximal exact matches, redefining the approach of FMAlign from searching for global chains to partial chains."
- [abstract] Segmentation of sequences using suffix array-based maximal exact matches: [evidence span withheld — anchor status: not_found]
- [readme] -l <int> (default: 30). Minimum MEM length.: "* `-l <int>` (default: 30). Minimum MEM length."
- [readme] If too few MEMs are found, try lowering -l (e.g., below 30).: "* If too few MEMs are found, try lowering `-l` (e.g., below 30).
* If MEM detection takes too long, consider `-f fast`; it runs faster but may find fewer MEMs."
