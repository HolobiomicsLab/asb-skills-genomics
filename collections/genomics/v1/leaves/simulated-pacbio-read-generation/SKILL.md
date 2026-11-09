---
name: simulated-pacbio-read-generation
description: 'Use when a de novo transcript reconstruction method must be evaluated
  against a known ground truth: simulate full-length PacBio Iso-Seq reads from multigene
  family isoforms at varying depths and abundances before running IsoCon or ICE.'
when_to_use_negative:
- You have experimental (not simulated) Iso-Seq reads already pre-processed by the
  Iso-Seq pipeline — run IsoCon directly on the full-length reads instead of simulating
  data
- Your reads do not span the region of interest end-to-end (e.g., short amplicons
  or fragmented transcripts) — IsoCon's model assumes reads span the transcript end-to-end
- You only need to error-correct or cluster existing real CCS reads (use the IsoCon
  pipeline / ICE directly rather than the simulation workflow)
tools:
- name: IsoCon_Eval
  role: Snakemake workflow repository that drives all simulations, runs the configured
    read counts/abundances, and executes the evaluation of IsoCon vs ICE on the generated
    read sets
  repo: https://github.com/ksahlin/IsoCon_Eval
- name: IsoCon
  role: Tool under evaluation that reconstructs transcripts from the simulated full-length
    read sets via 'IsoCon pipeline -fl_reads <reads> -outfolder <output>'
  repo: https://github.com/ksahlin/IsoCon
- name: ICE
  role: PacBio Iso-Seq clustering/correction baseline run on the identical simulated
    read sets for precision/recall comparison
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-018-06910-x
    title: Deciphering highly similar multigene family transcripts from Iso-Seq data
      with IsoCon
  source_content_hash: sha256:2044180f26817e86643989141a851b6229049c741d4066b0815f9c92ec0933b2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ksahlin/IsoCon_Eval
  tools:
  - ICE
  - Iso-Seq
  - IsoCon
  - IsoCon_Eval
  - PacBio
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# simulated-pacbio-read-generation

## Summary

Generates simulated PacBio Iso-Seq read sets with known ground-truth isoforms from highly similar multigene family copies (e.g., HSFY2, DAZ2, TSPY13P) across read depths, abundance models, and mutation rates, enabling precision/recall benchmarking of transcript reconstruction methods like IsoCon against ICE.

## When to use

Use when a de novo transcript reconstruction method must be evaluated against a known ground truth: simulate full-length PacBio Iso-Seq reads from multigene family isoforms at varying depths and abundances before running IsoCon or ICE.

## When NOT to use

- You have experimental (not simulated) Iso-Seq reads already pre-processed by the Iso-Seq pipeline — run IsoCon directly on the full-length reads instead of simulating data
- Your reads do not span the region of interest end-to-end (e.g., short amplicons or fragmented transcripts) — IsoCon's model assumes reads span the transcript end-to-end
- You only need to error-correct or cluster existing real CCS reads (use the IsoCon pipeline / ICE directly rather than the simulation workflow)

## Inputs

- simulated_experiments.json config file (gene members, read counts, runs, abundance models, tools, paths)
- Reference sequences of target gene family members (e.g., HSFY2, DAZ2, TSPY13P) with defined isoform sets and mutation rates
- Installed IsoCon (commit 79589f3) and ICE, plus snakemake (python3)

## Outputs

- Simulated PacBio Iso-Seq full-length read sets (FASTA/FASTQ) per gene family, read depth, abundance model, and replicate run
- Simulated ground-truth isoform set for scoring precision and recall
- Reconstructed transcript sets from IsoCon (final_candidates.fa) and ICE for identical simulated read sets

## How to apply

Install IsoCon (paper commit 79589f3 from ksahlin/IsoCon) and ICE first, since the IsoCon_Eval snakemake pipeline assumes both tools are installed. Edit the config file simulated_experiments.json to set GENE_MEMBERS (e.g., ["HSFY2", "DAZ2", "TSPY13P"]), READ_COUNTS (e.g., 20, 100, 500, 2500, 12500 full-length reads), RUNS (1–10 replicates), ABUNDANCE (constant or exponential, yielding the unequal-abundance isoform mixtures of ~30 isoforms), and TOOLS (ISOCON, ICE); remove experiments via the EXPERIMENTS field if needed. Fill in all paths in the config, then dry-run with `snakemake --configfile simulated_experiments.json --latency-wait 100 --verbose -n` before launching the real pipeline on a cluster (e.g., via sbatch clustering). The pipeline generates simulated read sets per family/depth/abundance/run together with the simulated ground-truth isoform set, against which nucleotide-level precision and recall of each tool's reconstructed transcripts (IsoCon's final_candidates.fa vs ICE output) are computed.

## Related tools

- **IsoCon_Eval** (Snakemake workflow repository that drives all simulations, runs the configured read counts/abundances, and executes the evaluation of IsoCon vs ICE on the generated read sets) — https://github.com/ksahlin/IsoCon_Eval
- **IsoCon** (Tool under evaluation that reconstructs transcripts from the simulated full-length read sets via 'IsoCon pipeline -fl_reads <reads> -outfolder <output>') — https://github.com/ksahlin/IsoCon
- **ICE** (PacBio Iso-Seq clustering/correction baseline run on the identical simulated read sets for precision/recall comparison)

## Examples

```
snakemake --configfile simulated_experiments.json --latency-wait 100 --verbose -n
```

## Evaluation signals

- The snakemake dry run (`-n`) completes listing all expected jobs for the configured GENE_MEMBERS x READ_COUNTS x ABUNDANCE x RUNS combinations without errors
- Generated read sets match the configured READ_COUNTS (e.g., 20, 100, 500, 2500, 12500 full-length reads) per replicate run
- Each simulation carries a known ground-truth isoform set so nucleotide-level precision and recall can be computed for both tools
- IsoCon's final_candidates.fa on the simulated data matches the published reconstruction output
- Downstream benchmark reproduces the headline behaviour: ICE precision near 0 at read depth >= 500 while IsoCon precision stays above 80%

## Limitations

- The pipeline is designed to be run on a cluster with job distribution (e.g., sbatch); a local run of the full experiment grid (5 depths x 10 runs x 2 abundance models x multiple families x 2 tools) may be impractical
- Very low-abundance simulated transcripts may be unrecoverable at shallow depths, mirroring the article's note that low-abundance transcripts might not be captured since deep PacBio sequencing can be cost-prohibitive
- Simulated reads must span the transcript end-to-end (as in targeted Iso-Seq), so results do not transfer to datasets where reads only cover part of the transcript

## Evidence

- [readme] Repository for running experiments with snakemake pipeline. Evaluation scripts for IsoCon.: [evidence span withheld — anchor status: external_doc]
- [readme] This experiment pipeline assumes you have IsoCon and ICE installed.: [evidence span withheld — anchor status: external_doc]
- [readme] You can remove tools or experiments by changing the fields in the config file `simulated_experiments.json`.: [evidence span withheld — anchor status: external_doc]
- [intro] Using simulated data, we demonstrate that IsoCon has substantially higher precision and recall than ICE18 across a wide range of sequencing depths: "Using simulated data, we demonstrate that IsoCon has substantially higher precision and recall than ICE18 across a wide range of sequencing depths"
