---
name: spades-pipeline-k-selection-consistency-checking
description: Use when benchmarking or configuring rnaSPAdes/SPAdes runs and you need
  to know whether k-mer sizes must be set manually or can be recomputed from read
  length for each RNA-Seq dataset.
when_to_use_negative:
- The assembler is plain SPAdes (not rnaSPAdes) run with its developer-chosen fixed
  default iterative k series (e.g., 21, 33, 55, 77), which is not derived from read
  length
- The user has explicitly supplied -k values on the command line, overriding automatic
  schedule computation
- The data are metatranscriptomic or long-read-only, which the rnaSPAdes approach
  was not designed or evaluated for
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: rnaSPAdes
  role: Assembler whose automatic read-length-derived k-mer schedule is being reconstructed
    and verified
  repo: github.com/ablab/spades
- name: SPAdes
  role: Baseline assembler with fixed developer-chosen default k-values used for contrast
    in the consistency check
  repo: github.com/ablab/spades
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giz100
    title: 'rnaSPAdes: a <i>de novo</i> transcriptome assembler and its application
      to RNA-Seq data'
  source_content_hash: sha256:59187a1861ba3c1efca6750ac7a9d214c2b48e922211684b1c3ba64851ed7097
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SPAdes
  - rnaSPAdes
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# SPAdes pipeline k-mer selection consistency checking

## Summary

Reconstructs and validates the automatic two-value k-mer schedule that rnaSPAdes derives from input read length (upper k = largest odd number ≤ read_length/2 - 1; lower k ≈ read_length/3 with a minimum of 29), distinguishing it from the fixed default k-mer series used by other assemblers.

## When to use

Use when benchmarking or configuring rnaSPAdes/SPAdes runs and you need to know whether k-mer sizes must be set manually or can be recomputed from read length for each RNA-Seq dataset.

## When NOT to use

- The assembler is plain SPAdes (not rnaSPAdes) run with its developer-chosen fixed default iterative k series (e.g., 21, 33, 55, 77), which is not derived from read length
- The user has explicitly supplied -k values on the command line, overriding automatic schedule computation
- The data are metatranscriptomic or long-read-only, which the rnaSPAdes approach was not designed or evaluated for

## Inputs

- Read lengths for each benchmark RNA-Seq dataset, obtained from SRA metadata (Human SRR5133163, Human large SRR1957703/SRR1957706, Mouse SRX648736, Worm SRR1560107, Corn SRR1588569, Arabidopsis SRR5344669/SRR5344670)
- Documented rnaSPAdes default k-value behavior per read length (for cross-checking)

## Outputs

- Per-dataset two-value k-mer schedule table (computed lower-k and upper-k for each read length)
- Pass/fail consistency verdict per dataset against the stated selection formula

## How to apply

Extract each dataset's read length from SRA metadata (e.g., Human SRR5133163, Human large SRR1957703/SRR1957706, Mouse SRX648736, Worm SRR1560107, Corn SRR1588569, Arabidopsis SRR5344669/SRR5344670). Compute the upper k as the largest odd number not exceeding read_length/2 - 1 (nearly half the read length), and the lower k as approximately read_length/3 with a floor of 29, since k-values below 29 dramatically increase erroneous contigs from transcripts sharing the same k-mer. Apply the rules to every dataset read length to obtain the two-value schedule rnaSPAdes uses instead of SPAdes' longer iterative series (e.g., 21, 33, 55, 77). Assemble the per-dataset lower-k and upper-k values into a summary table and cross-check each computed schedule against known rnaSPAdes default behavior for that read length, requiring no dataset to remain unmatched.

## Related tools

- **rnaSPAdes** (Assembler whose automatic read-length-derived k-mer schedule is being reconstructed and verified) — github.com/ablab/spades
- **SPAdes** (Baseline assembler with fixed developer-chosen default k-values used for contrast in the consistency check) — github.com/ablab/spades

## Examples

```
python -c "rl=150; kmax=rl//2-1; kmax=kmax if kmax%2 else kmax-1; kmin=max(29, rl//3); print(kmin, kmax)"
```

## Evaluation signals

- Every computed upper k is odd and does not exceed read_length/2 - 1 for its dataset
- Every computed lower k is at least 29 and approximately read_length/3
- Each computed schedule reproduces the rnaSPAdes default k-values for that read length, with no dataset left unmatched
- The computed schedule contains exactly two k values, contrasting with SPAdes' longer iterative series
- The summary table covers all benchmark datasets (Human, Human large, Mouse, Worm, Corn, Arabidopsis) without gaps

## Limitations

- k-values below 29 dramatically increase erroneous contigs from transcripts sharing the same k-mer, hence the floor of 29
- Small k-mer sizes have pitfalls in transcriptome assembly: SPAdes uses small k = 21 in its first iteration to assemble extremely low-covered genes where read overlaps are short, a regime where rnaSPAdes' automatic schedule can underperform
- The approach was designed for RNA-Seq from a single organism; metatranscriptome assembly (repetitive patterns in homologous genes, extreme mRNA quantity differences) is out of scope

## Evidence

- [methods] the key techniques that allows SPAdes to assemble contiguous genomic sequences from data with non-uniform coverage depth is the iterative de Bruijn graph construction: "the key techniques that allows SPAdes to assemble contiguous genomic sequences from data with non-uniform coverage depth is the iterative de Bruijn graph construction"
- [discussion] By using small k = 21 during the first iteration SPAdes manages to assemble extremely low-covered genes, where overlaps between reads are short.: "By using small k = 21 during the first iteration SPAdes manages to assemble extremely low-covered genes, where overlaps between reads are short."
- [readme] SPAdes package contains assembly pipelines for isolated and single-cell bacterial, as well as metagenomic and transcriptomic data.: "SPAdes package contains assembly pipelines for isolated and single-cell bacterial, as well as metagenomic and transcriptomic data."
