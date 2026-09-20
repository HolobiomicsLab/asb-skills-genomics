---
name: hla-prediction-validation-against-clinical-types
description: Use when HLAminer HPRA predictions have been generated from long-read
  WGS (PacBio HiFi or ONT PromethION) and a clinically confirmed HLA type list is
  available to benchmark prediction concordance.
when_to_use_negative:
- Input is Illumina-sized short reads, where the HPTASR targeted-assembly route (reads
  100 bp and up, ideally 150 bp) is the recommended HLAminer mode
- No clinically confirmed HLA reference types exist for the sample, so there is nothing
  to benchmark predictions against
- Input reads are already assembled contigs or a published HLA call set rather than
  raw WGS long reads
edam_operation: http://edamontology.org/operation_0452
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: HLAminer
  role: Scores streamed alignments and emits ranked HLA-I/II allele predictions (HPRA
    mode) that are validated against clinical types
  repo: https://github.com/bcgsc/HLAminer
- name: minimap2
  role: Aligns concatenated PacBio HiFi reads with -ax map-hifi --MD and streams SAM
    output directly into HLAminer
  repo: https://github.com/lh3/minimap2
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/cpz1.70124
    title: Streaming Long‐Read Sequence Alignments for HLA Predictions Using HLAminer
  source_content_hash: sha256:e6b1661492327271ef6d155ad70762241980768e15b08966d64b4556c3053c04
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bcgsc/HLAminer
  tools:
  - HLAminer
  - minimap2
  edam_operation: http://edamontology.org/operation_0452
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# hla-prediction-validation-against-clinical-types

## Summary

Validates HLAminer HPRA HLA-I/II predictions from streamed long-read alignments against clinically confirmed HLA types, checking that top-ranked alleles (e.g. B*35:01, DRB1*04:03P) appear in HLAminer_HPRA.csv and that runtime scales as reported (~1 hr 14 min at 30× PacBio HiFi).

## When to use

Use when HLAminer HPRA predictions have been generated from long-read WGS (PacBio HiFi or ONT PromethION) and a clinically confirmed HLA type list is available to benchmark prediction concordance.

## When NOT to use

- Input is Illumina-sized short reads, where the HPTASR targeted-assembly route (reads 100 bp and up, ideally 150 bp) is the recommended HLAminer mode
- No clinically confirmed HLA reference types exist for the sample, so there is nothing to benchmark predictions against
- Input reads are already assembled contigs or a published HLA call set rather than raw WGS long reads

## Inputs

- Concatenated PacBio CCS/HiFi FASTQ read set (e.g. 39 NA24385 subread files, ~30× coverage)
- SAM alignment stream from minimap2 -ax map-hifi --MD against GRCh38 (GCA_000001405.15) with chr6 masked plus HLA-I/II genomic alleles
- Clinically confirmed HLA type list for the individual (e.g. NA24385)
- HLAminer databases: HLA-I_II_GEN.fasta and hla_nom_p.txt (P designation file)

## Outputs

- HLAminer_HPRA.csv (top two highest-scoring predictions per HLA gene)
- HLAminer_HPRA.log (command, alignments and scores for all predictions)
- Concordance comparison of ranked predictions vs. clinical HLA types

## How to apply

Run the streamed minimap2 -ax map-hifi --MD → HLAminer.pl HPRA pipeline (-a stream, -h ../database/HLA-I_II_GEN.fasta, -s 500, -q 1, -i 1, -p ../database/hla_nom_p.txt) and collect HLAminer_HPRA.csv and HLAminer_HPRA.log. From the CSV, take the top two highest-scoring predictions per HLA gene, noting that identical antigen-binding domains are collapsed to the 'P' designated allele. Compare these ranked calls against the individual's clinically confirmed HLA types, checking that expected alleles such as B*35:01 and DRB1*04:03P for NA24385 appear among the top predictions. Accept the run when runtime is in line with the reported ~1 hr 14 min at 30× coverage and the reported clinical types are reproduced; expect near-miss allele-level discrepancies (e.g. B*35:08, DRB1*04:02P clinically) to resolve at P-designation resolution.

## Related tools

- **HLAminer** (Scores streamed alignments and emits ranked HLA-I/II allele predictions (HPRA mode) that are validated against clinical types) — https://github.com/bcgsc/HLAminer
- **minimap2** (Aligns concatenated PacBio HiFi reads with -ax map-hifi --MD and streams SAM output directly into HLAminer) — https://github.com/lh3/minimap2

## Examples

```
minimap2 -ax map-hifi --MD GRCh38_noChr6_HLA-I_II_GEN.fa NA24385_hifi.fq | ../bin/HLAminer.pl -a stream -h ../database/HLA-I_II_GEN.fasta -s 500 -q 1 -i 1 -p ../database/hla_nom_p.txt
```

## Evaluation signals

- HLAminer_HPRA.csv contains only the top two highest-scoring predictions per HLA gene, with identical antigen-binding-domain alleles collapsed to the P designation
- Expected clinical alleles (e.g. B*35:01 and DRB1*04:03P for NA24385) appear among the top-ranked predictions in the CSV
- Runtime is in line with the reported ~1 hr 14 min for the 30× NA24385 PacBio HiFi run
- HLAminer_HPRA.log records the command, alignments and scores used to derive each prediction

## Limitations

- Predictions may resolve only at P-designation resolution: B*35:01 predicted vs. B*35:08 clinically typed, and DRB1*04:03P vs. DRB1*04:02P, because shared antigen-binding domains are collapsed
- Robustness is demonstrated for relatively low (10×) coverage and older, less accurate WGS nanopore datasets, so accuracy may vary with chemistry and depth
- The 200 nt minimum contig filter and scoring thresholds (-s, -q, -i) trade off specificity vs. sensitivity; larger k values in TASR increase specificity at possible expense of sensitivity

## Evidence

- [methods] PacBio HiFi validation benchmark and expected alleles: [evidence span withheld — anchor status: not_found]
- [methods] P-designation collapsing rule: "Keeps top two [highest-scoring by HLA group] predictions per gene and only the 'P' designated allele when the summary include HLA Sequences reported to have the same antigen binding domain."
- [intro] Clinically confirmed benchmark individuals: "we utilized one PacBio HiFi and four ONT PromethION whole‐genome shotgun datasets derived from three individuals with known, clinically confirmed HLA types"
- [abstract] Robustness finding at low coverage and old chemistry: "demonstrate the robustness of predictions even with older, less accurate WGS nanopore datasets and relatively low (10×) sequencing coverage"
- [methods] HPRA is the simpler, faster mode used for streaming validation: "The HLA prediction from direct read alignment (HPRA) method is conceptually simpler and faster to execute, since paired reads are aligned up-front to reference HLA alleles."
