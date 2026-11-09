---
name: assembly-quality-overcorrection-assessment
description: 'Use when a polished HiFi/T2T assembly must be validated against its
  draft: you need to confirm QV gains, reduction of homozygous high-quality variants,
  and absence of overcorrection or switch errors before accepting the polished consensus.'
when_to_use_negative:
- The assembly was produced from noisy long reads only without HiFi data and the goal
  is de novo error correction rather than validating a polish step
- No high-coverage short reads (>=60X) are available, since NextPolish2's error checking
  relies heavily on short-read k-mers and the QV/variant evaluation cannot be grounded
- The sample lacks a GIAB high-confidence benchmark set, making the homozygous-variant
  error count unreliable
edam_operation: http://edamontology.org/operation_3180
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0622
tools:
- name: NextPolish2
  role: Polishing tool whose output is being assessed for QV gains and overcorrection
  repo: https://github.com/Nextomics/NextPolish2
- name: yak
  role: Builds k=21/k=31 short-read k-mer datasets used for QV evaluation and NextPolish2
    error checking
  repo: https://github.com/lh3/yak
- name: Winnowmap
  role: Maps HiFi reads to the draft assembly using a repetitive k-mer list from meryl
  repo: https://github.com/marbl/Winnowmap
- name: minimap2
  role: Alternative HiFi read mapper to the draft assembly
  repo: https://github.com/lh3/minimap2/
- name: meryl
  role: Counts k=15 repetitive k-mers (distinct>0.9998) for winnowmap's -W option
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gpbjnl/qzad009
    title: 'NextPolish2: A Repeat-aware Polishing Tool for Genomes Assembled Using
      HiFi Long Reads'
  source_content_hash: sha256:d167b6322afdc31db13239aab681af1cee2be9a1c867ae41b34c230f03a8b524
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Nextomics/NextPolish2
  tools:
  - NextPolish2
  - Winnowmap
  - meryl
  - minimap2
  - yak
  edam_operation: http://edamontology.org/operation_3180
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# assembly-quality-overcorrection-assessment

## Summary

Assess whether polishing a HiFi-assembled genome (e.g., with NextPolish2) improved consensus accuracy without overcorrection or haplotype switch errors, using Merqury QV, DeepVariant variant counting against GIAB benchmarks, and switch-error evaluation.

## When to use

Use when a polished HiFi/T2T assembly must be validated against its draft: you need to confirm QV gains, reduction of homozygous high-quality variants, and absence of overcorrection or switch errors before accepting the polished consensus.

## When NOT to use

- The assembly was produced from noisy long reads only without HiFi data and the goal is de novo error correction rather than validating a polish step
- No high-coverage short reads (>=60X) are available, since NextPolish2's error checking relies heavily on short-read k-mers and the QV/variant evaluation cannot be grounded
- The sample lacks a GIAB high-confidence benchmark set, making the homozygous-variant error count unreliable

## Inputs

- Draft HiFi-assembled genome (e.g., HiFiasm primary assembly, FASTA/FASTA.GZ)
- Polished assembly output (e.g., asm.np2.fa from nextPolish2)
- PacBio HiFi read set aligned to the draft (hifi.map.sort.bam)
- High-coverage (>=60X) short reads for yak k-mer datasets (k21.yak, k31.yak)
- GIAB high-confidence benchmarking variant set for the sample (e.g., HG005)
- GRCh37 reference for variant calling comparison

## Outputs

- Merqury QV values for draft and polished assemblies
- Count of homozygous high-quality variants (potential errors) for draft vs. polished assembly
- Switch-error counts in heterozygous regions (Paftools.js)
- Pass/fail judgment that polishing improved consensus without overcorrection

## How to apply

Run the polishing pipeline (winnowmap/meryl-based mapping, yak k=21/k=31 datasets, nextPolish2) then evaluate the draft and polished assemblies in parallel with three complementary approaches: (1) Merqury to compute the k-mer-based quality value (QV); (2) DeepVariant/GIAB comparison to count homozygous high-quality variants against GRCh37 that fall outside the high-confidence GIAB benchmark set — these are potential errors, so polishing should reduce them (HG005: 14,155 → 6,955; QV 53.8544 → 55.6257); (3) Paftools.js to evaluate haplotype switch errors in heterozygous regions. Judge success when QV increases, homozygous high-quality variants decrease substantially, and switch errors are not increased (NextPolish2's phasing module should maintain or slightly reduce them), confirming no overcorrection even in repetitive regions.

## Related tools

- **NextPolish2** (Polishing tool whose output is being assessed for QV gains and overcorrection) — https://github.com/Nextomics/NextPolish2
- **yak** (Builds k=21/k=31 short-read k-mer datasets used for QV evaluation and NextPolish2 error checking) — https://github.com/lh3/yak
- **Winnowmap** (Maps HiFi reads to the draft assembly using a repetitive k-mer list from meryl) — https://github.com/marbl/Winnowmap
- **minimap2** (Alternative HiFi read mapper to the draft assembly) — https://github.com/lh3/minimap2/
- **meryl** (Counts k=15 repetitive k-mers (distinct>0.9998) for winnowmap's -W option)

## Examples

```
./target/release/nextPolish2 -t 5 hifi.map.sort.bam asm.fa.gz k21.yak k31.yak > asm.np2.fa
```

## Evaluation signals

- Merqury QV of the polished assembly exceeds the draft (HG005 reference result: 53.8544 → 55.6257)
- Homozygous high-quality variants (GRCh37 vs. sample, outside GIAB high-confidence regions) decrease substantially (HG005: 14,155 → 6,955)
- Switch errors in heterozygous regions do not increase; the built-in phasing module should maintain or slightly reduce them
- No excess of new variants relative to the reference that would indicate overcorrection, even in repetitive/segmental-duplication regions
- Polished output (asm.np2.fa) is a complete assembly of the same length/content as the draft, with only base-level (SNV/Indel) corrections

## Limitations

- Performance relies heavily on short reads to check whether a K-mer contains errors; biases inherent in short-read sequencing can leave uncorrected errors, and no specific coverage threshold beyond the >=60X recommendation is given
- No software version numbers are stated for NextPolish2 or its dependencies, and no changelog is available, complicating reproducibility
- The homozygous-variant error count depends on availability of GIAB high-confidence benchmark variants for the sample

## Evidence

- [methods] NextPolish2 improved the QV of the assembled genome from 53.8544 to 55.6257 and reduced the number of homozygous high-quality variants (potential errors) from 14,155 to 6,955.: "NextPolish2` improved the QV of the assembled genome from 53.8544 to 55.6257 and reduced the number of homozygous high-quality variants (potential errors) from 14,155 to 6,955."
- [methods] Three evaluation approaches: Merqury QV, DeepVariant, and Paftools.js.: "we employed three approaches to evaluate the genome's accuracy before and after polishing:

1. **Merqury:** To assess the quality value (QV)."
- [discussion] Performance relies heavily on short reads to check whether a K-mer contains errors.: "The performance of NextPolish2 relies heavily on short reads to check whether a K-mer contains errors."
- [intro] Built-in phasing module maintains haplotype consistency and slightly reduces switch errors.: "Through the built-in phasing module, it can not only correct the error bases, but also maintain the original haplotype consistency. In fact, our evaluation shows that it even slightly reduces switch"
- [readme] Short-read QC with fastp and >=60X short-read recommendation for yak datasets.: "# Quality control and filtering.
# fastp -5 -3 -n 0 -f 5 -F 5 -t 5 -T 5 -q 20"
