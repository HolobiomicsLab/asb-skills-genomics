---
name: fastx-toolkit-command-line-processing
description: Use when raw small-RNA sequencing reads (FASTQ or FASTA libraries in
  INSERTS_DIRS) still carry 3' adaptor sequences and must be clipped with fastx_clipper
  before Wbench filtering, e.g. under miRPursuit's --trim flag.
when_to_use_negative:
- Input is already adaptor-trimmed FASTA libraries — running --trim again would clip
  at remaining adaptor-like motifs; start from Step 1 (Wbench Filter) instead
- Libraries without the expected 5'/3' adaptor architecture — fastx_clipper assumes
  each sRNA sequence lies between the two adaptor sequences
- Quality control is desired before trimming — miRPursuit's fastqc-based QC is not
  yet implemented, so this skill does not provide pre-trim QC
edam_operation: http://edamontology.org/operation_2005
edam_topics:
- http://edamontology.org/topic_3169
tools:
- name: FASTX-Toolkit
  role: Provides fastx_clipper (adaptor clipping of sRNA reads) and fastq_to_fasta
    (FASTQ-to-FASTA conversion of raw libraries)
  repo: http://hannonlab.cshl.edu/fastx_toolkit/
- name: miRPursuit
  role: Orchestrates the trimming via the --trim flag and the ADAPTOR variable in
    workdirs.cfg, feeding trimmed FASTA libraries into the Wbench Filter step
  repo: https://github.com/forestbiotech-lab/miRPursuit
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/1873-3468.12746
    title: mi<scp>RP</scp>ursuit—a pipeline for automated analyses of small <scp>RNA</scp>s
      in model and nonmodel plants
  source_content_hash: sha256:e2e49bfdbc98168c9dbbf5b6f41e7896f25b2392e921811d8a474bd041b15e44
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - FASTX-Toolkit
  - miRPursuit
  edam_operation: http://edamontology.org/operation_2005
  edam_topics:
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# fastx-toolkit-command-line-processing

## Summary

Applies command-line FASTX-Toolkit utilities (fastx_clipper for adaptor clipping, fastq_to_fasta for format conversion) to small-RNA reads, as orchestrated by the miRPursuit pipeline's Pre-preprocessing stage.

## When to use

Use when raw small-RNA sequencing reads (FASTQ or FASTA libraries in INSERTS_DIRS) still carry 3' adaptor sequences and must be clipped with fastx_clipper before Wbench filtering, e.g. under miRPursuit's --trim flag.

## When NOT to use

- Input is already adaptor-trimmed FASTA libraries — running --trim again would clip at remaining adaptor-like motifs; start from Step 1 (Wbench Filter) instead
- Libraries without the expected 5'/3' adaptor architecture — fastx_clipper assumes each sRNA sequence lies between the two adaptor sequences
- Quality control is desired before trimming — miRPursuit's fastqc-based QC is not yet implemented, so this skill does not provide pre-trim QC

## Inputs

- Raw small-RNA FASTQ reads (optionally fastq.gz archives) or library FASTA files in INSERTS_DIR
- ADAPTOR sequence variable from config/workdirs.cfg (e.g. TGGAATTCTCGGGTGCCAAGG)

## Outputs

- Adaptor-trimmed FASTA read libraries in the working directory, one per input library, ready for the Wbench Filter step
- Read-count statistics in [workdir]/counts and execution logs in [workdir]/logs

## How to apply

Read the ADAPTOR value from the config/workdirs.cfg file (the test configuration uses the Solexa-style adaptor 'TGGAATTCTCGGGTGCCAAGG') and confirm the --trim flag was set on the miRPursuit.sh command line. For each converted library FASTA in INSERTS_DIR, run fastx_clipper from the FASTX-Toolkit, searching each read for the adaptor sequence and treating each sRNA read as the nucleotide string between the 5' and 3' adaptor sequences. Write the adaptor-trimmed reads as FASTA output libraries in the working directory, then hand them to the Wbench Filter module (Step 1: Adaptor trimming & Wbench Filter). When converting raw FASTQ input, fastq_to_fasta from FASTX-Toolkit is used (installable via install.sh); keep the toolkit on the PATH or set paths in config/software_dirs.cfg. Validate that one trimmed FASTA exists per input library and that read ends no longer contain adaptor remnants.

## Related tools

- **FASTX-Toolkit** (Provides fastx_clipper (adaptor clipping of sRNA reads) and fastq_to_fasta (FASTQ-to-FASTA conversion of raw libraries)) — http://hannonlab.cshl.edu/fastx_toolkit/
- **miRPursuit** (Orchestrates the trimming via the --trim flag and the ADAPTOR variable in workdirs.cfg, feeding trimmed FASTA libraries into the Wbench Filter step) — https://github.com/forestbiotech-lab/miRPursuit

## Examples

```
bash pathToMiRPursuit/miRPursuit.sh -f 1 -l 2 --fasta test_dataset- --trim
```

## Evaluation signals

- One trimmed FASTA output file exists for each input library in INSERTS_DIR
- Output files are valid FASTA with adaptor remnants removed from read ends
- Trimmed reads fall within the sRNA size range expected by the downstream filter defaults (min_length=18, max_length=26 in the workbench filter configuration)
- Execution log for the trimming step ends in .log.ok and read-count statistics are written to [workdir]/counts
- The --trim flag is only accepted with no argument and the adaptor used equals the ADAPTOR value in workdirs.cfg

## Limitations

- Quality control with fastqc is not yet implemented in the miRPursuit workflow, so no read-quality check precedes trimming
- FASTX-Toolkit must be present on the PATH or installed via install.sh; fastq_to_fasta unavailability forces a different conversion path
- Trimming assumes reads are sequences between the 5' and 3' adaptors; libraries lacking this structure will be mishandled

## Evidence

- [methods] fastx_clipper adaptor search in the pipeline: "the reads are then searched for adaptor sequences using the fastx_clipper software of the FASTX toolkit"
- [readme] --trim flag and ADAPTOR configuration variable: "Set this flag to perform adaptor triming. No argument should be given. The adaptor is in the workdirs.cfg config file in the variable ADAPTOR."
- [methods] fastq_to_fasta availability from FASTX-Toolkit: "If `fastq_to_fasta <http://hannonlab.cshl.edu/fastx_toolkit/commandline.html#fastq_to_fasta_usage>`_ from `FASTX-Toolkit <http://hannonlab.cshl.edu/fastx_toolkit/>`_ is not on available"
- [readme] FASTX toolkit installed via install.sh: [evidence span withheld — anchor status: external_doc]
