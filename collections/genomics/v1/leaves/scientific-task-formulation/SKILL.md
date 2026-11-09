---
name: scientific-task-formulation
description: Use when a section's text must be triaged into research-relevant sub-tasks
  (e.g. benchmark simulations, ablations) versus setup/maintenance operations, and
  only bounded scientific tasks with measurable outputs should be emitted.
when_to_use_negative:
- The section text covers only contributor workflow mechanics — mamba/poetry environment
  setup, dependency updates, release tagging, PyPI and bioconda publishing — with
  no research operation present
- The input is a tool installation or packaging README (e.g. 'pip install seq2squiggle',
  'poetry publish') rather than a description of a scientific analysis or benchmark
- A task card already exists for the same bounded experiment (e.g. the four-mode noise
  ablation on R10.4.1); reformulating it again would duplicate rather than scope a
  new task
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: seq2squiggle
  role: 'Scientific subject of admissible tasks: transformer-based nanopore signal
    simulator whose noise-mode ablations (noise sampler, duration sampler) define
    bounded, artifact-producing sub-tasks'
  repo: https://github.com/ZKI-PH-ImageAnalysis/seq2squiggle
- name: mamba
  role: 'Example of a filtered-out operation class: package-manager environment setup
    is infrastructure, not a scientific sub-task'
- name: poetry
  role: 'Example of a filtered-out operation class: dependency management and PyPI
    publishing are setup, not research'
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae744
    title: End-to-end simulation of nanopore sequencing signals with feed-forward
      transformers
  source_content_hash: sha256:78d8feb0015dd336394f9e3ccbf248af98b8456b85f4c776973b0e5b98bf36cd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZKI-PH-ImageAnalysis/seq2squiggle
  tools:
  - mamba
  - poetry
  - seq2squiggle
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# scientific_task_formulation

## Summary

Formulates a bounded, artifact-producing scientific sub-task from article or repository text (e.g. reproducing the four-mode noise ablation of AUC match rate with seq2squiggle on R10.4.1 data), while explicitly filtering out contributor/packaging mechanics such as mamba/poetry environment setup, version bumping, release tagging, and PyPI/bioconda publishing.

## When to use

Use when a section's text must be triaged into research-relevant sub-tasks (e.g. benchmark simulations, ablations) versus setup/maintenance operations, and only bounded scientific tasks with measurable outputs should be emitted.

## When NOT to use

- The section text covers only contributor workflow mechanics — mamba/poetry environment setup, dependency updates, release tagging, PyPI and bioconda publishing — with no research operation present
- The input is a tool installation or packaging README (e.g. 'pip install seq2squiggle', 'poetry publish') rather than a description of a scientific analysis or benchmark
- A task card already exists for the same bounded experiment (e.g. the four-mode noise ablation on R10.4.1); reformulating it again would duplicate rather than scope a new task

## Inputs

- Article section text or repository README chunk (e.g. methods section, installation/contributing section)
- Extracted entity index: tools, workflow_steps, filter_steps, findings with section-attributed evidence spans
- Prior source cards surfacing the skill (task_id, research_question, finding, workflow, tools)

## Outputs

- Zero or more structured scientific sub-task cards, each with research_question, bounded method, tools, and expected measurable output (or explicitly no card when the section fails the admissibility filter)

## How to apply

First inspect the section text and classify every described operation: package-manager environment creation (mamba/conda create), dependency management (poetry add/update/lock), release mechanics (poetry version patch, git tag, GitHub release), and publishing (poetry build/publish to PyPI, bioconda) are setup, not research. Apply the scientific-contributution admissibility filter: an operation is admissible only if it consumes a scientific input (e.g. FASTA/FASTQ reads, blow5 signals), produces a bounded artifact (simulated signals, a metric table), and has a success criterion expressible as a measurement (e.g. AUC of average match rate improving from 85.41 to 93.93 with amplitude noise for seq2squiggle on R10.4.1). For admissible content, phrase the sub-task with concrete tools, chemistry profiles, and noise-mode toggles drawn from the article/README (e.g. seq2squiggle's --noise-sampling and --duration-sampling four-mode ablation). Validate by re-reading the emitted card: if no bounded, artifact-producing scientific operation is present, emit no card at all — silence is the correct output for packaging-only sections.

## Related tools

- **seq2squiggle** (Scientific subject of admissible tasks: transformer-based nanopore signal simulator whose noise-mode ablations (noise sampler, duration sampler) define bounded, artifact-producing sub-tasks) — https://github.com/ZKI-PH-ImageAnalysis/seq2squiggle
- **mamba** (Example of a filtered-out operation class: package-manager environment setup is infrastructure, not a scientific sub-task)
- **poetry** (Example of a filtered-out operation class: dependency management and PyPI publishing are setup, not research)

## Examples

```
seq2squiggle predict example.fasta -o example.blow5 --duration-sampling False --dwell-mean 10.0 --dwell-std 0.0 --noise-sampling False --noise-std 0.0
```

## Evaluation signals

- Every emitted card names a scientific input (FASTA/FASTQ, blow5) and a bounded artifact (simulated signals, AUC table); cards citing only 'poetry publish' or 'git tag v0.3.1' indicate filter failure
- For the R10.4.1 ablation task, expected metric ranges are checkable against the article: seq2squiggle AUC of average match rate improves from 85.41 to 93.93, squigulator from 76.26 to 90.70, when amplitude noise is added
- The task is reproducible from README commands (four noise modes: both samplers enabled, --noise-sampling False, --duration-sampling False, both static) without inventing unlisted parameters
- Sections whose only content is environment setup or release mechanics yield zero cards — an empty output is the correct validation result
- Each card's tools list contains only tools with section-attributed evidence (e.g. seq2squiggle from abstract/intro), not generic build tooling

## Limitations

- The article's own methods section mixes contributor-guide content with science; naive extraction surfaces packaging steps (poetry install/update, git tagging) as workflow steps and must be filtered manually
- Testing workflow is not implemented upstream ('### Testing / **Not implemented yet**'), so tasks built on the dev environment cannot assume automated test validation
- Performance claims depend on hardware: README notes GPU is recommended and CPU-only inference is slower, affecting the feasibility boundary of benchmark-reproduction tasks

## Evidence

- [methods] Section text covers contributor workflow mechanics, not research: "we use mamba, a faster replacement for the conda package manager"
- [readme] Admissible scientific subject of task formulation: "`seq2squiggle` is a deep learning-based tool for generating artifical nanopore signals from DNA sequence data."
- [readme] Concrete bounded experiment: four noise-mode ablation options: [evidence span withheld — anchor status: external_doc]
- [abstract] Measurable success criterion grounding the R10.4.1 ablation finding: "Seq2squiggle consistently outperforms existing tools across multiple datasets, demonstrating superior similarity to real data"
