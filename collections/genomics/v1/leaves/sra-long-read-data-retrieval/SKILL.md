---
name: sra-long-read-data-retrieval
description: Use when a benchmark or reproduction requires empirical long-read WGS
  read sets paired with specific reference assemblies — e.g. NCBI SRA accessions (SRR10028111,
  SRR13070625) or GIAB HG002 PacBio HiFi data.
when_to_use_negative:
- The dataset of interest is CycloneSEQ G400-ER HG002 data — this was generated in-house
  from the HG002 lymphoid cell line and cannot be retrieved from SRA or GIAB.
- Synthetic data with controlled depth, accuracy, and read length is required — use
  the PBSIM3-based simulation scripts in kNN-overlap-finder instead of SRA retrieval.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: kNN-overlap-finder
  role: Provides the real-data processing pipelines (quality control and filtering,
    format conversion and normalization, metadata management) applied to the retrieved
    SRA/GIAB read sets
  repo: https://github.com/jzhang-dev/kNN-overlap-finder
- name: Fedrann
  role: Downstream consumer of the retrieved long-read FASTQ/FASTA inputs for overlap
    detection benchmarking
  repo: https://github.com/jzhang-dev/FEDRANN
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giag048
    title: 'FEDRANN: effective long-read overlap detection based on dimensionality
      reduction and approximate nearest neighbors'
  source_content_hash: sha256:7033e243aca5505de72162b5495756fbd7a29053e952a262586c4487c0fdfd67
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jzhang-dev/kNN-overlap-finder
  tools:
  - BLEND
  - Fedrann
  - MECAT2
  - MHAP
  - Python
  - kNN-overlap-finder
  - minimap2
  - wtdbg2
  - xRead
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9444444444444444
n_claims: 18
---

# sra-long-read-data-retrieval

## Summary

Retrieves the real whole-genome long-read sequencing datasets (C. elegans ONT SRR10028111, D. melanogaster ONT SRR13070625 from NCBI SRA; HG002 PacBio HiFi from GIAB) and their matched reference assemblies (WBcel235, Drosophila Release 6, T2T-CHM13 v2.0) needed to reproduce the FEDRANN overlap-detection benchmarks.

## When to use

Use when a benchmark or reproduction requires empirical long-read WGS read sets paired with specific reference assemblies — e.g. NCBI SRA accessions (SRR10028111, SRR13070625) or GIAB HG002 PacBio HiFi data.

## When NOT to use

- The dataset of interest is CycloneSEQ G400-ER HG002 data — this was generated in-house from the HG002 lymphoid cell line and cannot be retrieved from SRA or GIAB.
- Synthetic data with controlled depth, accuracy, and read length is required — use the PBSIM3-based simulation scripts in kNN-overlap-finder instead of SRA retrieval.

## Inputs

- NCBI SRA accession SRR10028111 (ONT R9 reads, C. elegans)
- NCBI SRA accession SRR13070625 (ONT reads, D. melanogaster)
- Genome in a Bottle (GIAB) HG002 PacBio HiFi sequencing data
- NCBI Assembly ID GCA_000002985.3 (C. elegans WBcel235, WormBase WS285)
- NCBI Assembly ID GCF_000001215.4 (Drosophila melanogaster Release 6)
- T2T-CHM13 human assembly v2.0

## Outputs

- Raw long-read FASTQ/FASTA files per accession, QC-filtered and normalized for benchmarking
- Reference assembly FASTA files (WBcel235, Drosophila Release 6, T2T-CHM13 v2.0)
- Metadata records managed via the kNN-overlap-finder data-processing pipelines

## How to apply

Retrieve the ONT R9 sequencing data for C. elegans (SRR10028111) and D. melanogaster (SRR13070625) from the NCBI Sequence Read Archive, and the PacBio HiFi HG002 data from the Genome in a Bottle (GIAB) project. Pair each read set with its corresponding reference assembly: Caenorhabditis elegans WBcel235 (WormBase release WS285; NCBI Assembly ID GCA_000002985.3), Drosophila melanogaster Release 6 (GCF_000001215.4), or the complete human T2T-CHM13 assembly v2.0. Use the data-processing pipelines in github.com/jzhang-dev/kNN-overlap-finder for quality control and filtering, format conversion and normalization, and metadata management before benchmarking Fedrann, minimap2, MHAP, MECAT2, wtdbg2, xRead, or BLEND on these inputs. For human H4C/H4O/H4P benchmarks, supply k-mer size hints from the paper (k=30 for H4C, k=50 for H4O) alongside the datasets. Note that CycloneSEQ G400-ER HG002 data were generated in-house from the HG002 lymphoid cell line and are not available via SRA.

## Related tools

- **kNN-overlap-finder** (Provides the real-data processing pipelines (quality control and filtering, format conversion and normalization, metadata management) applied to the retrieved SRA/GIAB read sets) — https://github.com/jzhang-dev/kNN-overlap-finder
- **Fedrann** (Downstream consumer of the retrieved long-read FASTQ/FASTA inputs for overlap detection benchmarking) — https://github.com/jzhang-dev/FEDRANN

## Evaluation signals

- Each downloaded FASTQ/FASTA file matches the expected accession (SRR10028111 for C. elegans ONT, SRR13070625 for D. melanogaster ONT) and read-length distribution consistent with ONT R9 or PacBio HiFi chemistry.
- Each read set is paired with the correct reference assembly ID: GCA_000002985.3 for C. elegans, GCF_000001215.4 for D. melanogaster, T2T-CHM13 v2.0 for human HG002.
- Retrieved data passes the kNN-overlap-finder quality-control and filtering steps and is converted to the format expected by each benchmarked overlap-detection tool.
- Ground-truth alignments derived from these datasets satisfy the paper's filtering criteria (read length ≥ 5 kb, aligned fraction ≥ 50%, mapping quality ≥ 30; ≥ 100 matched bases and ≥ 10% identity), confirming data/reference compatibility.

## Limitations

- CycloneSEQ G400-ER HG002 data are only available as in-house generated data from the HG002 lymphoid cell line, so the full 5-dataset benchmark cannot be reproduced from public repositories alone.
- No DOI or Zenodo archive is provided for the data-processing workflows; reproducibility of real-data handling depends solely on the GitHub repositories (e.g. github.com/jzhang-dev/kNN-overlap-finder).

## Evidence

- [discussion] ONT R9 sequencing data for C. elegans (SRR10028111): "ONT R9 sequencing data for C. elegans (SRR10028111)"
- [discussion] and D. melanogaster (SRR13070625) were retrieved from the NCBI Sequence Read Archive (SRA): "and D. melanogaster (SRR13070625) were retrieved from the NCBI Sequence Read Archive (SRA)"
- [discussion] PacBio HiFi sequencing data for the HG002 sample was downloaded from the Genome in a Bottle (GIAB) project: "PacBio HiFi sequencing data for the HG002 sample was downloaded from the Genome in a Bottle (GIAB) project"
- [discussion] the Caenorhabditis elegans WBcel235 assembly (WormBase release WS285; NCBI Assembly ID: GCA_000002985.3): "the Caenorhabditis elegans WBcel235 assembly (WormBase release WS285; NCBI Assembly ID: GCA_000002985.3)"
- [readme] Pipelines for processing empirical sequencing data: Quality control and filtering, Format conversion and normalization: [evidence span withheld — anchor status: external_doc]
