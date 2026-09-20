---
name: simulated-read-benchmarking-across-simulators
description: Use when evaluating whether simulated nanopore reads are fit for downstream
  analysis, specifically when reproducing the paper's de novo bacterial assembly benchmark
  across E. coli, K. pneumoniae, M. morganii, P. aeruginosa, and P. mirabilis genomes.
when_to_use_negative:
- When the goal is signal-level fidelity (basecalling accuracy, Q-scores) rather than
  downstream assembly utility — the paper's abstract-level benchmark of median basecalling
  accuracy >99% and Q-scores above 22.8 is a separate evaluation.
- When the input is real experimental read data rather than simulator output; this
  skill specifically compares simulators against a real-read baseline.
- When assessing human variant calling (SNP F1-scores 0.9953-0.9973, indel F1-scores
  0.7862-0.8612) rather than bacterial de novo assembly quality.
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: NanoSimFormer
  role: Generates high-fidelity simulated nanopore signals for the five bacterial
    genomes; the --basecall option produces the FASTQ read sets used as assembly input
  repo: https://github.com/BioinfoSZU/NanoSimFormer
- name: Dorado
  role: Basecalls simulated reads into FASTQ after signal simulation (via NanoSimFormer
    --basecall)
  repo: https://github.com/nanoporetech/dorado
- name: nanoMDBG
  role: Performs de novo bacterial assembly on each simulator's read set using the
    corresponding genome as target
- name: Medaka
  role: Polishes each nanoMDBG assembly to correct residual consensus errors (e.g.
    medaka_consensus with draft assembly and basecalled FASTQ)
  repo: https://github.com/nanoporetech/medaka
- name: QUAST
  role: Evaluates polished assemblies against ground-truth reference genomes, extracting
    mismatches per 100 kbp and genome fraction into quast_report.tsv
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btag402
    title: 'NanoSimFormer: an end-to-end transformer-based nanopore signal simulator
      with basecaller guidance'
  source_content_hash: sha256:5763ee3c81b46aa837be2ccc9ed1de4258292eedb8124fbb0a2afe66ea3706b9
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
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# simulated-read-benchmarking-across-simulators

## Summary

Benchmark reads simulated by NanoSimFormer and competitor simulators by running the same downstream pipeline (nanoMDBG assembly, Medaka polishing, QUAST evaluation) on each simulator's basecalled FASTQ output and comparing assembly quality metrics against real experimental reads.

## When to use

Use when evaluating whether simulated nanopore reads are fit for downstream analysis, specifically when reproducing the paper's de novo bacterial assembly benchmark across E. coli, K. pneumoniae, M. morganii, P. aeruginosa, and P. mirabilis genomes.

## When NOT to use

- When the goal is signal-level fidelity (basecalling accuracy, Q-scores) rather than downstream assembly utility — the paper's abstract-level benchmark of median basecalling accuracy >99% and Q-scores above 22.8 is a separate evaluation.
- When the input is real experimental read data rather than simulator output; this skill specifically compares simulators against a real-read baseline.
- When assessing human variant calling (SNP F1-scores 0.9953-0.9973, indel F1-scores 0.7862-0.8612) rather than bacterial de novo assembly quality.

## Inputs

- Basecalled FASTQ read sets from NanoSimFormer and each competitor simulator (reference-based simulation of five bacterial genomes: E. coli, K. pneumoniae, M. morganii, P. aeruginosa, P. mirabilis)
- Ground-truth bacterial reference genome FASTA for each species (assembly target and QUAST reference)
- Assemblies from real experimental reads for comparison baselines

## Outputs

- Polished nanoMDBG assemblies (Medaka-corrected FASTA) per simulator per genome
- QUAST metrics per genome and simulator (mismatches per 100 kbp, genome fraction)
- quast_report.tsv compiling per-genome, per-simulator assembly quality metrics

## How to apply

Load the basecalled FASTQ read sets produced by NanoSimFormer and each competitor simulator from reference-based simulation over the five bacterial genomes. For each read set, run nanoMDBG de novo assembly using the corresponding genome as target, then polish the assembly with Medaka to correct residual consensus errors. Evaluate each polished assembly with QUAST against the ground-truth reference, extracting mismatches per 100 kbp and genome fraction, and compile per-genome, per-simulator metrics into quast_report.tsv. The validation criterion from the paper is that NanoSimFormer reaches consensus error below one mismatch per 100 kbp with genome fraction 99.98%-100.00%, while competitors show shortfalls such as Squigulator's 1,067 mismatches per 100 kbp on E. coli and 94.8% genome fraction on P. mirabilis. Applying the identical assembly-polish-evaluate chain to every simulator is essential so that differences in metrics reflect simulation fidelity rather than pipeline variation.

## Related tools

- **NanoSimFormer** (Generates high-fidelity simulated nanopore signals for the five bacterial genomes; the --basecall option produces the FASTQ read sets used as assembly input) — https://github.com/BioinfoSZU/NanoSimFormer
- **Dorado** (Basecalls simulated reads into FASTQ after signal simulation (via NanoSimFormer --basecall)) — https://github.com/nanoporetech/dorado
- **nanoMDBG** (Performs de novo bacterial assembly on each simulator's read set using the corresponding genome as target)
- **Medaka** (Polishes each nanoMDBG assembly to correct residual consensus errors (e.g. medaka_consensus with draft assembly and basecalled FASTQ)) — https://github.com/nanoporetech/medaka
- **QUAST** (Evaluates polished assemblies against ground-truth reference genomes, extracting mismatches per 100 kbp and genome fraction into quast_report.tsv)

## Examples

```
medaka_consensus -i simulated_basecalls.fastq -d nanoMDBG_draft_assembly.fasta -o medaka_consensus -t $(nproc) && quast.py medaka_consensus/consensus.fasta -r reference_genome.fasta -o quast_out
```

## Evaluation signals

- NanoSimFormer-derived assemblies show fewer than one mismatch or indel per 100 kbp across all five evaluated bacterial genomes
- NanoSimFormer genome fraction falls within 99.98%-100.00% against the ground-truth reference
- Metrics match assemblies produced from real experimental reads for the same genomes
- Known competitor shortfalls are reproduced, e.g. Squigulator shows 1,067 mismatches per 100 kbp on E. coli and 94.8% genome fraction on P. mirabilis
- quast_report.tsv contains per-genome, per-simulator rows for all five genomes and all simulators, enabling direct side-by-side comparison

## Limitations

- Benchmark validity depends on using the identical assembly-polish-evaluate pipeline for every simulator; differing pipelines confound the comparison.
- The paper's assembly benchmark is demonstrated only on five bacterial genomes; results may not transfer to eukaryotic or metagenomic targets.
- Competitor shortfalls (e.g. Squigulator) are specific to the evaluated conditions and may not reflect performance under other chemistries or coverage levels.

## Evidence

- [results] NanoSimFormer-derived reads yield near-perfect bacterial assemblies with fewer than one mismatch or indel per 100 kbp across all evaluated datasets, matching assemblies from real experimental reads.: [evidence span withheld — anchor status: not_found]
- [abstract] NanoSimFormer-derived reads enable high-quality de novo bacterial assembly with consensus error rates below one mismatch per 100 kbp: "NanoSimFormer-derived reads enable high-quality de novo bacterial assembly with consensus error rates below one mismatch per 100 kbp"
- [methods] Run nanoMDBG de novo assembly on each simulator's read set using the corresponding bacterial genome as the assembly target.: [evidence span withheld — anchor status: not_found]
- [results] The report reproduces the paper's criteria — NanoSimFormer consensus error below one mismatch per 100 kbp with genome fraction 99.98%-100.00%, while competitors show shortfalls such as Squigulator's 1,067 mismatches per 100 kbp on E. coli: [evidence span withheld — anchor status: not_found]
- [readme] medaka can be run using its default settings through the medaka_consensus program. An assembly in .fasta format and basecalls in .fasta or .fastq formats are required.: [evidence span withheld — anchor status: external_doc]
