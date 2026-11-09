---
name: long-read-assembly-polishing
description: Use when you need to test whether basecalled FASTQ reads (e.g., from
  a nanopore signal simulator or real ONT run) yield near-perfect bacterial genome
  assemblies, judged by QUAST mismatches per 100 kbp and genome fraction.
when_to_use_negative:
- The input is already a polished assembly or a QUAST report — this skill builds and
  polishes assemblies from basecalled reads.
- The goal is read-level basecalling accuracy (median identity, Q-scores) rather than
  consensus-level assembly quality — use the simulator's read-level benchmarking instead.
- The reference genome is not a small bacterial genome; the paper's assembly benchmark
  was defined and validated only on the five bacterial genomes.
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: NanoSimFormer
  role: Generates the synthetic nanopore signals and, via --basecall, the basecalled
    FASTQ reads that feed the assembly benchmark
  repo: https://github.com/BioinfoSZU/NanoSimFormer
- name: Dorado
  role: Basecalls the simulated signals into FASTQ reads used as assembly input
  repo: github.com/nanoporetech/dorado
- name: nanoMDBG
  role: Performs de novo bacterial assembly from each simulator's read set
- name: Medaka
  role: Polishes each nanoMDBG draft assembly to correct residual consensus errors
  repo: github.com/nanoporetech/medaka
- name: QUAST
  role: Evaluates polished assemblies against reference genomes, extracting mismatches
    per 100 kbp and genome fraction
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btag402
    title: 'NanoSimFormer: an end-to-end transformer-based nanopore signal simulator
      with basecaller guidance'
  source_content_hash: sha256:bf16e6420ecd825b4888280e28909b80aa310dc92961a6b7078fd53574af54e3
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
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9473684210526315
n_claims: 19
---

# long-read-assembly-polishing

## Summary

Benchmark de novo long-read bacterial assembly quality from simulated nanopore reads by running nanoMDBG assembly, Medaka consensus polishing, and QUAST evaluation, validating that simulator-derived reads reproduce the assembly fidelity of real experimental data.

## When to use

Use when you need to test whether basecalled FASTQ reads (e.g., from a nanopore signal simulator or real ONT run) yield near-perfect bacterial genome assemblies, judged by QUAST mismatches per 100 kbp and genome fraction.

## When NOT to use

- The input is already a polished assembly or a QUAST report — this skill builds and polishes assemblies from basecalled reads.
- The goal is read-level basecalling accuracy (median identity, Q-scores) rather than consensus-level assembly quality — use the simulator's read-level benchmarking instead.
- The reference genome is not a small bacterial genome; the paper's assembly benchmark was defined and validated only on the five bacterial genomes.

## Inputs

- Basecalled FASTQ read sets from NanoSimFormer (or competitor simulators) simulated from bacterial reference genomes
- Ground-truth bacterial reference genome FASTA files (E. coli, K. pneumoniae, M. morganii, P. aeruginosa, P. mirabilis)

## Outputs

- nanoMDBG draft assemblies (FASTA) per simulator per genome
- Medaka-polished consensus assemblies (FASTA)
- quast_report.tsv with per-genome, per-simulator mismatches per 100 kbp and genome fraction

## How to apply

Load the basecalled FASTQ read sets — produced by NanoSimFormer with Dorado basecalling, or by competitor simulators — generated from five bacterial reference genomes (E. coli, K. pneumoniae, M. morganii, P. aeruginosa, P. mirabilis). Run nanoMDBG de novo assembly on each read set against its corresponding genome, then polish each assembly with Medaka (medaka_consensus on the draft FASTA plus basecalled FASTQ) to correct residual consensus errors. Evaluate every polished assembly with QUAST against the ground-truth reference, extracting mismatches per 100 kbp and genome fraction into quast_report.tsv. Success criterion from the paper: NanoSimFormer reads give consensus error below one mismatch per 100 kbp with genome fraction 99.98%–100.00%, matching real experimental reads; competitor shortfalls (e.g., Squigulator's 1,067 mismatches per 100 kbp on E. coli, 94.8% genome fraction on P. mirabilis) flag lower-fidelity simulation.

## Related tools

- **NanoSimFormer** (Generates the synthetic nanopore signals and, via --basecall, the basecalled FASTQ reads that feed the assembly benchmark) — https://github.com/BioinfoSZU/NanoSimFormer
- **Dorado** (Basecalls the simulated signals into FASTQ reads used as assembly input) — github.com/nanoporetech/dorado
- **nanoMDBG** (Performs de novo bacterial assembly from each simulator's read set)
- **Medaka** (Polishes each nanoMDBG draft assembly to correct residual consensus errors) — github.com/nanoporetech/medaka
- **QUAST** (Evaluates polished assemblies against reference genomes, extracting mismatches per 100 kbp and genome fraction)

## Examples

```
medaka_consensus -i basecalls.fastq -d assm_final.fa -o medaka_consensus -t $(nproc)
```

## Evaluation signals

- QUAST reports fewer than one mismatch or indel per 100 kbp for NanoSimFormer-derived assemblies across all five bacterial genomes
- Genome fraction falls within 99.98%–100.00% for NanoSimFormer assemblies, matching assemblies from real experimental reads
- Competitor simulators show reproducible shortfalls, e.g., Squigulator yields ~1,067 mismatches per 100 kbp on E. coli and 94.8% genome fraction on P. mirabilis
- quast_report.tsv contains one row per genome-per-simulator combination with mismatches-per-100-kbp and genome-fraction columns populated
- Medaka polishing step completes (consensus FASTA produced) for every nanoMDBG draft before QUAST evaluation

## Limitations

- The assembly benchmark was established only on five bacterial genomes; extrapolation to larger or repetitive eukaryotic genomes is not validated by this study.
- Consensus quality depends on upstream simulation fidelity; simulators with homopolymer or STR false positives (as noted for previous simulators) degrade assembly metrics.
- Medaka requires samtools, minimap2, tabix, and bgzip (>=1.14 for samtools, >=2.17 for minimap2) on PATH unless installed via the from-source or conda route; GPU mode may need batch-size reduction (-b 100 for 11 GB GPUs).

## Evidence

- [abstract] NanoSimFormer-derived reads enable high-quality de novo bacterial assembly with consensus error rates below one mismatch per 100 kbp: "NanoSimFormer-derived reads enable high-quality de novo bacterial assembly with consensus error rates below one mismatch per 100 kbp"
- [readme] Medaka is a tool to create consensus sequences and variant calls from nanopore sequencing data.: "`medaka` is a tool to create consensus sequences and variant calls from
nanopore sequencing data."
- [methods] The --basecall option automatically runs Dorado to basecall the simulated reads into a FASTQ file after signal simulation.: [evidence span withheld — anchor status: not_found]
- [abstract] Compared with previous simulators, NanoSimFormer also substantially reduces false positives in homopolymer and short tandem repeat regions.: "Compared with previous simulators, NanoSimFormer also substantially reduces false positives in homopolymer and short tandem repeat regions."
- [readme] medaka_consensus -i ${BASECALLS} -d ${DRAFT} -o ${OUTDIR} -t ${NPROC}: [evidence span withheld — anchor status: external_doc]
