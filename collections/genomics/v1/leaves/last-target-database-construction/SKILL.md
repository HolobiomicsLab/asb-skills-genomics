---
name: last-target-database-construction
description: 'Use when preparing a NanoPipe run: a reference FASTA (full genome such
  as plasmodium.tgz or a gene region like EGFR exons 17-22) must be converted into
  a LAST database before aligning MinION query reads.'
when_to_use_negative:
- The query data is for metagenomics analysis — NanoPipe does not yet support metagenomics
  and expects a single defined target
- The alignment target already exists as a pre-built package (e.g., plasmodium.tgz
  already extracted and lastdb-indexed), so rebuilding would be redundant
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: NanoPipe
  role: Hosts the targets subfolder and helper script for creating the target database;
    bundles lastdb under NANOPIPE/tools/bin
  repo: https://github.com/IOB-Muenster/nanopipe2
- name: LAST 946
  role: lastdb builds the target index consumed by lastal during read-to-target alignment
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giy169
    title: NanoPipe—a web server for nanopore MinION sequencing data analysis
  source_content_hash: sha256:abf19de0e8567185f33594970f1b5683d3adaee7127a496fbea3c7c385f91b66
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IOB-Muenster/nanopipe2
  tools:
  - LAST
  - LAST 946
  - NanoPipe
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# last-target-database-construction

## Summary

Constructs a LAST alignment target database from a reference FASTA (e.g., P. falciparum 3D7, human EGFR region) so that MinION reads can be mapped by NanoPipe's lastal/last-split pipeline. It is the prerequisite step that turns any species/sequence of interest into a searchable NanoPipe target.

## When to use

Use when preparing a NanoPipe run: a reference FASTA (full genome such as plasmodium.tgz or a gene region like EGFR exons 17-22) must be converted into a LAST database before aligning MinION query reads.

## When NOT to use

- The query data is for metagenomics analysis — NanoPipe does not yet support metagenomics and expects a single defined target
- The alignment target already exists as a pre-built package (e.g., plasmodium.tgz already extracted and lastdb-indexed), so rebuilding would be redundant

## Inputs

- Reference FASTA file (e.g., target.fasta from plasmodium.tgz or a RefSeq EGFR FASTA such as hg38 GCF_000001405.38)
- NanoPipe installation (nanopipe2-master.tgz from github.com/IOB-Muenster/nanopipe2) with NANOPIPE/targets subfolder

## Outputs

- LAST database index files (target.prj, target.suf, target.sds, target.bck) in the targets folder, ready as input.target for nanopipe_calc.pl

## How to apply

Place the target FASTA into the NANOPIPE/targets subfolder (either downloaded packages like plasmodium.tgz, extracted, or user-supplied references such as the hg38/RefSeq GCF_000001405.38 EGFR sequence). Build the database with NanoPipe's bundled lastdb, e.g. 'lastdb target target.fasta', which produces the LAST index files consumed by the alignment stage (lastal with MinION-optimized parameters from last-train, e.g. -A 15 -B 4 -P 2 -Q 0 -S 1 -a 15 -b 3 -k 2). The target can be created either manually or with a helper script. Verify the installation afterwards by calling NANOPIPE/check.sh so that last (version 946) and samtools are available for the downstream alignment steps.

## Related tools

- **NanoPipe** (Hosts the targets subfolder and helper script for creating the target database; bundles lastdb under NANOPIPE/tools/bin) — https://github.com/IOB-Muenster/nanopipe2
- **LAST 946** (lastdb builds the target index consumed by lastal during read-to-target alignment)

## Examples

```
NANOPIPE/tools/bin/lastdb target target.fasta
```

## Evaluation signals

- lastdb completes without error and produces the expected LAST index files next to target.fasta in NANOPIPE/targets
- NANOPIPE/check.sh reports last and samtools installed correctly after target creation
- A subsequent nanopipe_calc.pl -q input.query -t input.target run successfully produces calc.lastalign.maf, confirming the target index is queryable
- In the P. falciparum test case, ~99.9% of query reads map against the newly built 3D7 target, indicating a correctly constructed database

## Limitations

- The range of target species/sequences is not limited, but the target must be supplied as a single FASTA and indexed locally; no pre-built index is downloaded
- Reads that are too short can map as 'noise'; a read-length limit (e.g., 200 nucleotides) may need to be set at the alignment stage, which reflects on target granularity choices
- No changelog found for nanopipe2, so version drift in target-building scripts cannot be tracked from the repository alone

## Evidence

- [methods] Build the target database with NanoPipe's lastdb: "You can create a target database either with the help of a script or manually."
- [methods] lastdb invocation on target FASTA: [evidence span withheld — anchor status: not_found]
- [results] EGFR target creation with bundled lastdb: [evidence span withheld — anchor status: not_found]
- [methods] Installation check after target setup: "For installed packages and software call the script NANOPIPE/check.sh"
- [readme] Repository acquisition for local target building: "Download the file nanopipe2-master.tgz from the github repository https://github.com/IOB-Muenster/nanopipe2"
