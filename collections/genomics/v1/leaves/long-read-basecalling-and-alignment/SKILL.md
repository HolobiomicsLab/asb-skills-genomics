---
name: long-read-basecalling-and-alignment
description: Use when starting from ONT POD5 signal files that must be basecalled
  and mapped (e.g. ENCODE dRNA replicates) before modification detection or transcript
  quantification.
when_to_use_negative:
- Input is already basecalled and mapped (unmapped/mapped BAM or FASTQ) — use Dogme's
  remap, kallisto, or annotateRNA entry points instead of the full basecalling workflow
- Input is single-cell barcoded cDNA without the seqspec/splitcode single-cell workflow
  (readType='CDNA' with singleCell=true), which has a separate path
- Sequencing was not performed on Oxford Nanopore (e.g. PacBio HiFi/Iso-Seq) — Dorado
  and modkit operate on ONT signal and modBAM data
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0196
tools:
- name: Dogme
  role: Nextflow pipeline that orchestrates model download, basecalling, BAM merging,
    mapping, and modification filtering
  repo: https://github.com/mortazavilab/dogme
- name: Dorado
  role: Performs basecalling of POD5 signal files, including modified-base calling
  repo: https://github.com/nanoporetech/dorado
- name: minimap2
  role: Spliced alignment of basecalled reads against the reference genome (direct-RNA/cDNA
    mapping)
  repo: https://github.com/mortazavilab/dogme
- name: modkit
  role: Extracts and converts modified-base calls from modBAM into bedMethyl/BED files
    that are then filtered
  repo: https://github.com/nanoporetech/modkit
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btag066
    title: 'Dogme: a nextflow pipeline for reprocessing nanopore RNA and DNA modifications'
  source_content_hash: sha256:ef2fdf9bd1042ad491ce2757acee1b7a12395680a72e1574de58602a865b0f49
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mortazavilab/dogme
  tools:
  - Dogme
  - Dorado
  - Nextflow
  - minimap2
  - modkit
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# long-read-basecalling-and-alignment

## Summary

Reprocess Oxford Nanopore POD5 signal files into aligned, modification-aware BAMs using the Dogme Nextflow pipeline, which automates Dorado basecalling, BAM merging, minimap2 spliced mapping, and modkit-based modification extraction/filtering.

## When to use

Use when starting from ONT POD5 signal files that must be basecalled and mapped (e.g. ENCODE dRNA replicates) before modification detection or transcript quantification.

## When NOT to use

- Input is already basecalled and mapped (unmapped/mapped BAM or FASTQ) — use Dogme's remap, kallisto, or annotateRNA entry points instead of the full basecalling workflow
- Input is single-cell barcoded cDNA without the seqspec/splitcode single-cell workflow (readType='CDNA' with singleCell=true), which has a separate path
- Sequencing was not performed on Oxford Nanopore (e.g. PacBio HiFi/Iso-Seq) — Dorado and modkit operate on ONT signal and modBAM data

## Inputs

- ONT POD5 (or FAST5) signal files for one or more samples/replicates (direct RNA, cDNA, or genomic DNA)
- Reference genome FASTA (e.g. mouse GRCm39) and annotation GTF (e.g. GENCODE v36)
- Dogme Nextflow config file specifying pod5 directory, output and work directories, and genome_annot_refs

## Outputs

- Basecalled, merged, and minimap2-aligned BAM files per sample
- modkit-derived bedMethyl/BED files of modification sites (m6A, m5C, inosine, pseudouridine, Nm) after filtering
- FASTQ and kallisto/LR-Kallisto transcript quantification for RNA and cDNA inputs
- QC artifacts such as qc_summary.csv, inventory_report.tsv, and per-sample softwareVersion.txt

## How to apply

Place the POD5 files (e.g. from ENCODE ENCSR160HKZ C2C12 dRNA replicates) in a top-level directory referenced from a Dogme config file, and ensure tool paths are listed in a dogme.profile or use the provided Docker/Singularity image. Dogme then automatically downloads the latest Dorado models (only if the model directory does not already exist), runs Dorado basecalling, merges the per-replicate BAMs, and maps reads with minimap2 — for RNA/cDNA inputs it converts the GTF to a junction BED and uses `--splice-max 500000` for long-intron spliced alignment (e.g. against mouse GRCm39 with GENCODE v36). For RNA and DNA read types it subsequently extracts modifications (m6A, m5C, inosine, pseudouridine, Nm) with modkit and filters them with scripts/filterbed.py using params.minCov and params.perMod thresholds. Aggregate detected sites per replicate (e.g. via scripts/report_bed_rep.py) and compare against expected counts to confirm correct reproduction; the paper reports 96,603 m6A, 43,476 m5C, 8,829 inosine, 10,055 pseudouridine, and 30,320 Nm sites across three biological replicates.

## Related tools

- **Dogme** (Nextflow pipeline that orchestrates model download, basecalling, BAM merging, mapping, and modification filtering) — https://github.com/mortazavilab/dogme
- **Dorado** (Performs basecalling of POD5 signal files, including modified-base calling) — https://github.com/nanoporetech/dorado
- **minimap2** (Spliced alignment of basecalled reads against the reference genome (direct-RNA/cDNA mapping)) — https://github.com/mortazavilab/dogme
- **modkit** (Extracts and converts modified-base calls from modBAM into bedMethyl/BED files that are then filtered) — https://github.com/nanoporetech/modkit

## Evaluation signals

- Dorado model download runs only once — the model directory must exist before subsequent steps
- Aligned BAMs produced by minimap2 for RNA/cDNA should reflect junction BED conversion and `--splice-max 500000` long-intron handling
- Filtered modification BEDs must respect params.minCov and params.perMod thresholds applied by scripts/filterbed.py
- Per-sample modification counts should match expected targets: e.g. task_003 averages of 147,915 m6A, 86,723 m5C, 21,256 inosine, 24,545 pseudouridine, 83,207 Nm (vs paper's 96,603 m6A, 43,476 m5C, 8,829 inosine, 10,055 pseudouridine, 30,320 Nm total across three replicates)
- Modification profiles should be reproducible across biological replicates (compare per-sample bedMethyl outputs with scripts/report_bed_rep.py)

## Limitations

- Version-dependent: tested with dorado 1.2, samtools 1.15.1, minimap2 2.28, mod_kit 0.5, kallisto 0.51.1, bustools 0.43.2; specific Dorado model versions/checksums are not recorded
- Requires tool paths in dogme.profile (or an empty file) in the launch directory, or the provided Docker/Singularity image; needs Nextflow and Java 17+
- Tested on SLURM clusters and Macs; long-running tasks depend on built-in retry/error strategies, and unclassified barcoded reads are published without downstream analysis

## Evidence

- [abstract] Dogme automates the reprocessing of ONT POD5 files by integrating basecalling using Dorado: "we developed Dogme to automate basecalling, alignment, modification detection, and transcript quantification"
- [intro] Core pipeline steps from POD5 to aligned BAMs and filtered modifications: "Given a folder of pod5 files, Dogme will download the latest dorado models, run dorado, merge bams, and map them with minimap2."
- [intro] Modification extraction and filtering applied after mapping: "For RNA and DNA, it will extract modifications using modkit and filter those."
- [readme] minimap2 spliced-alignment parameter used by the current Dogme version: [evidence span withheld — anchor status: external_doc]
- [readme] Filter thresholds governing retained modification sites: [evidence span withheld — anchor status: external_doc]
