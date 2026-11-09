---
name: nextflow-pipeline-execution
description: Use when untargeted/metagenomic nanopore FASTQ data from a MinION run
  must be screened for known virus species (e.g. Lassa, Ebola, Dengue) and their consensus
  genomes reconstructed reproducibly on a laptop.
when_to_use_negative:
- Input is not nanopore sequencing data but already-assembled contigs or short-read
  data — ViMOP is built for untargeted Oxford Nanopore reads
- The target is a large DNA virus with extensive repetitive regions (e.g. mpox) where
  assemblies may be inaccurate draft genomes rather than finished consensus
- No known virus reference is relevant — ViMOP detects and reconstructs known virus
  species, not novel virus discovery
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0622
tools:
- name: ViMOP
  role: The Nextflow-based pipeline being executed; orchestrates trimming, Centrifuge
    classification/filtering, minimap2 contaminant removal, Canu assembly, BLAST reference
    matching and consensus calling
  repo: https://github.com/opr-group-bnitm/vimop
- name: Nextflow
  role: Workflow engine used to launch the pipeline with a single command and selectable
    profiles (docker, conda, conda,mamba, apptainer)
  repo: https://www.nextflow.io/
- name: EPI2ME Desktop
  role: Optional graphical interface from ONT to install and run the same pipeline
    without the command line
  repo: https://nanoporetech.com/software/other/epi2me-desktop-application
- name: Docker
  role: Default execution profile providing reproducible software dependencies for
    every pipeline process
  repo: https://www.docker.com/
- name: Centrifuge
  role: Classifies reads (and contigs) against the ViMOP_DB index; classifications
    drive removal of non-viral reads at score >= 150 and produce the Krona plot
- name: medaka
  role: Neural-network consensus and variant calling on nanopore pileups against the
    BLAST-selected reference genome during consensus assembly
  repo: https://github.com/nanoporetech/medaka
- name: seqtk
  role: Lightweight FASTQ/FASTA processing used within read-level manipulation steps
    of the pipeline
  repo: https://github.com/lh3/seqtk
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btaf687
    title: 'ViMOP: a user-friendly and field-applicable pipeline for untargeted viral
      genome nanopore sequencing'
  source_content_hash: sha256:0c577c6944bc5440430800df85aef71c50f02a35c3903a18535280f10b8c2024
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/opr-group-bnitm/vimop
  tools:
  - Centrifuge
  - Docker
  - EPI2ME Desktop
  - EPI2ME desktop
  - Nextflow
  - ViMOP
  - medaka
  - minimap2
  - seqtk
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# nextflow-pipeline-execution

## Summary

Runs the ViMOP viral genome reconstruction pipeline for untargeted Oxford Nanopore reads via a single Nextflow command (or EPI2ME Desktop), producing trimmed, classified, cleaned reads and reference-guided consensus genomes summarized in an HTML report.

## When to use

Use when untargeted/metagenomic nanopore FASTQ data from a MinION run must be screened for known virus species (e.g. Lassa, Ebola, Dengue) and their consensus genomes reconstructed reproducibly on a laptop.

## When NOT to use

- Input is not nanopore sequencing data but already-assembled contigs or short-read data — ViMOP is built for untargeted Oxford Nanopore reads
- The target is a large DNA virus with extensive repetitive regions (e.g. mpox) where assemblies may be inaccurate draft genomes rather than finished consensus
- No known virus reference is relevant — ViMOP detects and reconstructs known virus species, not novel virus discovery

## Inputs

- Directory of nanopore FASTQ files passed via --fastq (optionally with barcode01, barcode02, ... subdirectories as samples)
- ViMOP_DB reference database (Centrifuge index all.1.cf-all.4.cf with prefix 'all', virus_taxids.txt, BLAST virus reference database, contamination.yaml host/reagent sets)
- contamination.yaml filter definitions (human GCF_000001405.39_GRCh38.p13_genomic.fna.gz, mouse GCA_000001635.8_GRCm38.p6_genomic.fna.gz, mastomys GCF_008632895.1_UCSF_Mcou_1_genomic.fna.gz, reagent)

## Outputs

- Cleaned read set after Centrifuge filtering (score >= 150 removes non-viral) and minimap2 host/contaminant mapping, with reads.tsv summary per sample
- De novo assembled contigs (Canu with iterative re-assembly) and BLAST-matched viral reference assignments
- Reference-guided consensus genomes with alignment .bam files, depth and variant files
- HTML report with Krona classification plot (optional separate Krona HTML via --output_krona_plot true)

## How to apply

Stage a FASTQ directory (optionally with barcode01/barcode02 subdirectories treated as separate samples) into a machine with the ViMOP_DB reference database and run the pipeline with `nextflow` using the default Docker profile (`-profile conda` or `-profile apptainer` on Linux without Docker, `-profile conda,mamba` for mamba). The pipeline first trims read ends with `--trim_length` to remove adapters and primers, then classifies reads with Centrifuge (Krona plot in the report; separate HTML via `--output_krona_plot true`) and removes non-viral reads scoring at least `--centrifuge_filter_min_score 150` (toggle with `--centrifuge_do_filter`). Surviving reads are mapped with minimap2 against host/reagent reference sets from contamination.yaml (default human; `--contamination_filters "reagent,mouse"` adds others), and non-mapping cleaned reads are de novo assembled with Canu followed by iterative re-assembly (`--reassemble_max_iter`, 0 to deactivate). Contigs are BLASTed against the ViMOP_DB virus reference database, the top hit guides reference-guided consensus assembly, and results are summarized in an HTML report. Hardware baseline is about 30 GB RAM and 16 CPUs; relax via `--min_cpus`, `--min_ram`, `--min_disk_space_work_gb`, `--min_disk_space_out_gb` at the cost of runtime (Canu is resource-hungry).

## Related tools

- **ViMOP** (The Nextflow-based pipeline being executed; orchestrates trimming, Centrifuge classification/filtering, minimap2 contaminant removal, Canu assembly, BLAST reference matching and consensus calling) — https://github.com/opr-group-bnitm/vimop
- **Nextflow** (Workflow engine used to launch the pipeline with a single command and selectable profiles (docker, conda, conda,mamba, apptainer)) — https://www.nextflow.io/
- **EPI2ME Desktop** (Optional graphical interface from ONT to install and run the same pipeline without the command line) — https://nanoporetech.com/software/other/epi2me-desktop-application
- **Docker** (Default execution profile providing reproducible software dependencies for every pipeline process) — https://www.docker.com/
- **Centrifuge** (Classifies reads (and contigs) against the ViMOP_DB index; classifications drive removal of non-viral reads at score >= 150 and produce the Krona plot)
- **medaka** (Neural-network consensus and variant calling on nanopore pileups against the BLAST-selected reference genome during consensus assembly) — https://github.com/nanoporetech/medaka
- **seqtk** (Lightweight FASTQ/FASTA processing used within read-level manipulation steps of the pipeline) — https://github.com/lh3/seqtk

## Examples

```
nextflow run opr-group-bnitm/vimop --fastq /data/fastq --trim_length 50 --centrifuge_filter_min_score 150 --contamination_filters "human" -profile docker
```

## Evaluation signals

- reads.tsv exists in each sample's tables directory summarizing read counts per filtering stage
- Cleaned read set contains no reads classified as non-viral with Centrifuge score >= 150 and no reads mapping to the selected host/reagent references
- Krona classification plot appears in the HTML report (and a separate HTML exists when --output_krona_plot true)
- For each detected virus, a BLAST-anchored reference and a reference-guided consensus genome with corresponding alignment .bam, depth and variant files are produced
- Resource guards pass: the run only starts when at least --min_cpus, --min_ram, --min_disk_space_work_gb and --min_disk_space_out_gb are satisfied

## Limitations

- For large DNA viruses with extensive repetitive regions (e.g. mpox), assemblies may contain inaccuracies and should be regarded as draft genomes; alignment .bam files should be reviewed
- Centrifuge accuracy is limited and the score-150 filter may remove some false-positive viral reads
- Pipeline performance degrades on lower-specification machines; Canu in particular needs substantial resources (recommended >= 30 GB RAM, 16 CPUs)
- Conda profile did not work with the 'nextflow run' command prior to version 1.0.4; Nextflow 25.10.2 triggered a 'Cannot invoke method optional() on null object' bug (fixed in 1.0.4)
- Fixed bugs: crash on non-standard nucleotides (not ACGT, not N) in 1.0.1, and unresolved error when <INV> inversions are present in the VCF before bcftools consensus calling

## Evidence

- [abstract] Executed with a single Nextflow command or via the EPI2ME Desktop interface (ONT), results are summarized in an HTML report.: "Executed with a single Nextflow command or via the EPI2ME Desktop interface (ONT), results are summarized in an HTML report."
- [readme] trim + centrifuge filter: "At the beginning the pipeline trims the ends of the reads to remove adapter sequences and primers."
- [readme] centrifuge filter threshold: "The pipeline will remove all reads that are classified to anything that is not viral with at least this score."
- [readme] host/contaminant removal: "Host and contaminant reads are removed by mapping them against reference sequences and extracting those that map."
- [abstract] consensus strategy: "The software assembles de novo contigs, matches contigs to known viral references and uses them to assemble consensus genomes."
- [readme] mpox limitation: "for large DNA viruses with extensive repetitive regions (e.g., mpox), assemblies may contain inaccuracies and should be regarded as draft genomes."
