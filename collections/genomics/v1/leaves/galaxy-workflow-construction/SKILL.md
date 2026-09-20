---
name: galaxy-workflow-construction
description: Use when building or reconstructing a reproducible multi-tool analysis
  pipeline for nanopore long-read data (assembly, polishing, taxonomic classification)
  as a connected Galaxy workflow with explicit inputs and outputs.
when_to_use_negative:
- The analysis requires a single standalone tool run with no chained steps — a Galaxy
  history with one job suffices, no workflow definition is needed
- Input data are short reads (150–300 bp) or require Pacific Biosciences-specific
  tools that were not intensively tested for ONT data in NanoGalaxy
- The user needs the newest long-read tools under development — the toolkit deliberately
  integrated only the most established and widely used tools
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0196
tools:
- name: Galaxy
  role: Platform that wraps the tools, handles software dependencies, chains history
    steps, and exports refined workflows (.ga definitions)
- name: minimap2
  role: Computes read-to-read overlaps (long-read overlap mode) and read-to-assembly
    alignments feeding miniasm and Racon
- name: miniasm
  role: Builds the string graph from minimap2 overlaps and generates initial assembly
    unitigs
- name: Racon
  role: Polishes miniasm unitigs using read-to-assembly alignments to produce the
    error-corrected consensus
- name: Kraken2
  role: Classifies long reads by exact k-mer matches against a pre-indexed database,
    emitting per-read output and a clade-ranked report
- name: Krona
  role: Consumes the Kraken2 report to render an interactive HTML pie chart of taxonomic
    composition
- name: Conda
  role: Manages dependencies for all installed tools and their versions inside Galaxy
- name: NanoGalaxy
  role: The Galaxy-based toolkit whose wrappers and workflows are distributed via
    Docker and the Galaxy Tool Shed
  repo: https://github.com/mmiladi/galaxy-nanopore
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giaa105
    title: 'NanoGalaxy: Nanopore long-read sequencing data analysis in Galaxy'
  source_content_hash: sha256:f7220f978528b513d98fd50e2418d5f55d1b44ca6cd0ee43ef3bd5567d3481ba
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mmiladi/galaxy-nanopore
  tools:
  - Conda
  - Galaxy
  - Kraken2
  - Krona
  - NanoGalaxy
  - Racon
  - miniasm
  - minimap2
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# galaxy-workflow-construction

## Summary

Assemble multi-step NanoGalaxy analysis workflows inside the Galaxy platform by chaining wrapped long-read tools (e.g. minimap2 → miniasm → Racon, or Kraken2 → Krona) with FASTQ/database inputs and defined outputs, then exporting the connected steps as a .ga workflow definition. Galaxy wraps tool dependencies via Conda and distributes wrappers via the Galaxy Tool Shed, removing the need for command-line scripting.

## When to use

Use when building or reconstructing a reproducible multi-tool analysis pipeline for nanopore long-read data (assembly, polishing, taxonomic classification) as a connected Galaxy workflow with explicit inputs and outputs.

## When NOT to use

- The analysis requires a single standalone tool run with no chained steps — a Galaxy history with one job suffices, no workflow definition is needed
- Input data are short reads (150–300 bp) or require Pacific Biosciences-specific tools that were not intensively tested for ONT data in NanoGalaxy
- The user needs the newest long-read tools under development — the toolkit deliberately integrated only the most established and widely used tools

## Inputs

- Nanopore long-read FASTQ reads (e.g. dataset deposited via Zenodo/GigaDB)
- Pre-indexed Kraken2 k-mer taxonomic database (for classification workflows)
- Read-to-read overlap alignments (PAF) from minimap2 long-read overlap mode
- miniasm unitig assembly (for read-remapping and Racon polishing)

## Outputs

- Polished consensus assembly as polished_assembly.fasta (miniasm + Racon workflow)
- Per-read Kraken2 classification output and clade-ranked report
- Interactive Krona HTML pie chart of taxonomic composition
- Exported Galaxy workflow definition file (e.g. kraken2_krona_workflow.ga)

## How to apply

Import the nanopore FASTQ dataset (e.g. from Zenodo/GigaDB) into a Galaxy history and attach any pre-indexed reference database (e.g. a Kraken2 k-mer database) as workflow inputs. Chain tools by data provenance: for assembly, run minimap2 in its long-read overlap mode to compute read-to-read overlaps, feed those into miniasm to build the string graph and emit unitigs, remap the original reads to the unitigs with minimap2, and polish with Racon to produce an error-corrected consensus exported as polished_assembly.fasta; for metagenomics, run Kraken2 to classify reads (per-read output plus clade-ranked report), then feed the report into Krona to render an interactive HTML pie chart. Mark the FASTQ and database as workflow inputs and the final artifacts (FASTA or Krona HTML) as workflow outputs, then save the connected steps as an exported Galaxy workflow definition (.ga file, e.g. kraken2_krona_workflow.ga). Deploy wrappers from the Galaxy Tool Shed and rely on Conda for dependency management; a Docker image (mmiladi/galaxy-nanopore) bundles the full toolkit. Validate by re-running the workflow on the deposited dataset and checking each intermediate history dataset matches the expected tool output type.

## Related tools

- **Galaxy** (Platform that wraps the tools, handles software dependencies, chains history steps, and exports refined workflows (.ga definitions))
- **minimap2** (Computes read-to-read overlaps (long-read overlap mode) and read-to-assembly alignments feeding miniasm and Racon)
- **miniasm** (Builds the string graph from minimap2 overlaps and generates initial assembly unitigs)
- **Racon** (Polishes miniasm unitigs using read-to-assembly alignments to produce the error-corrected consensus)
- **Kraken2** (Classifies long reads by exact k-mer matches against a pre-indexed database, emitting per-read output and a clade-ranked report)
- **Krona** (Consumes the Kraken2 report to render an interactive HTML pie chart of taxonomic composition)
- **Conda** (Manages dependencies for all installed tools and their versions inside Galaxy)
- **NanoGalaxy** (The Galaxy-based toolkit whose wrappers and workflows are distributed via Docker and the Galaxy Tool Shed) — https://github.com/mmiladi/galaxy-nanopore

## Examples

```
docker run -d -p 8080:80 mmiladi/galaxy-nanopore
```

## Evaluation signals

- Each history dataset's datatype matches the expected intermediate (minimap2 overlap/alignment files, miniasm unitig FASTA, Racon polished_assembly.fasta, Kraken2 report, Krona HTML)
- The exported .ga file exposes exactly the intended workflow inputs (FASTQ reads, Kraken2 database) and outputs (polished FASTA or Krona HTML) with connected steps
- Re-running the workflow on the deposited Zenodo/GigaDB nanopore dataset reproduces the documented assembly or classification result
- Wrappers pass the Travis continuous integration testing framework used for Galaxy wrapper development
- The Docker container (mmiladi/galaxy-nanopore) serves the Galaxy instance at localhost:8080 with the full toolkit installed and Conda-resolved dependencies

## Limitations

- Other long-read analysis tools exist or are under development but were not integrated; the focus was limited to the most established and widely used tools
- Pacific Biosciences-supporting tools were expected to 'also work' inside Galaxy but only ONT data was intensively tested
- No software version numbers for the integrated tools are stated in the publication's discussion; dependencies are only described as Conda-managed
- Community extension of the toolkit is expected, not yet realized; new tools must still be assembled into existing or new toolkits

## Evidence

- [abstract] Galaxy provides user-friendly workflow interface: "The Galaxy platform provides a user-friendly interface to computational command line–based tools, handles the software dependencies, and provides refined workflows."
- [discussion] Wrappers distributed via Tool Shed: "All developed Galaxy wrappers are available for installation from the Galaxy Tool Shed (https://toolshed.g2.bx.psu.edu/)"
- [discussion] Conda dependency management: "All tools and their dependencies are installed on the Galaxy platform and are managed by the Conda framework for dependency management"
- [readme] Docker run command from README: "NanoGalaxy: Galaxy toolkit and workflows for long-read Oxford Nanopore data"
