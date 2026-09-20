---
name: poa-consensus-sequence-generation
description: Use when you have haplotype-resolved long-read BAM alignments (ONT or
  PacBio) over candidate repetitive regions and need low-error haplotype-specific
  consensus sequences before repeat motif calling.
when_to_use_negative:
- Input reads are short Illumina reads with low error rates, where POA-based error
  correction is unnecessary
- Candidate regions lack sufficient spanning read support in the haplotype-resolved
  BAM, yielding unreliable consensus
- Repeats lie in unassembled regions such as centromeres or telomeres, which TRiCoLOR
  cannot identify
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: TRiCoLOR REFER
  role: Module that fetches spanning reads and invokes POA consensus generation before
    motif detection
  repo: https://github.com/davidebolo1993/TRiCoLOR/tree/master/TRiCoLOR
- name: SPOA
  role: Partial order alignment engine used to compute haplotype-specific low-error
    consensus sequences
- name: minimap2
  role: Realigns generated consensus sequences to GRCh38 for repeat multiplicity/period
    resolution
- name: Docker (davidebolo1993/tricolor)
  role: Containerized distribution of TRiCoLOR including the SPOA-based REFER pipeline
  repo: https://hub.docker.com/r/davidebolo1993/tricolor
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giaa101
    title: 'TRiCoLOR: tandem repeat profiling using whole-genome long-read sequencing
      data'
  source_content_hash: sha256:29266b09889bbe046f9373c4da75a0c96e0d20cf7567bc46971ab82664d9408c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Docker
  - Docker (davidebolo1993/tricolor)
  - SPOA
  - TRiCoLOR
  - TRiCoLOR REFER
  - TRiCoLOR SENSoR
  - minimap2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 19
---

# POA consensus sequence generation

## Summary

Builds haplotype-specific, low-error consensus sequences from error-prone ONT/PacBio long reads spanning candidate tandem repeat regions using partial order alignment (POA), as implemented by SPOA inside the TRiCoLOR REFER module. The consensus step approximates error reduction of ~77% for ONT and ~88% for PB reads, making motif detection by the downstream RegEx string matcher feasible.

## When to use

Use when you have haplotype-resolved long-read BAM alignments (ONT or PacBio) over candidate repetitive regions and need low-error haplotype-specific consensus sequences before repeat motif calling.

## When NOT to use

- Input reads are short Illumina reads with low error rates, where POA-based error correction is unnecessary
- Candidate regions lack sufficient spanning read support in the haplotype-resolved BAM, yielding unreliable consensus
- Repeats lie in unassembled regions such as centromeres or telomeres, which TRiCoLOR cannot identify

## Inputs

- Haplotype-resolved long-read BAM alignments (ONT or PacBio) over candidate TR regions
- BED file of candidate low-entropy repetitive regions from TRiCoLOR SENSoR
- Human GRCh38 reference genome (for subsequent realignment of consensus sequences)

## Outputs

- Haplotype-specific low-error consensus FASTA sequences per candidate region
- minimap2 realignment of consensus sequences to GRCh38 feeding motif detection

## How to apply

Within TRiCoLOR REFER, first fetch and trim reads spanning each candidate low-entropy region identified by SENSoR on the haplotype-resolved BAM. Run SPOA (partial order alignment) on the fetched read pile to compute a haplotype-specific consensus sequence, which yields an approximate error reduction of ~77% for ONT and ~88% for PB reads. The rationale is that raw ONT/PB error rates (~10-15%) prevent direct motif detection, so POA consensus generation must precede the RegEx-based approximate string matcher (motif length ≤6, ≥5 perfect repeats, edit distance ≤1). Finally, realign the consensus sequences to GRCh38 with minimap2 so repeat multiplicity and period size can be resolved haplotype-specifically. Expect the consensus step to be parameter-light; correctness hinges on the quality of the upstream region candidates and read coverage.

## Related tools

- **TRiCoLOR REFER** (Module that fetches spanning reads and invokes POA consensus generation before motif detection) — https://github.com/davidebolo1993/TRiCoLOR/tree/master/TRiCoLOR
- **SPOA** (Partial order alignment engine used to compute haplotype-specific low-error consensus sequences)
- **minimap2** (Realigns generated consensus sequences to GRCh38 for repeat multiplicity/period resolution)
- **Docker (davidebolo1993/tricolor)** (Containerized distribution of TRiCoLOR including the SPOA-based REFER pipeline) — https://hub.docker.com/r/davidebolo1993/tricolor

## Examples

```
docker pull davidebolo1993/tricolor
```

## Evaluation signals

- Consensus error reduction matches reported magnitudes (~77% for ONT, ~88% for PB consensus sequences)
- Each consensus sequence corresponds to exactly one haplotype-specific candidate region from the SENSoR BED output
- Realignments of consensus to GRCh38 with minimap2 map to the original region coordinates
- Downstream RegEx motif detection (motif length ≤6, ≥5 perfect repeats, edit distance ≤1) succeeds on consensus sequences that fail on raw reads
- Output feeds REFER's BCF-compliant storage of TRs ≥50 bp without unmapped or dropped candidates

## Limitations

- Approximate error reduction figures (~77% ONT, ~88% PB) are empirical estimates; residual errors still require the edit distance ≤1 approximate matching stage
- Consensus quality degrades in low-coverage regions — the upstream filtering step removes regions with mean coverage ≤8 (ONT) or ≤10x (PB) before REFER runs
- No support for haploid chromosomes (e.g., human Y) or polyploid species, so consensus is strictly haplotype-specific diploid logic

## Evidence

- [intro] POA rationale: "TRiCoLOR exploits partial order alignment (POA) [14] to compute haplotype-specific low-error consensus sequences"
- [methods] Error reduction figures: "to compute highly accurate consensus sequences with an approximate error reduction of ~77% and ~88% for ONT and PB, respectively"
- [methods] REFER module role: "TRiCoLOR can profile TRs in haplotype-resolved BAM files through the REFER (REpeats FindER) module"
- [methods] Downstream motif detection dependency: "looking for approximate repetitions of the identified motifs to account for remaining consensus errors, i.e., imperfect repeated motifs up to a user-defined edit distance (≤1, by default)"
- [readme] Docker availability: "TRiCoLOR is available to be installed from source or as a [Docker container](https://hub.docker.com/r/davidebolo1993/tricolor)."
