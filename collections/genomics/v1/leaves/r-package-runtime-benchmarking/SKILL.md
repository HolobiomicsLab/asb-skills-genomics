---
name: r-package-runtime-benchmarking
description: Use when a paper claims an R package or script outperforms a manual workflow
  in time, and you must reproduce the runtime claim on the paper's own reference inputs
  (e.g. plastid genomes AB086179 or AY178864.1).
when_to_use_negative:
- The goal is to predict the full set of RNA editing sites from RNA-Seq data — ReFernment
  is explicitly not intended to predict every RNA editing site like RNA-Seq-based
  packages.
- The GFF3 input was created in Sequin — ReFernment has a known error producing no
  annotations for Sequin files, so any timing would be meaningless (files exported
  from Geneious still work).
- You want a runtime comparison across many plastomes in one call — ReFernment can
  take several minutes for a large number of plastomes, so the per-genome sub-minute
  criterion does not apply to that aggregate run.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: ReFernment
  role: The R package being benchmarked; it fully annotates RNA editing sites in a
    fern or hornwort plastid genome and its end-to-end runtime is what is measured
  repo: github.com/TARobison/ReFernment
- name: devtools
  role: Used to install ReFernment from GitHub (install_github) so the exact install
    provenance of the benchmarked build is known
- name: R
  role: Runtime environment in which the annotation is executed and timed with system.time
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/aps3.1216
    title: 'ReFernment: An R package for annotating <scp>RNA</scp> editing in plastid
      genomes'
  source_content_hash: sha256:38c7a2f9c5b09439f870d0682500780ca97fe88d5ed38d8dae28e09c6ceb16f5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - ReFernment
  - devtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# r-package-runtime-benchmarking

## Summary

Benchmark the wall-clock runtime of an R scientific package on a defined reference input to quantify speedup over a manual alternative. Here, ReFernment's full RNA-editing annotation of a fern or hornwort plastid genome is timed and must complete in under 60 seconds versus hours of manual annotation.

## When to use

Use when a paper claims an R package or script outperforms a manual workflow in time, and you must reproduce the runtime claim on the paper's own reference inputs (e.g. plastid genomes AB086179 or AY178864.1).

## When NOT to use

- The goal is to predict the full set of RNA editing sites from RNA-Seq data — ReFernment is explicitly not intended to predict every RNA editing site like RNA-Seq-based packages.
- The GFF3 input was created in Sequin — ReFernment has a known error producing no annotations for Sequin files, so any timing would be meaningless (files exported from Geneious still work).
- You want a runtime comparison across many plastomes in one call — ReFernment can take several minutes for a large number of plastomes, so the per-genome sub-minute criterion does not apply to that aggregate run.

## Inputs

- GFF3 annotation file with sequence embedded (e.g. for GenBank plastome AB086179 or AY178864.1)
- R environment with devtools and the ReFernment package installed from github.com/TARobison/ReFernment
- gffFolder directory containing the GFF file and an empty, separate outputFolder
- genomes vector of plastome names without file extensions

## Outputs

- Measured total wall-clock elapsed time (seconds) for one complete annotation run, recorded in a runtime_benchmark with genome identifier and installation provenance
- ReFernment output annotation files in outputFolder (five-column GenBank feature table and protein FASTA with conceptual translations), confirming the timed run was a full successful annotation

## How to apply

Install the package from its GitHub source with devtools (install_github("TARobison/ReFernment")) and record installation provenance so the benchmark is reproducible. Prepare the reference input exactly as the paper does: a GFF3 file with the sequence embedded (test plastomes AB086179 or AY178864.1) placed in gffFolder, with a separate, empty outputFolder so originals are not overwritten. Declare the genomes vector without file extensions since ReFernment appends extensions automatically. Wrap the full ReFernment(gffFolder, fastaFolderPath=NULL, gbFolderPath=NULL, outputFolder, genomes) call in system.time to capture total elapsed wall-clock time for the complete annotation run. Validate success by checking the measured time is below the paper's threshold of less than one minute per plastid genome, alongside the paper's accuracy invariants (all nonsense mutations annotated, no false positives).

## Related tools

- **ReFernment** (The R package being benchmarked; it fully annotates RNA editing sites in a fern or hornwort plastid genome and its end-to-end runtime is what is measured) — github.com/TARobison/ReFernment
- **devtools** (Used to install ReFernment from GitHub (install_github) so the exact install provenance of the benchmarked build is known)
- **R** (Runtime environment in which the annotation is executed and timed with system.time)

## Examples

```
system.time(ReFernment(gffFolder, fastaFolderPath=NULL, gbFolderPath=NULL, outputFolder, genomes))
```

## Evaluation signals

- Measured wall-clock elapsed time from system.time for a single typical fern or hornwort plastid genome is less than 60 seconds.
- The timed run completes the full annotation — producing the five-column GenBank feature table and protein FASTA — not a partial or errored run.
- Output annotation correctness matches paper claims: every nonsense mutation in the cDNA-confirmed plastomes (AB086179, AY178864.1) annotated with no false positives.
- No ReFernment warnings (more than 5 internal stops in a gene, or protein-sequence translation errors) fired during the timed run, indicating a clean benchmark input.
- Package installation provenance (GitHub repo and method, e.g. devtools::install_github("TARobison/ReFernment")) is recorded alongside the runtime measurement.

## Limitations

- ReFernment's output quality depends on the quality of the input annotations; it is not a substitute for manually checking gene annotations, so a fast run on sloppy input is not a valid benchmark.
- A known error exists where ReFernment produces no annotations for files created in Sequin (fix planned); files exported from Geneious still work.
- If a coding sequence has more than five internal stops, ReFernment errors and asks the user to manually check that gene; more than 10 internal stops suggests assembly or annotation error.
- Sequences whose coding regions span the origin of the genome can fail translation; the origin should be shifted so no CDS overlaps it before timing.
- The paper states no software version number for ReFernment and the repo has no changelog, so runtime comparisons across builds cannot be pinned to a release.

## Evidence

- [abstract] Manually accounting for RNA edits generally takes hours for a typical fern or hornwort plastid genome, but with ReFernment, this process takes less than a minute.: "Manually accounting for RNA edits generally takes hours for a typical fern or hornwort plastid genome, but with ReFernment, this process takes less than a minute."
- [readme] ReFernment requires only a GFF3 file (with sequence embedded) to work, but you can optionally provide a GenBank flat file and seperate fasta file.: "ReFernment *requires* only a GFF3 file (with sequence embedded) to work, but you can optionally provide a GenBank flat file and seperate fasta file, if your workflow requires this."
- [readme] Next, we simply call the ReFernment function and wait for it to finish. This can take several minutes if you have a large number of plastomes.: "Next, we simply call the `ReFernment` function and wait for it to finish. This can take several minutes if you have a large number of plastomes."
- [readme] The warning will appear if a given coding sequence has more than 5 internal stops.: "The warning will appear if a given coding sequence has more than 5 internal stops."
