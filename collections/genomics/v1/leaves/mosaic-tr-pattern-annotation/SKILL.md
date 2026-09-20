---
name: mosaic-tr-pattern-annotation
description: Use when long-read (e.g. PacBio HiFi) sequences are suspected to contain
  mosaic tandem repeats with multiple repeat units that must be annotated with an
  exact decomposition pattern and mismatch statistics.
when_to_use_negative:
- When the analysis goal is counting sample variants or haplotypes rather than decomposing
  the pattern — use cTR, which clusters reads with a maximum Levenshtein distance
  of 0.01 times average read length
- When precise per-unit variant strings are needed downstream of an already-estimated
  pattern — use vTR, which outputs variant units instead of the concise pattern
- When input strings are short or contain no tandem repeat structure; -r will suppress
  output entirely if the mismatch ratio exceeds the default threshold of 0.3
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0080
tools:
- name: uTR
  role: 'Primary decomposition/annotation tool: decomposes the DNA string into a mosaic
    tandem repeat pattern with different repeat units and annotates the output FASTA'
  repo: https://github.com/morisUtokyo/uTR
- name: mTR
  role: Pre-step for computing long repeat units (>20 nt) that uTR can then use via
    -u
  repo: https://github.com/morisUtokyo/mTR
- name: cTR
  role: Optional pre-clustering of reads into groups of similar reads to generate
    smaller input FASTA files and reduce computation time
  repo: https://github.com/morisUtokyo/cTR
- name: vTR
  role: 'Downstream step that takes the estimated #Pat pattern plus input string and
    outputs a series of variant units for precise mismatch inspection'
  repo: https://github.com/morisUtokyo/vTR
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad185
    title: Decomposing mosaic tandem repeats accurately from long reads
  source_content_hash: sha256:8fb730f847363a39e8028aa1b238a61c04499bf347113175d92e449c38949132
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/morisUtokyo/uTR
  tools:
  - cTR
  - mTR
  - uTR
  - vTR
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# mosaic-tr-pattern-annotation

## Summary

Annotates DNA strings from long reads with mosaic tandem repeat (TR) patterns composed of different repeat units using uTR, which decomposes each read via optimal partial matching against inferred unit strings. It outputs annotations like '#Pat <CAG>19<CCG>38' with dissimilarity information.

## When to use

Use when long-read (e.g. PacBio HiFi) sequences are suspected to contain mosaic tandem repeats with multiple repeat units that must be annotated with an exact decomposition pattern and mismatch statistics.

## When NOT to use

- When the analysis goal is counting sample variants or haplotypes rather than decomposing the pattern — use cTR, which clusters reads with a maximum Levenshtein distance of 0.01 times average read length
- When precise per-unit variant strings are needed downstream of an already-estimated pattern — use vTR, which outputs variant units instead of the concise pattern
- When input strings are short or contain no tandem repeat structure; -r will suppress output entirely if the mismatch ratio exceeds the default threshold of 0.3

## Inputs

- FASTA file of long-read DNA sequences (e.g. realdata/realdata.fasta)
- Optional locus information string such as chr8:118366813-118366928
- Optional single representative unit string of a putative tandem repeat
- Optional pre-computed long repeat units (>20 nt) from mTR

## Outputs

- FASTA file annotated with tandem repeat patterns, e.g. '#Info (171,125,0.129) #Pat <CAG>19<CCG>38 #Decomp [2 (0,CCG,3,114,114) (1,AGC,3,57,57)]'
- Optional #Locus tag per record when locus information is supplied
- Dissimilarity ratio between the tandem repeat pattern and the input string per read

## How to apply

Feed the input FASTA with uTR via -f and obtain the annotated FASTA via -o; optionally supply locus information (-l, e.g. chr8:118366813-118366928) and a representative unit string (-u), since uTR otherwise auto-estimates units of 20 nt or less (longer units should first be computed with mTR). The default mode outputs the simpler pattern with more mismatches; use -y to prefer more complex patterns with fewer mismatches (e.g. 12.9% vs 4.1% dissimilarity) and -z to prioritize longer units by optimizing alignment score minus the number of units. Apply the -r filter to reject decompositions whose mismatch ratio between the representative unit and the tandem repeat exceeds a threshold (default 0.3, set by MAX_DIS_RATIO_DEFAULT in uTR.h). Verify output lines contain '#Info' (length, haplotypes, dissimilarity ratio), '#Pat', and '#Decomp' tags, and that unit rotation conventions make the first pattern unit's character match the read's first character.

## Related tools

- **uTR** (Primary decomposition/annotation tool: decomposes the DNA string into a mosaic tandem repeat pattern with different repeat units and annotates the output FASTA) — https://github.com/morisUtokyo/uTR
- **mTR** (Pre-step for computing long repeat units (>20 nt) that uTR can then use via -u) — https://github.com/morisUtokyo/mTR
- **cTR** (Optional pre-clustering of reads into groups of similar reads to generate smaller input FASTA files and reduce computation time) — https://github.com/morisUtokyo/cTR
- **vTR** (Downstream step that takes the estimated #Pat pattern plus input string and outputs a series of variant units for precise mismatch inspection) — https://github.com/morisUtokyo/vTR

## Examples

```
uTR -f realdata/realdata.fasta -o annotated.fasta -l chr8:118366813-118366928 -r 0.3
```

## Evaluation signals

- Output annotation contains the required tags in order: #Info (length, haplotype count, dissimilarity ratio), #Pat, and #Decomp
- Dissimilarity ratio in #Info is at or below the -r threshold (default 0.3); no tandem repeat is output if it exceeds this threshold
- Unit rotation check: the first character of the output pattern's first unit matches the first character of the input string
- Benchmark comparison: in simulated data, uTR correctly estimated patterns in 1000/1000 (2-20 range) and 998/1000 (2-500 range) cases and outperforms TRF and RepeatMasker
- Mode sanity checks: -y lowers dissimilarity versus the default (e.g. 0.041 vs 0.129 on the sample CAG/CCG read), and -z output contains at least one longer unit such as <CCGCAG>

## Limitations

- If a representative unit string is not supplied with -u, uTR automatically estimates tandem repeat units only of length 20 nt or less; longer units require mTR as a pre-step
- The default mode trades complexity for simplicity: it outputs a simpler pattern with more mismatches (12.9% on the sample read) unless -y or -z is used
- uTR's version number is not stated in the paper or README
- Wide discrepancies in pattern length affect accuracy: on simulated (AC)m(AG)n data, prediction dropped from 1000/1000 to 998/1000 as average pattern length grew to 1011

## Evidence

- [intro] The program uTR decomposes a DNA string into mosaic tandem repeats with different repeat units.: "The program uTR decomposes a DNA string into mosaic tandem repeats with different repeat units."
- [abstract] our program named uTR outperforms TRF and RepeatMasker in terms of prediction accuracy: "our program named uTR outperforms TRF and RepeatMasker in terms of prediction accuracy"
- [methods] uTR [-f input fasta file] [-o output fasta file with annotation] [-l locus information] [-u input representative unit string]: "uTR [-f input fasta file] [-o output fasta file with annotation] [-l locus information] [-u input representative unit string]"
- [readme] the annotation contains "#Pat \<CAG\>19\<CCG\>38", which shows a tandem repeat pattern: "the annotation contains "#Pat \<CAG\>19\<CCG\>38", which shows a tandem repeat pattern, and "#Info (171,125,0.129)""
- [readme] -r : Give a maximum threshold on the mismatch ratio between the representaitve unit and a tandem repeat of the unit.: "-r : Give a maximum threshold on the mismatch ratio between the representaitve unit and a tandem repeat of the unit."
- [readme] "-z" prioritizes a decomposition with longer units, say \<AGAGAGCTCT\>3 in the above example: ""-z" prioritizes a decomposition with longer units, say \<AGAGAGCTCT\>3 in the above example."
