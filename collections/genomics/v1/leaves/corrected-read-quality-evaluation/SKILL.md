---
name: corrected-read-quality-evaluation
description: Use when corrected long reads (PacBio or ONT, FASTA output from a corrector
  such as CONSENT, Canu, Daccord, FLAS, or MECAT) must be quantitatively compared
  for correction accuracy against a reference genome.
when_to_use_negative:
- The reads are uncorrected raw long reads and the goal is polishing an existing assembly
  rather than scoring corrected reads — use CONSENT-polish/ELECTOR assembly evaluation
  instead.
- No reference genome is available, so ELECTOR cannot align corrected reads to compute
  error rate, recall, and precision.
- The goal is comparing assembly quality (e.g. Flye assembly polishing outcomes) rather
  than read-level correction accuracy.
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: ELECTOR
  role: Evaluates corrected reads against the reference to compute post-correction
    error rate, recall, and precision for the benchmark table
- name: CONSENT
  role: Self-corrector under evaluation; run with --type PB on PacBio datasets at
    default parameters before ELECTOR scoring
  repo: https://github.com/morispi/CONSENT
- name: Canu
  role: State-of-the-art self-correction baseline run with recommended PacBio settings
    for comparison
- name: Daccord
  role: State-of-the-art self-correction baseline; achieved best output volume/quality
    on E. coli and S. cerevisiae but with 3x-11x more memory than CONSENT or MECAT
- name: FLAS
  role: State-of-the-art self-correction baseline for comparison on the six simulated
    PacBio datasets
- name: MECAT
  role: State-of-the-art self-correction baseline for comparison; memory-frugal like
    CONSENT
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-020-80757-5
    title: Scalable long read self-correction and assembly polishing with multiple
      sequence alignment
  source_content_hash: sha256:d59850876a0a97d23cd6787ae3b4b8747cf411b4d364c430931f05f83ba49b3f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/morispi/CONSENT
  tools:
  - CONSENT
  - Canu
  - Daccord
  - ELECTOR
  - FLAS
  - MECAT
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# corrected-read-quality-evaluation

## Summary

Benchmark the quality of long reads after self-correction by scoring corrected reads against a reference genome with ELECTOR (post-correction error rate, recall, precision) and recording each corrector's wall-clock runtime and peak RAM, as done to produce Table 1 of the CONSENT paper.

## When to use

Use when corrected long reads (PacBio or ONT, FASTA output from a corrector such as CONSENT, Canu, Daccord, FLAS, or MECAT) must be quantitatively compared for correction accuracy against a reference genome.

## When NOT to use

- The reads are uncorrected raw long reads and the goal is polishing an existing assembly rather than scoring corrected reads — use CONSENT-polish/ELECTOR assembly evaluation instead.
- No reference genome is available, so ELECTOR cannot align corrected reads to compute error rate, recall, and precision.
- The goal is comparing assembly quality (e.g. Flye assembly polishing outcomes) rather than read-level correction accuracy.

## Inputs

- Corrected long reads in FASTA format (output of CONSENT-correct, Canu, Daccord, FLAS, or MECAT)
- Reference genome for the sequenced organism (E. coli, S. cerevisiae, or C. elegans)
- Raw simulated PacBio long-read datasets at 30x and 60x coverage
- Per-run runtime and peak RAM measurements for each corrector

## Outputs

- Per-dataset post-correction error rate, recall, and precision computed by ELECTOR
- Wall-clock runtime and peak RAM consumption per corrector run
- Aggregated benchmark table in the layout of Table 1

## How to apply

Run each corrector on the same long-read dataset (e.g. simulated PacBio E. coli, S. cerevisiae, or C. elegans at 30x/60x, with CONSENT run as `CONSENT-correct --type PB` at defaults: windowSize 500, minSupport 4, maxSupport 150, merSize 9, solid 4, anchorSupport 8, minAnchors 2). Evaluate every tool's corrected reads against the reference with ELECTOR to compute per-dataset post-correction error rate, recall, and precision. In parallel, record wall-clock runtime and peak RAM consumption for every corrector run (e.g. CONSENT on the example dataset uses up to ~750 MB RAM on 4 cores). Aggregate error rate, recall, precision, runtime, and memory into a single benchmark table in the layout of Table 1. Validation anchors: reproduced CONSENT error rates should approximate 0.3111% (E. coli 30x), 0.1833% (E. coli 60x), and 0.3858% (C. elegans 60x), with recall, precision, runtime, and memory within reported ranges.

## Related tools

- **ELECTOR** (Evaluates corrected reads against the reference to compute post-correction error rate, recall, and precision for the benchmark table)
- **CONSENT** (Self-corrector under evaluation; run with --type PB on PacBio datasets at default parameters before ELECTOR scoring) — https://github.com/morispi/CONSENT
- **Canu** (State-of-the-art self-correction baseline run with recommended PacBio settings for comparison)
- **Daccord** (State-of-the-art self-correction baseline; achieved best output volume/quality on E. coli and S. cerevisiae but with 3x-11x more memory than CONSENT or MECAT)
- **FLAS** (State-of-the-art self-correction baseline for comparison on the six simulated PacBio datasets)
- **MECAT** (State-of-the-art self-correction baseline for comparison; memory-frugal like CONSENT)

## Examples

```
./CONSENT-correct --in example/reads.fasta --out example/correctedReads.fasta --type PB
```

## Evaluation signals

- Reproduced CONSENT post-correction error rates match published Table 1 values: ~0.3111% (E. coli 30x), ~0.1833% (E. coli 60x), ~0.3858% (C. elegans 60x)
- Recall and precision values fall within the reported Table 1 ranges for each dataset and corrector
- Runtime and peak RAM measurements are within reported ranges (e.g. CONSENT example dataset: ~2 min and up to 750 MB RAM on 4 cores)
- All six simulated datasets (three organisms x two coverages) are evaluated with the same ELECTOR metrics so tools are compared on identical bases
- Benchmark table reproduces the Table 1 layout with error rate, recall, precision, runtime, and memory columns for every tool

## Limitations

- Daccord's alignment-based overlapping step consumed 3x-11x more memory than CONSENT or MECAT and it failed to scale to C. elegans, so missing entries are an expected outcome for some correctors
- CONSENT's correction-step runtime tends to be higher than other state-of-the-art methods, so runtime comparisons should be interpreted alongside memory and quality
- Results on simulated PacBio datasets do not directly transfer to real ONT data, where CONSENT is the only method that efficiently scales to ultra-long reads (full human dataset, reads up to 1.5 Mbp, 10 days, ~5 TB disk for the overlaps file)

## Evidence

- [results] ELECTOR benchmark validation: [evidence span withheld — anchor status: not_found]
- [readme] expected CONSENT example runtime/memory: "This should take about 2 min and use up to 750 MB of RAM, using 4 cores."
- [results] benchmark findings on Daccord: [evidence span withheld — anchor status: not_found]
- [discussion] CONSENT correction runtime caveat: "However, the runtime of CONSENT’s correction step tends to be higher than that of other state-of-the-art methods."
- [abstract] ONT scalability claim: "CONSENT is the only method that efficiently scales to ultra-long reads, and allows to process a full human dataset, containing reads reaching up to 1.5 Mbp, in 10 days."
