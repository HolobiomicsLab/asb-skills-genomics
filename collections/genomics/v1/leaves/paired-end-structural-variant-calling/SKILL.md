---
name: paired-end-structural-variant-calling
description: Use when analyzing sorted/indexed Illumina paired-end BAM/CRAM alignments
  (e.g. bwa mem-aligned HG002 WGS at ~20-40x) to detect SVs/indels ≥30 bp, or when
  benchmarking PE SV callers against a GIAB truth set with svbench.
when_to_use_negative:
- Input is long-read data (PacBio/ONT) — use dysgu's 'call' with --mode presets (e.g.
  nanopore-r10, pacbio-revio) instead of the PE run pipeline
- Sample is somatic/tumor with copy-number changes or poly-clonality where events
  have low allelic fraction — the default diploid model underperforms; use --diploid
  False
- Goal is resolving full-length insertion sequences or multi-allelic sites from PE
  data — the paper notes dysgu lacks a de novo assembly stage and these remain unresolved
edam_operation: http://edamontology.org/operation_3228
edam_topics:
- http://edamontology.org/topic_3175
- http://edamontology.org/topic_0622
tools:
- name: dysgu
  role: Primary SV/indel caller run in paired-end mode; fetch+call pipeline with diploid
    ML quality model, consensus contigs, and remapping of anomalous sequences
  repo: https://github.com/kcleal/dysgu
- name: manta
  role: PE baseline SV caller for comparison on the same alignments
- name: delly
  role: PE baseline SV caller for comparison
- name: lumpy
  role: PE baseline SV caller for comparison
- name: svbench
  role: Evaluates dysgu and baseline callsets against the GIAB truth VCF (PASS-only,
    size/interval-overlap criteria) to compute precision, recall, F1
  repo: https://github.com/kcleal/svbench
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkac039
    title: 'Dysgu: efficient structural variant calling using short or long reads'
  source_content_hash: sha256:6cf09e8fca2dfc2363e3b52a88d02f73795c661a46840c5601d9af4d34e639a7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kcleal/dysgu
  tools:
  - delly
  - dysgu
  - gatk
  - lumpy
  - manta
  - strelka
  - svbench
  edam_operation: http://edamontology.org/operation_3228
  edam_topics:
  - http://edamontology.org/topic_3175
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# paired-end-structural-variant-calling

## Summary

Call structural variants (deletions, insertions) from Illumina paired-end sequencing data using dysgu in paired-end mode, benchmarked against PE callers such as manta, delly, and lumpy on GIAB HG002 Tier 1 SVs.

## When to use

Use when analyzing sorted/indexed Illumina paired-end BAM/CRAM alignments (e.g. bwa mem-aligned HG002 WGS at ~20-40x) to detect SVs/indels ≥30 bp, or when benchmarking PE SV callers against a GIAB truth set with svbench.

## When NOT to use

- Input is long-read data (PacBio/ONT) — use dysgu's 'call' with --mode presets (e.g. nanopore-r10, pacbio-revio) instead of the PE run pipeline
- Sample is somatic/tumor with copy-number changes or poly-clonality where events have low allelic fraction — the default diploid model underperforms; use --diploid False
- Goal is resolving full-length insertion sequences or multi-allelic sites from PE data — the paper notes dysgu lacks a de novo assembly stage and these remain unresolved

## Inputs

- Sorted, indexed .bam/.cram of Illumina paired-end reads (e.g. bwa mem-aligned GIAB HG002 NIST_HiSeq_HG002_Homogeneity-10953946)
- Indexed reference genome FASTA (e.g. GRCh37/GRCh38)
- GIAB truth-set VCF (NIST_SVs_Integration_v0.6) and Tier 1 high-confidence region BED

## Outputs

- SV/indel callset VCF from dysgu PE mode (deletions, insertions, duplications, inversions with model probability scores)
- Per-caller precision/recall/F1 benchmark table (e.g. benchmark_table.tsv) stratified by Tier 1 vs all regions

## How to apply

Run dysgu in paired-end mode ('dysgu run reference.fa temp_dir input.bam > svs.vcf'), which wraps the 'fetch' step (separating split/discordant reads and soft-clipped reads ≥15 bp into a dysgu_reads.bam) and the 'call' step; use '--ibam all_reads.bam' with call so insert-size metrics are inferred from the full alignment file. Set '-p' for multiprocessing, and note that fetch generates a >5 GB temp file of SV-associated reads and a 30x genome takes ~1 h per core with <6 GB RAM. Use the default diploid model (which models read-depth around break sites) for germline WGS, switching to '--diploid False' for low-allelic-fraction somatic samples. Benchmark outputs with svbench against the GIAB NIST_SVs_Integration_v0.6 truth set with --pass-only, min event sizes of 30/50 bp, 1000 bp interval overlap and 15% size similarity, stratified to Tier 1 and all regions.

## Related tools

- **dysgu** (Primary SV/indel caller run in paired-end mode; fetch+call pipeline with diploid ML quality model, consensus contigs, and remapping of anomalous sequences) — https://github.com/kcleal/dysgu
- **manta** (PE baseline SV caller for comparison on the same alignments)
- **delly** (PE baseline SV caller for comparison)
- **lumpy** (PE baseline SV caller for comparison)
- **svbench** (Evaluates dysgu and baseline callsets against the GIAB truth VCF (PASS-only, size/interval-overlap criteria) to compute precision, recall, F1) — https://github.com/kcleal/svbench

## Examples

```
dysgu run -p4 reference.fa temp_dir input.bam > svs.vcf
```

## Evaluation signals

- Dysgu should attain the highest Tier 1 F1 among PE callers: 4.7% higher F1 than manta for deletions and 13.1% higher for insertions at 20x on HG002
- Reported reference metrics at 20x HG002 Tier 1: dysgu deletion precision 95.7%, recall 62.2%; insertion recall 23.7%
- Dysgu called the largest number of true deletions+insertions (n = 3913), 727 more than manta (n = 3186)
- svbench must be run with PASS-only filtering, min-size 30/50 bp, 1000 bp interval overlap and 15% size similarity for the benchmark to be comparable
- VCF should be non-empty and sorted; fetch should produce a dysgu_reads.bam of SV-associated reads (>5 GB temp file expected for a 30x genome)

## Limitations

- Version of dysgu used in the paper is not stated; software versions for manta/delly/lumpy/strelka/gatk also unstated
- No de novo assembly stage: multi-allelic sites and full-length insertion sequences in PE data remain unresolved, and insertion sizes are expected to be less accurate than assembly-based callers such as manta and gatk (data not shown)
- Heuristics (span position distance, minimizer support) are unoptimized and could be tuned with a larger test set; the ML classifier was trained on manually labelled (potentially biased) data
- Using the default --divergence 0.02 on lower-accuracy reads filters too many reads and reduces sensitivity; use '--divergence auto' for noisy data

## Evidence

- [results] On HG002 Tier 1 SVs at 20× coverage, dysgu showed higher F1 scores than the next best caller manta, with an F1 score 4.7% higher for deletions and 13.1% higher for insertions.: [evidence span withheld — anchor status: not_found]
- [results] Dysgu had the highest precision for deletion calls (95.7%), as well as the highest recall for deletions (62.2%) and insertions (23.7%): "Dysgu had the highest precision for deletion calls (95.7%), as well as the highest recall for deletions (62.2%) and insertions (23.7%)"
- [readme] dysgu run reference.fa temp_dir input.bam > svs.vcf: "For paired-end data the `run` command is recommended which wraps `fetch` and `call`::

    dysgu run reference.fa temp_dir input.bam > svs.vcf"
- [readme] The first stage of the "run" pipeline is to separate SV-associated reads - split/discordant reads, and reads with a soft-clip >= clip_length (15 bp by default for paired-end reads).: "The first stage of the "run" pipeline is to separate SV-associated reads - split/discordant reads,
and reads with a soft-clip >= clip_length (15 bp by default for paired-end reads)."
- [intro] dysgu identifies SVs from alignment cigar information as well as discordant and split-read mappings: "Conceptually, dysgu identifies SVs from alignment cigar information as well as discordant and split-read mappings."
