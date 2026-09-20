---
name: rna-seq-sra-read-acquisition
description: Use when the starting point is NCBI SRA accession numbers (SRR/SRX) for
  the paper's single-organism RNA-Seq datasets and paired-end FASTQ files are needed
  to launch rnaSPAdes, SPAdes, Trinity, or other transcriptome assemblers.
when_to_use_negative:
- If the reads are already local FASTQ files — acquisition is unnecessary and assembly/evaluation
  should start directly.
- 'If the data are Oxford Nanopore or PacBio long reads: the article notes their high
  error rate prevents using output reads directly as complete transcripts, and SPAdes
  is primarily developed for Illumina data.'
- If the target is metatranscriptome assembly — the paper explicitly states the approach
  was initially designed for RNA-Seq data obtained from a single organism and only
  potentially applicable to metatranscriptomes.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3168
tools:
- name: rnaSPAdes
  role: downstream de novo transcriptome assembler that consumes the acquired paired-end
    FASTQ reads
  repo: https://github.com/ablab/spades
- name: SPAdes
  role: genome assembler benchmarked on the same acquired RNA-Seq FASTQ reads for
    runtime/RAM comparison
  repo: https://github.com/ablab/spades
- name: Trinity
  role: downstream transcriptome assembler consuming the acquired FASTQ reads in the
    benchmarking
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giz100
    title: 'rnaSPAdes: a <i>de novo</i> transcriptome assembler and its application
      to RNA-Seq data'
  source_content_hash: sha256:0c7389592caf56219928ae57ad107717530f1806c1a583a1ad237692b1f83b8a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ablab/spades
  tools:
  - SPAdes
  - Trinity
  - rnaSPAdes
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# rna-seq-sra-read-acquisition

## Summary

Retrieves the paired-end RNA-Seq FASTQ reads used in the rnaSPAdes benchmarking (Human large SRR1957703/SRR1957706, Arabidopsis SRR5344669/SRR5344670, plus Human SRR5133163, Mouse SRX648736, Worm SRR1560107, Corn SRR1588569) from NCBI SRA so the compared assemblers can be run and timed on the paper's datasets.

## When to use

Use when the starting point is NCBI SRA accession numbers (SRR/SRX) for the paper's single-organism RNA-Seq datasets and paired-end FASTQ files are needed to launch rnaSPAdes, SPAdes, Trinity, or other transcriptome assemblers.

## When NOT to use

- If the reads are already local FASTQ files — acquisition is unnecessary and assembly/evaluation should start directly.
- If the data are Oxford Nanopore or PacBio long reads: the article notes their high error rate prevents using output reads directly as complete transcripts, and SPAdes is primarily developed for Illumina data.
- If the target is metatranscriptome assembly — the paper explicitly states the approach was initially designed for RNA-Seq data obtained from a single organism and only potentially applicable to metatranscriptomes.

## Inputs

- NCBI SRA run accessions: SRR1957703, SRR1957706 (Human large); SRR5344669, SRR5344670 (Arabidopsis); SRR5133163 (Human); SRX648736 (Mouse, experiment-level); SRR1560107 (Worm); SRR1588569 (Corn)
- Internet access to NCBI SRA and the SRA Toolkit (fasterq-dump) for FASTQ retrieval

## Outputs

- Paired-end Illumina FASTQ files per accession (mate 1 / mate 2), one dataset per pair of runs, ready as input to rnaSPAdes, SPAdes, Trinity, SOAPdenovo-Trans, BinPacker, and Bridger
- A dataset-to-accession manifest mapping each FASTQ pair to the paper's dataset name (Human large, Arabidopsis, etc.)

## How to apply

Resolve the run accessions from the article's methods section: Human large = SRR1957703 + SRR1957706 and Arabidopsis = SRR5344669 + SRR5344670 (the two largest datasets used for the runtime/RAM comparison), with Human SRR5133163, Mouse SRX648736 (experiment-level, may expand to runs), Worm SRR1560107, and Corn SRR1588569 for the full panel. Download each accession in FASTQ format with paired mates split into separate files (e.g., fasterq-dump --split-files), since SPAdes-family tools require FASTQ or BAM for their read-processing steps. Verify that both mates of every pair were retrieved with matching read counts before launching assembly under /usr/bin/time -v so wall-clock time and peak RAM can be attributed to the correct dataset. The resulting FASTQs are what make the Table 5 anchors checkable: SOAPdenovo-Trans at least ~3x faster with <30 GB RAM, while BinPacker, Bridger, and Trinity exceed 100 GB peak RAM on Arabidopsis.

## Related tools

- **rnaSPAdes** (downstream de novo transcriptome assembler that consumes the acquired paired-end FASTQ reads) — https://github.com/ablab/spades
- **SPAdes** (genome assembler benchmarked on the same acquired RNA-Seq FASTQ reads for runtime/RAM comparison) — https://github.com/ablab/spades
- **Trinity** (downstream transcriptome assembler consuming the acquired FASTQ reads in the benchmarking)

## Examples

```
fasterq-dump --split-files SRR1957703 SRR1957706 SRR5344669 SRR5344670
```

## Evaluation signals

- Both mates of each paired-end accession are present as separate FASTQ files with equal read counts (e.g., SRR1957703 _1/_2 pairs).
- Accession-to-dataset mapping matches the methods: SRR1957703 + SRR1957706 = Human large; SRR5344669 + SRR5344670 = Arabidopsis.
- FASTQs load correctly into the assemblers, since SPAdes read error correction requires FASTQ or BAM format.
- Data volume is sufficient to reproduce the Table 5 anchors: SOAPdenovo-Trans under 30 GB peak RAM and at least ~3x faster, with BinPacker, Bridger, and Trinity exceeding 100 GB peak RAM on Arabidopsis.
- The Mouse dataset resolved from experiment accession SRX648736 yields the run(s) used in the paper's benchmarking.

## Limitations

- Mouse reads are specified only as experiment accession SRX648736, which may expand to multiple runs, whereas the other datasets are given as run-level SRR accessions.
- The benchmarked datasets are single-organism RNA-Seq; the paper warns that metatranscriptome assembly implies unaddressed challenges such as repetitive patterns in homologous genes.
- SPAdes-family tools are primarily for Illumina data (IonTorrent possible), so this acquisition path does not cover long-read transcriptomics.

## Evidence

- [methods] Human large: SRR1957703, SRR1957706: "Human large: SRR1957703, SRR1957706"
- [methods] Arabidopsis: SRR5344669, SRR5344670: "Arabidopsis: SRR5344669, SRR5344670"
- [methods] In order to run read error correction, reads should be in FASTQ or BAM format: "In order to run read error correction, reads should be in FASTQ or BAM format"
- [intro] To perform sufficient benchmarking of rnaSPAdes and the other aforementioned transcriptome assemblers, we assembled several simulated and publicly available real RNA-Seq datasets from the organisms: "To perform sufficient benchmarking of rnaSPAdes and the other aforementioned transcriptome assemblers, we assembled several simulated and publicly available real RNA-Seq datasets from the organisms"
- [readme] SPAdes is primarily developed for Illumina sequencing data, but can be used for IonTorrent as well.: "SPAdes is primarily developed for Illumina sequencing data, but can be used for IonTorrent as well."
