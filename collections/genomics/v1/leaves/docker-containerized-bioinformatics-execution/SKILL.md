---
name: docker-containerized-bioinformatics-execution
description: Use when executing Mana's --mrna QC analysis on ONT alignment BAM files
  and a reference FASTA, and a self-managed dependency install of SAMtools/BCFtools/Pysam/bedtools
  is undesirable.
when_to_use_negative:
- When a stable internet connection or Docker daemon is unavailable and dependencies
  must be installed manually from their listed websites
- When the input is not an ONT alignment BAM plus reference FASTA (e.g. raw FASTQ
  reads that have not yet been aligned with Minimap2)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3573
tools:
- name: Mana
  role: Command-line QC tool executed inside the pre-built Docker image to analyse
    ONT alignments and report mRNA quality statistics
  repo: https://github.com/scchess/Mana
- name: SAMtools
  role: Dependency bundled in the Mana container used to generate sorted and indexed
    BAM files consumed by the analysis
  repo: http://www.htslib.org
- name: BCFtools
  role: Dependency resolved automatically by the Docker build, used for variant calling
    and consensus generation
  repo: http://www.htslib.org
- name: Pysam
  role: Python dependency inside the container used for BAM-level statistics
  repo: https://pysam.readthedocs.io/en/latest/api.html
- name: bedtools
  role: Dependency listed in the Mana README and resolved automatically by the Docker
    image
  repo: https://bedtools.readthedocs.io/en/latest/
provenance:
  source_task_ids:
  - task_003
  - task_007
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-41354-y
    title: mRNA vaccine quality analysis using RNA sequencing
  source_content_hash: sha256:7d12d58309fb62de0e7d5e253ed2a9aa69fd52fa57fb73fe290693906e9797f2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scchess/Mana
  tools:
  - BCFtools
  - Mana
  - Pysam
  - SAMTool
  - SAMtools
  - bedtools
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3573
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# docker-containerized-bioinformatics-execution

## Summary

Run the Mana mRNA vaccine QC toolkit inside its pre-built Docker image so that dependencies (SAMtools, BCFtools, Pysam, bedtools) are resolved automatically, using volume mounts to expose local BAM/FASTA files to the container.

## When to use

Use when executing Mana's --mrna QC analysis on ONT alignment BAM files and a reference FASTA, and a self-managed dependency install of SAMtools/BCFtools/Pysam/bedtools is undesirable.

## When NOT to use

- When a stable internet connection or Docker daemon is unavailable and dependencies must be installed manually from their listed websites
- When the input is not an ONT alignment BAM plus reference FASTA (e.g. raw FASTQ reads that have not yet been aligned with Minimap2)

## Inputs

- Sorted, indexed ONT alignment BAM file (e.g. /src/alignment.bam)
- Reference transcript FASTA file (e.g. /src/file.fasta)
- Docker with access to the pre-built scchess/mana image

## Outputs

- Mana QC report files written to the output directory specified by -o (e.g. /src/out), including mRNA length distribution and integrity statistics

## How to apply

Pull the pre-built image with 'docker pull scchess/mana && docker tag scchess/mana mana' rather than building locally, since a self-build takes >= 20 minutes and is not recommended. Mount the host working directory into the container at /src with '-v ${PWD}:/src' so input BAM and FASTA files are visible inside the container. Invoke the analysis in mRNA mode with 'python3 mana.py --mrna', supplying the ONT alignment via '-b /src/alignment.bam', the reference transcript via '-f /src/file.fasta', and the output directory via '-o /src/out'. The container internally resolves SAMtools, BCFtools 1.16, Pysam 0.20.0 and bedtools 2.30.0, so no manual dependency installation is needed. Expect the analysis to complete in about 10 minutes; afterwards inspect the /src/out directory for the generated QC report.

## Related tools

- **Mana** (Command-line QC tool executed inside the pre-built Docker image to analyse ONT alignments and report mRNA quality statistics) — https://github.com/scchess/Mana
- **SAMtools** (Dependency bundled in the Mana container used to generate sorted and indexed BAM files consumed by the analysis) — http://www.htslib.org
- **BCFtools** (Dependency resolved automatically by the Docker build, used for variant calling and consensus generation) — http://www.htslib.org
- **Pysam** (Python dependency inside the container used for BAM-level statistics) — https://pysam.readthedocs.io/en/latest/api.html
- **bedtools** (Dependency listed in the Mana README and resolved automatically by the Docker image) — https://bedtools.readthedocs.io/en/latest/

## Examples

```
docker run -v ${PWD}:/src -i -t mana python3 mana.py --mrna -b /src/alignment.bam -f /src/file.fasta -o /src/out
```

## Evaluation signals

- The analysis completes within the expected ~10 minutes runtime stated in the README
- The output directory passed via '-o /src/out' exists and contains the QC report files produced by Mana (file_exists check)
- Reported mRNA statistics match published values, e.g. 58.2% full-length eGFP reads and 23% fragmented molecules relative to the 1153 nt expected transcript
- The docker run command exits without dependency errors, confirming SAMtools, BCFtools, Pysam and bedtools resolved inside the container

## Limitations

- Building the Docker image yourself takes >= 20 minutes and is explicitly not recommended; the pre-built image should be pulled instead
- The Mana GitHub repository is archived, so future maintenance of the pre-built image is uncertain
- Files must be mounted correctly into the container with the -v option; incorrect mounts cause the tool to fail to see input files

## Evidence

- [readme] docker pull pre-built image instruction: "docker pull scchess/mana && docker tag scchess/mana mana"
- [readme] self-build duration warning: "If you would like to build it yourself (>= 20 minutes and not recommended, a pre-built docker image is available)"
- [readme] mounting requirement: "In order to run the program with your files, it is important to make sure the files are mounted correctly to docker."
- [readme] analysis runtime: "The analysis time will be about 10 minutes."
- [readme] docker resolves dependencies: "Docker will automatically work out the dependencies."
