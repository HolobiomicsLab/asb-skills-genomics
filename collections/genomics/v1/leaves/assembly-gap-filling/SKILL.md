---
name: assembly-gap-filling
description: Use when a draft long-read assembly (e.g. FASTA with N-gaps from Flye)
  should be scaffolded with long reads and have its Ns replaced by spanning read sequence,
  to improve N50 and gap-free contiguity.
when_to_use_negative:
- The input assembly contains no N-gaps and only single-round scaffolding is needed
  — use ntLink scaffolding with overlap detection (Basic Protocol 1) instead of gap-filling
  rounds.
- The goal is misassembly correction rather than gap closure — use ntLink mappings
  with Tigmint-long (Alternate Protocol 2) instead.
- No long-read data spanning the gaps is available, so gap-filling with raw read sequence
  cannot proceed.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: ntLink
  role: Performs minimizer-based long-read scaffolding with gap-filling and in-code
    scaffolding iterations (run_rounds_gaps Makefile target)
  repo: https://github.com/bcgsc/ntLink
- name: ABySS
  role: Provides abyss-fac (v2.3.0+) to compute N50 and total scaffold length for
    each round's output
  repo: https://github.com/BirolLab/abyss
- name: QUAST
  role: Evaluates assemblies in --split-scaffold mode, breaking scaffolds at Ns for
    misassembly-aware contiguity comparison
  repo: https://github.com/ablab/quast
- name: minimap2
  role: Supports mapping liftover to reduce computational cost of subsequent scaffolding
    rounds
  repo: https://github.com/lh3/minimap2
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/cpz1.733
    title: 'ntLink: A Toolkit for <i>De Novo</i> Genome Assembly Scaffolding and Mapping
      Using Long Reads'
  source_content_hash: sha256:da45aa9050f278ac3d2585479130756df0ae9145c70a69f59448331e3e09c68d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bcgsc/ntLink
  tools:
  - ABySS
  - QUAST
  - minimap2
  - ntLink
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# assembly-gap-filling

## Summary

Fill N-gaps in a long-read-draft genome assembly during ntLink scaffolding, using raw long-read sequence spanning the gaps; demonstrated with ONT reads (SRR10028109) scaffolded on a Flye C. elegans assembly. Gap-filling is combined with in-code scaffolding rounds to maximize contiguity gains.

## When to use

Use when a draft long-read assembly (e.g. FASTA with N-gaps from Flye) should be scaffolded with long reads and have its Ns replaced by spanning read sequence, to improve N50 and gap-free contiguity.

## When NOT to use

- The input assembly contains no N-gaps and only single-round scaffolding is needed — use ntLink scaffolding with overlap detection (Basic Protocol 1) instead of gap-filling rounds.
- The goal is misassembly correction rather than gap closure — use ntLink mappings with Tigmint-long (Alternate Protocol 2) instead.
- No long-read data spanning the gaps is available, so gap-filling with raw read sequence cannot proceed.

## Inputs

- Draft genome assembly FASTA with N-gaps (e.g. celegans_flye.fa from Flye)
- Long ONT reads in FASTQ/FASTA (SRR10028109 dataset)
- Scaffold k-mer and window sizes (k=24, w=250) and number of rounds (e.g. 3)

## Outputs

- Final scaffolded, gap-filled assembly FASTA with *ntLink.scaffolds.fa suffix
- Per-round intermediate assemblies named with the target.k<w>.w<w>.z<z> prefix scheme
- abyss-fac stats table (N50, total scaffold length per round)
- QUAST --split-scaffold misassembly-aware contiguity report

## How to apply

Place the target assembly (e.g. celegans_flye.fa) and ONT long reads (SRR10028109) in the working directory, linked if needed. Run ntLink's gap-filling scaffolding with k=24 and w=250; when further contiguity is desired, use the ntLink_rounds Makefile's run_rounds_gaps target with rounds=3 so that in-code iterations re-scaffold each round's output, with mapping liftover reducing the cost of subsequent rounds. Collect per-round intermediate assemblies (named with the target.k<w>.w<w>.z<z> prefix scheme) and the final *ntLink.scaffolds.fa, in which gaps are filled with raw read sequence. Assess each round and the input with abyss-fac (ABySS v2.3.0+) for N50 and total scaffold length, and run QUAST in --split-scaffold mode to break scaffolds at Ns and compare misassembly-aware contiguity across rounds.

## Related tools

- **ntLink** (Performs minimizer-based long-read scaffolding with gap-filling and in-code scaffolding iterations (run_rounds_gaps Makefile target)) — https://github.com/bcgsc/ntLink
- **ABySS** (Provides abyss-fac (v2.3.0+) to compute N50 and total scaffold length for each round's output) — https://github.com/BirolLab/abyss
- **QUAST** (Evaluates assemblies in --split-scaffold mode, breaking scaffolds at Ns for misassembly-aware contiguity comparison) — https://github.com/ablab/quast
- **minimap2** (Supports mapping liftover to reduce computational cost of subsequent scaffolding rounds) — https://github.com/lh3/minimap2

## Examples

```
make -f ntLink_rounds.mk run_rounds_gaps target=celegans_flye.fa reads=SRR10028109.fastq.gz k=24 w=250 rounds=3
```

## Evaluation signals

- abyss-fac shows increasing scaffold N50 across rounds 1→3 relative to the input celegans_flye.fa, with total scaffold length preserved (no large over-joining).
- QUAST --split-scaffold confirms that contiguity gains are not artefacts of N-padding (scaffolds split at Ns retain improved contiguity).
- The final output FASTA carries the *ntLink.scaffolds.fa suffix and gaps are filled with raw read sequence rather than left as N runs.
- Per-round intermediate files following the target.k<w>.w<w>.z<z> prefix scheme exist, confirming each in-code round executed and lifted mappings correctly.
- All assembly and read input files are verified present in the working directory (soft links if necessary) before the Makefile run begins.

## Limitations

- Gaps are filled with raw (unpolished) long-read sequence, so filled regions may contain errors unless later polished (e.g. with ntEdit/Sealer or Racon).
- Gap-filling requires long reads that actually span the N-gaps; unsatisfied gaps remain as Ns.
- Computational cost of multi-round runs grows with each iteration; the article mitigates this via mapping liftover rather than re-mapping from scratch each round.

## Evidence

- [abstract] Recent improvements to ntLink have added important features such as overlap detection, gap-filling, and in-code scaffolding iterations: "Recent improvements to ntLink have added important features such as overlap detection, gap-filling, and in-code scaffolding iterations"
- [methods] Basic Protocol 3 runs three in-code rounds with gap-filling and liftover: "1. Compute ordered minimizer sketches of the input target assembly and long reads"
- [abstract] ntLink infers order and orientation from minimizer-based mappings rather than full read alignments: "ntLink utilizes minimizer-based mappings to infer how input sequences should be ordered and oriented into scaffolds"
- [readme] Evaluation uses abyss-fac and QUAST with scaffold splitting: [evidence span withheld — anchor status: not_found]
