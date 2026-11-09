---
name: assembly-gap-filling-with-raw-read-sequence
description: Use when a long-read-scaffolded assembly contains runs of ambiguous Ns
  at contig joins and you want to replace those scaffold gaps with actual raw read
  sequence, then quantify how well gaps were filled.
when_to_use_negative:
- The assembly has no scaffold gaps or N runs to fill — plain ntLink scaffolding with
  overlap detection (Basic Protocol 1) is sufficient
- You only need to correct misassemblies in a draft assembly using ntLink mappings
  — use Tigmint-long (Alternate Protocol 2) instead
- Input is short-read data — ntLink is a long-read (e.g. ONT) minimizer-based scaffolder,
  not a short-read gap closer
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: ntLink
  role: Minimizer-based long-read scaffolder run in gap_fill mode (with overlap=True)
    to join contigs and fill scaffold gaps with raw read sequence
  repo: https://github.com/bcgsc/ntLink
- name: abyss-fac (ABySS)
  role: Computes contiguity and N-content statistics on the gap-filled scaffolds
  repo: https://github.com/BirolLab/abyss
- name: QUAST
  role: 'Runs with --split-scaffold to report # N''s per 100 kbp and separate Contig-level
    vs Scaffold-level NG50/NGA50 metrics for gap-quality assessment'
  repo: https://github.com/ablab/quast
- name: Flye
  role: Produces the draft long-read assembly (C. elegans) that serves as the gap-filling
    input
- name: ncbi/sra-tools
  role: Source for downloading the raw ONT reads (SRR10028109) used for gap-filling
  repo: https://github.com/ncbi/sra-tools
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/cpz1.733
    title: 'ntLink: A Toolkit for <i>De Novo</i> Genome Assembly Scaffolding and Mapping
      Using Long Reads'
  source_content_hash: sha256:08a5afc707ed704efc89a183a145df093a983d811a1f3ed0164aec55bbb127e4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bcgsc/ntLink
  tools:
  - Flye
  - QUAST
  - abyss-fac (ABySS)
  - ncbi/sra-tools
  - ntLink
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# assembly-gap-filling-with-raw-read-sequence

## Summary

Uses ntLink's gap-filling mode (Basic Protocol 2) to scaffold a draft assembly with long reads and replace scaffold gaps with real raw read sequence instead of ambiguous Ns, enabling accurate assessment of gap quality with abyss-fac and QUAST --split-scaffold statistics.

## When to use

Use when a long-read-scaffolded assembly contains runs of ambiguous Ns at contig joins and you want to replace those scaffold gaps with actual raw read sequence, then quantify how well gaps were filled.

## When NOT to use

- The assembly has no scaffold gaps or N runs to fill — plain ntLink scaffolding with overlap detection (Basic Protocol 1) is sufficient
- You only need to correct misassemblies in a draft assembly using ntLink mappings — use Tigmint-long (Alternate Protocol 2) instead
- Input is short-read data — ntLink is a long-read (e.g. ONT) minimizer-based scaffolder, not a short-read gap closer

## Inputs

- Draft assembly FASTA (e.g. celegans_flye.fa, a Flye assembly of C. elegans)
- Raw long-read FASTQ file (e.g. SRR10028109.fq.gz, C. elegans ONT reads)
- ntLink parameters: k=32, w=100, t=5, overlap=True

## Outputs

- Gap-filled scaffold FASTA containing raw read sequence in place of scaffold gaps
- abyss-fac assembly statistics (contiguity and N-content) for the gap-filled scaffolds
- QUAST --split-scaffold report with # N's per 100 kbp and Contig-level vs Scaffold-level NG50/NGA50

## How to apply

Run ntLink in scaffold gap_fill mode on the draft assembly (e.g. the C. elegans Flye assembly) with the raw ONT reads, setting k=32, w=100, t=5, and overlap=True, since overlap detection is required for the gap_fill feature to work. ntLink computes ordered minimizer sketches of the target contigs and long reads, maps reads to contigs via minimizers, infers ordering/orientation of contig joins, and fills scaffold gaps with raw read sequence rather than inserting Ns. Compute contiguity and N-content statistics on the gap-filled scaffolds with abyss-fac from ABySS (v2.3.0+). Then run QUAST with the --split-scaffold option to obtain # N's per 100 kbp and separate Contig-level vs Scaffold-level NG50/NGA50 metrics. Gap-filling is judged efficient when Contig NG50/NGA50 statistics approach the Scaffold statistics and Ns per 100 kbp decreases, while Scaffold NG50 is maintained (compare against Table 4 values).

## Related tools

- **ntLink** (Minimizer-based long-read scaffolder run in gap_fill mode (with overlap=True) to join contigs and fill scaffold gaps with raw read sequence) — https://github.com/bcgsc/ntLink
- **abyss-fac (ABySS)** (Computes contiguity and N-content statistics on the gap-filled scaffolds) — https://github.com/BirolLab/abyss
- **QUAST** (Runs with --split-scaffold to report # N's per 100 kbp and separate Contig-level vs Scaffold-level NG50/NGA50 metrics for gap-quality assessment) — https://github.com/ablab/quast
- **Flye** (Produces the draft long-read assembly (C. elegans) that serves as the gap-filling input)
- **ncbi/sra-tools** (Source for downloading the raw ONT reads (SRR10028109) used for gap-filling) — https://github.com/ncbi/sra-tools

## Examples

```
ntLink scaffold gap_fill target=celegans_flye.fa reads=SRR10028109.fq.gz k=32 w=100 t=5 overlap=True
```

## Evaluation signals

- The gap-filled FASTA exists and parses as valid FASTA
- abyss-fac statistics on the gap-filled scaffolds reproduce the trend in Table 4: maintained (or improved) NG50 with reduced N content
- QUAST --split-scaffold shows # N's per 100 kbp decreases relative to the pre-gap-fill assembly
- QUAST --split-scaffold Contig-level NG50/NGA50 approach the Scaffold-level NG50/NGA50, indicating most gaps were filled with sequence rather than Ns
- The ntLink command ran with overlap=True; without it the gap_fill feature will not function as intended

## Limitations

- overlap=True is required for gap_fill; running gap_fill without overlap detection will not perform gap-filling
- Gap-filling quality depends on long-read depth and alignment: insufficient read evidence across a join leaves the gap as Ns
- Contig-level vs Scaffold-level statistics must be interpreted via --split-scaffold; ordinary QUAST metrics alone conflate filled and unfilled gaps

## Evidence

- [methods] Gap quality after ntLink gap-filling can be assessed with QUAST --split-scaffold statistics: with efficient gap-filling, Contig NG50/NGA50 statistics approach the Scaffold statistics and the number of Ns per 100 kbp decreases.: [evidence span withheld — anchor status: not_found]
- [methods] Run ntLink scaffold gap_fill target=celegans_flye.fa reads=SRR10028109.fq.gz k=32 w=100 t=5 (overlap=True is required for the gap_fill feature): [evidence span withheld — anchor status: not_found]
- [abstract] Recent improvements to ntLink have added important features such as overlap detection, gap-filling, and in-code scaffolding iterations: "Recent improvements to ntLink have added important features such as overlap detection, gap-filling, and in-code scaffolding iterations"
- [abstract] Instead of using read alignments to identify candidate joins, ntLink utilizes minimizer-based mappings to infer how input sequences should be ordered and oriented into scaffolds: "ntLink utilizes minimizer-based mappings to infer how input sequences should be ordered and oriented into scaffolds"
- [methods] Compute assembly contiguity and N-content statistics on the gap-filled scaffolds using abyss-fac from ABySS (v2.3.0+).: [evidence span withheld — anchor status: not_found]
