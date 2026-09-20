---
name: bacterial-de-novo-genome-assembly
description: Use when benchmarking de novo bacterial genome assembly from basecalled
  nanopore FASTQ reads, e.g., comparing simulator-derived vs. real reads by QUAST
  mismatches per 100 kbp and genome fraction.
when_to_use_negative:
- Input is raw POD5 signal that has not been basecalled — assembly and Medaka polishing
  require FASTQ/FASTA reads, so Dorado basecalling must run first.
- The goal is human-sample variant-calling benchmarking (SNP/indel F1-scores across
  five human samples) rather than bacterial genome reconstruction — the paper evaluates
  that with a separate pipeline.
- The input is already a polished assembly or reference FASTA — this skill consumes
  read-level FASTQ; assemblies are its intermediate product, not its input.
edam_operation: http://edamontology.org/operation_0524
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: NanoSimFormer
  role: Generates synthetic nanopore signals from reference bacterial genomes and,
    with --basecall, produces the Dorado-basecalled FASTQ read sets that feed the
    assembly benchmark.
  repo: https://github.com/BioinfoSZU/NanoSimFormer
- name: Dorado
  role: Basecalls NanoSimFormer-simulated signals into the FASTQ read sets assembled
    de novo by nanoMDBG.
  repo: https://github.com/nanoporetech/dorado
- name: nanoMDBG
  role: De novo assembles each basecalled FASTQ read set into a draft bacterial genome
    using the corresponding genome as the assembly target.
- name: Medaka
  role: Polishes nanoMDBG drafts via medaka_consensus to correct residual consensus
    errors before QUAST evaluation.
  repo: https://github.com/nanoporetech/medaka
- name: QUAST
  role: Evaluates polished assemblies against ground-truth reference genomes, extracting
    mismatches per 100 kbp and genome fraction for quast_report.tsv.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btag402
    title: 'NanoSimFormer: an end-to-end transformer-based nanopore signal simulator
      with basecaller guidance'
  source_content_hash: sha256:c0d4f0bfae905504481daecfb2c9e8bed566c2b97fa10e7c0763cdfb42675698
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioinfoSZU/NanoSimFormer
  tools:
  - Dorado
  - Medaka
  - NanoSimFormer
  - QUAST
  - nanoMDBG
  edam_operation: http://edamontology.org/operation_0524
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# bacterial-de-novo-genome-assembly

## Summary

De novo assembly of complete bacterial genomes from basecalled nanopore FASTQ reads using nanoMDBG, followed by Medaka consensus polishing and QUAST evaluation (mismatches per 100 kbp, genome fraction). Used to test whether simulator-derived reads (e.g., NanoSimFormer) yield assembly quality matching real experimental reads.

## When to use

Use when benchmarking de novo bacterial genome assembly from basecalled nanopore FASTQ reads, e.g., comparing simulator-derived vs. real reads by QUAST mismatches per 100 kbp and genome fraction.

## When NOT to use

- Input is raw POD5 signal that has not been basecalled — assembly and Medaka polishing require FASTQ/FASTA reads, so Dorado basecalling must run first.
- The goal is human-sample variant-calling benchmarking (SNP/indel F1-scores across five human samples) rather than bacterial genome reconstruction — the paper evaluates that with a separate pipeline.
- The input is already a polished assembly or reference FASTA — this skill consumes read-level FASTQ; assemblies are its intermediate product, not its input.

## Inputs

- Basecalled nanopore FASTQ read set per simulator x genome (NanoSimFormer signal simulation with Dorado basecalling via --basecall)
- Ground-truth bacterial reference genome FASTA for each of E. coli, K. pneumoniae, M. morganii, P. aeruginosa, P. mirabilis
- nanoMDBG draft assembly FASTA (intermediate input to Medaka polishing)

## Outputs

- Polished bacterial genome assembly FASTA per genome x simulator
- QUAST metrics per polished assembly: mismatches per 100 kbp and genome fraction vs. ground-truth reference
- quast_report.tsv compiling per-genome, per-simulator assembly quality metrics

## How to apply

Load the basecalled FASTQ read sets — NanoSimFormer signals produced with the `--basecall` option are automatically basecalled by Dorado into FASTQ — for each bacterial genome (E. coli, K. pneumoniae, M. morganii, P. aeruginosa, P. mirabilis) and each simulator. Run nanoMDBG de novo assembly on each read set targeting the corresponding genome. Polish each nanoMDBG draft with Medaka (`medaka_consensus`), which builds consensus from a pileup of basecalled .fastq reads against the .fasta draft, to correct residual consensus errors before judging simulator fidelity. Evaluate every polished assembly with QUAST against its ground-truth reference genome, extracting mismatches per 100 kbp and genome fraction, then compile per-genome, per-simulator metrics into quast_report.tsv. Judge success against the paper's criteria: NanoSimFormer consensus error below one mismatch per 100 kbp with genome fraction 99.98%–100.00%, while competitor simulators show shortfalls such as Squigulator's 1,067 mismatches per 100 kbp on E. coli and 94.8% genome fraction on P. mirabilis.

## Related tools

- **NanoSimFormer** (Generates synthetic nanopore signals from reference bacterial genomes and, with --basecall, produces the Dorado-basecalled FASTQ read sets that feed the assembly benchmark.) — https://github.com/BioinfoSZU/NanoSimFormer
- **Dorado** (Basecalls NanoSimFormer-simulated signals into the FASTQ read sets assembled de novo by nanoMDBG.) — https://github.com/nanoporetech/dorado
- **nanoMDBG** (De novo assembles each basecalled FASTQ read set into a draft bacterial genome using the corresponding genome as the assembly target.)
- **Medaka** (Polishes nanoMDBG drafts via medaka_consensus to correct residual consensus errors before QUAST evaluation.) — https://github.com/nanoporetech/medaka
- **QUAST** (Evaluates polished assemblies against ground-truth reference genomes, extracting mismatches per 100 kbp and genome fraction for quast_report.tsv.)

## Examples

```
medaka_consensus -i nanosimformer_reads.fastq -d nanomdbg_draft/assm_final.fa -o medaka_consensus -t $(nproc)
```

## Evaluation signals

- QUAST reports fewer than one mismatch (or indel) per 100 kbp for NanoSimFormer-derived assemblies on all five bacterial genomes.
- Genome fraction for NanoSimFormer read sets falls within 99.98%-100.00% of each reference genome.
- Known competitor shortfalls are reproduced, e.g., Squigulator shows ~1,067 mismatches per 100 kbp on E. coli and 94.8% genome fraction on P. mirabilis.
- quast_report.tsv contains one entry per genome x simulator with mismatches per 100 kbp and genome fraction populated.
- Medaka polishing reduces QUAST mismatch counts relative to the unpolished nanoMDBG draft (residual consensus errors corrected).

## Limitations

- The <1 mismatch per 100 kbp and 99.98%-100.00% genome-fraction criteria were established on five bacterial genomes; they do not automatically transfer to larger or eukaryotic assemblies.
- Assembly quality reflects simulator fidelity — competitor simulators (e.g., Squigulator) yield substantially worse consensus error and genome fraction than NanoSimFormer.
- Medaka can hit GPU out-of-memory errors during inference; per the README, reduce the batch size (e.g., -b 100 for 11 GB GPUs) when running medaka_consensus.

## Evidence

- [abstract] NanoSimFormer-derived reads enable high-quality de novo bacterial assembly with consensus error rates below one mismatch per 100 kbp and maintain high correlations with experimental abundance in: "NanoSimFormer-derived reads enable high-quality de novo bacterial assembly with consensus error rates below one mismatch per 100 kbp and maintain high correlations with experimental abundance in"
- [methods] The `--basecall` option automatically runs Dorado to basecall the simulated reads into a FASTQ file after signal simulation.: [evidence span withheld — anchor status: not_found]
- [readme] medaka is a tool to create consensus sequences and variant calls from nanopore sequencing data.: [evidence span withheld — anchor status: not_found]
