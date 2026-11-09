---
name: assembly-quality-evaluation-with-quast
description: Use when FASTQ read sets (simulated or experimental nanopore) have been
  assembled de novo and consensus quality must be scored per genome against a ground-truth
  reference to judge simulator or sequencer fidelity.
when_to_use_negative:
- The input is raw signal (POD5) or basecalled reads that have not yet been assembled
  de novo — assembly must precede QUAST evaluation.
- The goal is read-level basecalling accuracy or variant-calling F1 evaluation rather
  than assembly consensus quality.
- No ground-truth reference genome is available, since QUAST metrics here (mismatches
  per 100 kbp, genome fraction) are reference-based.
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: nanoMDBG
  role: Performs de novo assembly of each simulator's basecalled FASTQ read set against
    the bacterial genome target
- name: Medaka
  role: Polishes each nanoMDBG draft assembly (medaka_consensus) to correct residual
    consensus errors before QUAST evaluation
  repo: https://github.com/nanoporetech/medaka
- name: QUAST
  role: Evaluates polished assemblies against ground-truth references, extracting
    mismatches per 100 kbp and genome fraction
- name: Dorado
  role: Basecalls the simulated reads into FASTQ, the input read set for assembly
  repo: https://github.com/nanoporetech/dorado
- name: NanoSimFormer
  role: Generates the simulated nanopore signals whose basecalled reads are assembled
    and benchmarked
  repo: https://github.com/BioinfoSZU/NanoSimFormer
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btag402
    title: 'NanoSimFormer: an end-to-end transformer-based nanopore signal simulator
      with basecaller guidance'
  source_content_hash: sha256:fd00eb6515b49d9ea53592b3635e9cdda78833b16f4ee02bd8b8fdec756fc383
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nanoporetech/medaka
  tools:
  - Dorado
  - Medaka
  - NanoSimFormer
  - QUAST
  - nanoMDBG
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# assembly-quality-evaluation-with-quast

## Summary

Benchmark de novo bacterial assemblies built from simulated or real nanopore reads by polishing nanoMDBG assemblies with Medaka and evaluating QUAST metrics (mismatches per 100 kbp, genome fraction) against the ground-truth reference genome, as used to validate NanoSimFormer read fidelity.

## When to use

Use when FASTQ read sets (simulated or experimental nanopore) have been assembled de novo and consensus quality must be scored per genome against a ground-truth reference to judge simulator or sequencer fidelity.

## When NOT to use

- The input is raw signal (POD5) or basecalled reads that have not yet been assembled de novo — assembly must precede QUAST evaluation.
- The goal is read-level basecalling accuracy or variant-calling F1 evaluation rather than assembly consensus quality.
- No ground-truth reference genome is available, since QUAST metrics here (mismatches per 100 kbp, genome fraction) are reference-based.

## Inputs

- Basecalled FASTQ read sets produced by NanoSimFormer (or competitor simulators) for five bacterial genomes (E. coli, K. pneumoniae, M. morganii, P. aeruginosa, P. mirabilis)
- Ground-truth bacterial reference genome FASTA for each dataset
- nanoMDBG draft assembly FASTA per read set
- Medaka-polished consensus FASTA per assembly

## Outputs

- Polished consensus FASTA per genome/simulator combination
- quast_report.tsv with per-genome, per-simulator QUAST metrics (mismatches per 100 kbp, genome fraction)

## How to apply

Run nanoMDBG de novo assembly on each basecalled FASTQ read set using the corresponding bacterial genome as the assembly target, then polish each draft assembly with Medaka (medaka_consensus) to correct residual consensus errors. Evaluate every polished assembly with QUAST against its ground-truth reference, extracting mismatches per 100 kbp and genome fraction, and compile per-genome, per-simulator metrics into quast_report.tsv. The article's acceptance criteria are consensus error below one mismatch per 100 kbp with genome fraction 99.98%-100.00% for NanoSimFormer-derived reads, mirroring assemblies from real experimental reads. Decision point: assemblies falling short of these thresholds (e.g. competitor simulators) indicate consensus-level simulation artifacts, as seen with Squigulator's 1,067 mismatches per 100 kbp on E. coli and 94.8% genome fraction on P. mirabilis.

## Related tools

- **nanoMDBG** (Performs de novo assembly of each simulator's basecalled FASTQ read set against the bacterial genome target)
- **Medaka** (Polishes each nanoMDBG draft assembly (medaka_consensus) to correct residual consensus errors before QUAST evaluation) — https://github.com/nanoporetech/medaka
- **QUAST** (Evaluates polished assemblies against ground-truth references, extracting mismatches per 100 kbp and genome fraction)
- **Dorado** (Basecalls the simulated reads into FASTQ, the input read set for assembly) — https://github.com/nanoporetech/dorado
- **NanoSimFormer** (Generates the simulated nanopore signals whose basecalled reads are assembled and benchmarked) — https://github.com/BioinfoSZU/NanoSimFormer

## Examples

```
medaka_consensus -i simulated_basecalls.fastq -d nanoMDBG_draft.fasta -o medaka_consensus; quast.py -r reference.fasta -o quast_out polished_consensus.fasta
```

## Evaluation signals

- NanoSimFormer-derived assemblies show consensus error below one mismatch per 100 kbp on all five bacterial genomes
- Genome fraction falls within 99.98%-100.00% for NanoSimFormer assemblies, matching assemblies from real experimental reads
- quast_report.tsv contains per-genome, per-simulator rows covering all five genomes and all evaluated simulators
- Competitor benchmarks behave as documented, e.g. Squigulator yields 1,067 mismatches per 100 kbp on E. coli and 94.8% genome fraction on P. mirabilis
- Every QUAST evaluation is performed on a Medaka-polished assembly (not the raw nanoMDBG draft) aligned to the correct ground-truth reference

## Limitations

- Validation scope is limited to the five evaluated bacterial genomes (E. coli, K. pneumoniae, M. morganii, P. aeruginosa, P. mirabilis); behavior on other taxa is not established by this workflow
- Benchmark quality depends on upstream simulation fidelity — competitor simulators such as Squigulator fail the <1 mismatch/100 kbp and near-complete genome-fraction criteria
- Raw nanoMDBG drafts must be Medaka-polished before QUAST comparison, since the acceptance thresholds apply to polished consensus sequences

## Evidence

- [other] Polish each nanoMDBG assembly with Medaka to correct residual consensus errors.: [evidence span withheld — anchor status: not_found]
- [other] NanoSimFormer consensus error below one mismatch per 100 kbp: [evidence span withheld — anchor status: not_found]
- [abstract] NanoSimFormer-derived reads enable high-quality de novo bacterial assembly: "NanoSimFormer-derived reads enable high-quality de novo bacterial assembly with consensus error rates below one mismatch per 100 kbp"
- [readme] medaka is a tool to create consensus sequences: "`medaka` is a tool to create consensus sequences and variant calls from
nanopore sequencing data. This task is performed using neural networks applied
a pileup of individual sequencing reads"
