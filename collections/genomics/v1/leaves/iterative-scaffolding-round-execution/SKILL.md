---
name: iterative-scaffolding-round-execution
description: Use when a draft long-read assembly (e.g. celegans_flye.fa) shows residual
  fragmentation and you want to test whether repeated gap-filling scaffolding rounds
  with ONT reads (SRR10028109) yield additional N50 gains beyond a single ntLink run.
when_to_use_negative:
- The input assembly is already highly contiguous and a single ntLink scaffolding
  round (Basic Protocol 2) suffices — extra rounds add compute without expected N50
  gains.
- 'Only short-read data is available: this skill requires long reads for minimizer-based
  scaffolding and gap-filling.'
- The goal is misassembly correction rather than scaffolding — use Tigmint-long (Alternate
  Protocol 2) with ntLink mappings instead.
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: ntLink
  role: Runs in-code iterative scaffolding rounds with gap-filling via the ntLink_rounds
    Makefile (run_rounds_gaps target) using minimizer-based mappings
  repo: https://github.com/bcgsc/ntLink
- name: ABySS
  role: abyss-fac (ABySS v2.3.0+) computes N50 and total scaffold length for the input
    assembly and each round's scaffold output
  repo: https://github.com/BirolLab/abyss
- name: QUAST
  role: Run in --split-scaffold mode to break scaffolds at Ns and compare misassembly-aware
    contiguity across rounds
  repo: https://github.com/ablab/quast
- name: minimap2
  role: Supports mapping liftover between rounds, reducing the computational cost
    of subsequent scaffolding iterations
  repo: https://github.com/lh3/minimap2
- name: btllib
  role: Required library dependency (1.7.1 or higher) for ntLink
  repo: https://github.com/BirolLab/btllib
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/cpz1.733
    title: 'ntLink: A Toolkit for <i>De Novo</i> Genome Assembly Scaffolding and Mapping
      Using Long Reads'
  source_content_hash: sha256:7fb79669e9e6808b52beb63b5289c711403656ad38ee34a41daa94739f01f135
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bcgsc/ntLink
  tools:
  - ABySS
  - QUAST
  - btllib
  - minimap2
  - ntLink
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# iterative-scaffolding-round-execution

## Summary

Executes multiple in-code rounds of ntLink long-read scaffolding with gap-filling via the ntLink_rounds Makefile (e.g. 3 rounds on a Flye C. elegans draft with ONT reads), leveraging mapping liftover to reduce per-round cost, and quantifies per-round contiguity gains with abyss-fac and QUAST --split-scaffold.

## When to use

Use when a draft long-read assembly (e.g. celegans_flye.fa) shows residual fragmentation and you want to test whether repeated gap-filling scaffolding rounds with ONT reads (SRR10028109) yield additional N50 gains beyond a single ntLink run.

## When NOT to use

- The input assembly is already highly contiguous and a single ntLink scaffolding round (Basic Protocol 2) suffices — extra rounds add compute without expected N50 gains.
- Only short-read data is available: this skill requires long reads for minimizer-based scaffolding and gap-filling.
- The goal is misassembly correction rather than scaffolding — use Tigmint-long (Alternate Protocol 2) with ntLink mappings instead.

## Inputs

- Draft long-read assembly FASTA (celegans_flye.fa)
- ONT long-read FASTQ files (SRR10028109 reads)
- ntLink_rounds Makefile (ntLink, btllib 1.7.1+ dependency)

## Outputs

- Per-round intermediate scaffold assemblies named with the target.k<w>.w<w>.z<z> prefix scheme
- Final 3-round post-scaffolding FASTA with *ntLink.scaffolds.fa suffix, gaps filled with raw read sequence
- abyss-fac statistics table (N50, total scaffold length) for input and each round
- QUAST --split-scaffold misassembly-aware contiguity report

## How to apply

Place the draft assembly (celegans_flye.fa) and SRR10028109 ONT FASTQ reads in the working directory, linking files if needed. Run the ntLink_rounds Makefile with the run_rounds_gaps target (gap-filling enabled), setting target=celegans_flye.fa, k=24, w=250, and rounds=3, so ntLink performs three in-code scaffolding iterations using minimizer-based mappings rather than read alignments; subsequent rounds use mapping liftover to avoid recomputing mappings from scratch. Collect each round's intermediate assemblies (named with the target.k<w>.w<w>.z<z> prefix scheme) and the final *ntLink.scaffolds.fa, in which gaps are filled with raw read sequence. Assess contiguity by computing N50 and total scaffold length for the input and each round's output with abyss-fac (ABySS v2.3.0+), and run QUAST in --split-scaffold mode to break scaffolds at Ns and compare misassembly-aware contiguity across rounds. Tabulate per-round metrics to determine whether additional rounds still improve contiguity or gains plateau.

## Related tools

- **ntLink** (Runs in-code iterative scaffolding rounds with gap-filling via the ntLink_rounds Makefile (run_rounds_gaps target) using minimizer-based mappings) — https://github.com/bcgsc/ntLink
- **ABySS** (abyss-fac (ABySS v2.3.0+) computes N50 and total scaffold length for the input assembly and each round's scaffold output) — https://github.com/BirolLab/abyss
- **QUAST** (Run in --split-scaffold mode to break scaffolds at Ns and compare misassembly-aware contiguity across rounds) — https://github.com/ablab/quast
- **minimap2** (Supports mapping liftover between rounds, reducing the computational cost of subsequent scaffolding iterations) — https://github.com/lh3/minimap2
- **btllib** (Required library dependency (1.7.1 or higher) for ntLink) — https://github.com/BirolLab/btllib

## Examples

```
make -f Makefile_ntLink_rounds run_rounds_gaps target=celegans_flye.fa reads=SRR10028109.fastq.gz k=24 w=250 rounds=3
```

## Evaluation signals

- Per-round abyss-fac N50 and total scaffold length are tabulated; each successive round should show non-decreasing N50 relative to the input celegans_flye.fa if scaffolding is working.
- Final output FASTA carries the expected *ntLink.scaffolds.fa suffix and intermediates follow the target.k<w>.w<w>.z<z> prefix scheme (k24.w250).
- QUAST --split-scaffold contiguity metrics on N-split assemblies agree with the abyss-fac trend, confirming gains are not artifacts of long N-gap stretches.
- Gap-filled sequences in the final scaffolds trace back to raw SRR10028109 read sequence.
- Mapping liftover logs show subsequent rounds reuse lifted mappings instead of recomputing mappings from scratch.

## Limitations

- Gains may plateau: the protocol is designed to test whether additional rounds yield further contiguity gains, and later rounds can show diminishing improvements.
- Gap-filling inserts raw read sequence, so filled gaps depend on long-read quality and may need downstream polishing.
- Computational cost of subsequent rounds is mitigated by mapping liftover, but each round still produces new intermediate assemblies that require storage and evaluation.

## Evidence

- [abstract] Recent improvements to ntLink have added important features such as overlap detection, gap-filling, and in-code scaffolding iterations: "Recent improvements to ntLink have added important features such as overlap detection, gap-filling, and in-code scaffolding iterations"
- [methods] in-code rounds mode executes three rounds of gap-filling scaffolding: [evidence span withheld — anchor status: not_found]
- [methods] Run iterative scaffolding with the ntLink_rounds Makefile using the run_rounds_gaps target: [evidence span withheld — anchor status: not_found]
- [methods] Run QUAST in --split-scaffold mode on the same assemblies to break scaffolds at Ns and compare misassembly-aware contiguity: [evidence span withheld — anchor status: not_found]
- [abstract] ntLink utilizes minimizer-based mappings to infer how input sequences should be ordered and oriented into scaffolds: "ntLink utilizes minimizer-based mappings to infer how input sequences should be ordered and oriented into scaffolds"
