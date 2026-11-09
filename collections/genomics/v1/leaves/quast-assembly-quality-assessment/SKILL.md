---
name: quast-assembly-quality-assessment
description: Use when a hybrid-scaffolding pipeline (npScarf) has produced final scaffolds
  (prefix.fin.fasta) and you must quantify contig number, N50 and completeness against
  competitor methods or reference values.
when_to_use_negative:
- Input is not a finished scaffold/draft assembly FASTA (e.g. raw nanopore FASTQ,
  SAM alignments or SPAdes graph output) — QUAST evaluates assembled sequences, not
  reads or alignments.
- You only need to monitor scaffolding progress during streaming; npScarf itself reports
  assembly metrics in real-time, and QUAST is a post-hoc evaluation of final files.
- You are evaluating long-read-only assemblers' internal stages rather than comparing
  final hybrid-scaffolding assemblies against paper-reported contig/N50 values.
edam_operation: http://edamontology.org/operation_0544
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: QUAST
  role: Assembly quality assessment tool producing contig count, N50 and completeness
    reports for npScarf scaffolds and competitor assemblies
- name: npScarf
  role: Producer of the scaffold FASTA files (<prefix>.fin.fasta) that are the subject
    of the QUAST evaluation
  repo: https://github.com/mdcao/npScarf
- name: SPAdes
  role: Produces the pre-scaffolding SPAdes contigs draft (spades/contigs.fasta) used
    as the comparison baseline
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_001
  - task_002
  source_papers:
  - doi: 10.1038/ncomms14515
    title: Scaffolding and completing genome assemblies in real-time with nanopore
      sequencing
  source_content_hash: sha256:ac282533aa2722f16864139d78791b7e2fe3fe1aee0198fb3a45ac6c180a6094
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mdcao/npScarf
  tools:
  - BWA-MEM
  - QUAST
  - SPAdes
  - japsa
  - jsa.np.timeEmulate
  - jsa.seq.sort
  - npReader
  - npScarf
  - trimmomatic
  edam_operation: http://edamontology.org/operation_0544
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# quast-assembly-quality-assessment

## Summary

Run QUAST on npScarf scaffolded assemblies (prefix.fin.fasta) and on competitor assemblies to obtain contig counts, N50 and completeness metrics used to validate reproduction of the paper's reported results (e.g. 19 contigs/N50 913 Kb for S. cerevisiae W303, 9 contigs/N50 864 kb for S. Typhi H58).

## When to use

Use when a hybrid-scaffolding pipeline (npScarf) has produced final scaffolds (prefix.fin.fasta) and you must quantify contig number, N50 and completeness against competitor methods or reference values.

## When NOT to use

- Input is not a finished scaffold/draft assembly FASTA (e.g. raw nanopore FASTQ, SAM alignments or SPAdes graph output) — QUAST evaluates assembled sequences, not reads or alignments.
- You only need to monitor scaffolding progress during streaming; npScarf itself reports assembly metrics in real-time, and QUAST is a post-hoc evaluation of final files.
- You are evaluating long-read-only assemblers' internal stages rather than comparing final hybrid-scaffolding assemblies against paper-reported contig/N50 values.

## Inputs

- npScarf final scaffold FASTA files (e.g. <prefix>.fin.fasta such as Kp2146-realtime.fin.fasta, EcK12S.fin.fasta)
- Competitor-method assemblies (e.g. Nanocorr, LINKS, NaS, npGraph outputs) for side-by-side comparison
- Reference genome (e.g. E. coli K12 MG1655 reference genome, S. cerevisiae W303 reference) for completeness evaluation
- Pre-scaffolding SPAdes draft assembly (Kp2146_spades.fasta) as baseline

## Outputs

- QUAST HTML report (e.g. quast_w303_report.html / report.html) with contig count, N50 and assembly size per assembly
- Side-by-side comparison table of npScarf vs competitive-method assemblies
- Validated scaffold metrics matching the deposited npScarf QUAST reports

## How to apply

After npScarf scaffolding finishes, run QUAST on the final scaffold FASTA files (e.g. Kp2146-realtime.fin.fasta, StH58-realtime/ StH58-batch .fin.fasta, EcK12S.fin.fasta) together with the competitive-method assemblies so contig counts, N50 and assembly sizes are computed on a common basis. Compare the QUAST report metrics against the paper-reported values: npScarf real-time W303 (19 contigs, N50 ≈ 913 Kb, circularized mitochondrial contig), S. Typhi H58 real-time (9 contigs, N50 ~864 kb) vs batch (8 contigs) vs the 34-contig pre-scaffolding draft (N50 319 kb), K. pneumoniae ATCC BAA-2146 (5 contigs, N50 5.4 Mb, no misassembly) and E. coli K12 MG1655 (one circular contig at ≤30-fold nanopore coverage). Save the comparison report (e.g. quast_w303_report.html) and, for W303, check against the reference values published at the npScarf results site (QUAST/W303/report.html). Correct application means the QUAST metrics reproduce the deposited report at data.genomicsresearch.org/Projects/npScarf/results/QUAST/<sample>/report.html.

## Related tools

- **QUAST** (Assembly quality assessment tool producing contig count, N50 and completeness reports for npScarf scaffolds and competitor assemblies)
- **npScarf** (Producer of the scaffold FASTA files (<prefix>.fin.fasta) that are the subject of the QUAST evaluation) — https://github.com/mdcao/npScarf
- **SPAdes** (Produces the pre-scaffolding SPAdes contigs draft (spades/contigs.fasta) used as the comparison baseline)

## Examples

```
quast.py Kp2146-realtime.fin.fasta Kp2146-batch.fin.fasta -o quast_kp2146_report
```

## Evaluation signals

- W303: QUAST report reproduces 19 contigs with N50 ≈ 913 Kb and a circularized mitochondrial contig, matching the deposited npScarf results site (QUAST/W303/report.html).
- S. Typhi H58: real-time assembly collapses the draft to 9 contigs and batch to 8 contigs with N50 ~864 kb, consistent with the published StH58 QUAST report.
- K. pneumoniae Kpn2146: Kp2146-realtime.fin.fasta exists and its QUAST metrics are consistent with the deposited report at data.genomicsresearch.org/Projects/npScarf/results/QUAST/Kp2146/report.html.
- E. coli K12 MG1655: report.html shows one complete circular contig matching the reference genome, achieved with fewer than 30-fold nanopore coverage.
- N50 of npScarf scaffolds must exceed the next best competitor (e.g. 913 Kb vs Nanocorr's 600 Kb for W303; 864 kb vs the 319 kb of the 34-contig draft).

## Limitations

- QUAST evaluates post-hoc; it cannot provide the real-time assembly-metric reporting that lets you terminate a sequencing run once sufficient quality is obtained — that is npScarf's own -realtime output.
- Validation depends on matching published reference values at data.genomicsresearch.org/Projects/npScarf/results/QUAST/<sample>/report.html; deviations may stem from upstream parameter changes (e.g. SPAdes version, bwa mem parameter set) rather than QUAST itself.
- npScarf is no longer maintained; its replacement npGraph may produce assemblies whose comparison requires a different evaluation setup.

## Evidence

- [other] Validation: the QUAST report for W303 must reproduce the paper-reported npScarf assembly statistics (19 contigs, N50 ≈ 913 Kb, circular mitochondrial contig): [evidence span withheld — anchor status: not_found]
- [other] verify the real-time assembly collapses the draft to 9 contigs and the batch assembly to 8 contigs with an N50 of ~864 kb: [evidence span withheld — anchor status: not_found]
- [methods] Data and results from npScarf presented in the paper are made available following: "Data and results from npScarf presented in the paper are made available following"
- [abstract] It reports assembly metrics in real-time so the sequencing run can be terminated once an assembly of sufficient quality is obtained.: "It reports assembly metrics in real-time so the sequencing run can be terminated once an assembly of sufficient quality is obtained."
- [other] npScarf required only 20-fold coverage (<120 Mb) of MinION data to complete the K. pneumoniae ATCC BAA-2146 assembly, reducing it to five contigs: [evidence span withheld — anchor status: not_found]
