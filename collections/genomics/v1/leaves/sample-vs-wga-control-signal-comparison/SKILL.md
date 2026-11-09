---
name: sample-vs-wga-control-signal-comparison
description: Use when you have matched ONT fast5 datasets from a native (methylated)
  sample and a WGA (unmethylated) control, resquiggled against a reference genome,
  and want to de novo identify modified-base motifs from raw signal differences.
when_to_use_negative:
- You have only a native sample with no WGA control — Snapper's sample comparison
  requires -control_fast5dir and cannot be applied
- Reads are not ONT raw-signal fast5 files (e.g. basecalled FASTQ or PacBio data)
  — the signal-level k-mer comparison depends on resquiggled nanopore signal
- The goal is per-read per-base modification calling rather than de novo motif discovery
edam_operation: http://edamontology.org/operation_0415
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0160
tools:
- name: Snapper
  role: Performs the sample-vs-WGA-control signal-level k-mer comparison and greedy
    motif enrichment to de novo detect methylated motifs; installed as pip package
    snapper-ont
  repo: DNKonanov/Snapper
- name: Tombo
  role: Resquiggles raw nanopore signal to the reference sequence, aligning signal
    needed for the sample-vs-control comparison
  repo: https://github.com/nanoporetech/tombo
- name: ont_fast5_api
  role: Converts resquiggled single-read fast5 files to the multi-read fast5 format
    required by snapper
  repo: https://github.com/nanoporetech/ont_fast5_api
- name: Nanodisco
  role: Alternative methylation-analysis tool with which the sample/control signal
    comparison approach is contrasted; its automated mode fails on J99 by merging
    CATG and GCGC into GCRYG
provenance:
  source_task_ids:
  - task_005
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad702
    title: 'Snapper: high-sensitive detection of methylation motifs based on Oxford
      Nanopore reads'
  source_content_hash: sha256:117e9cf7f55230f8cfd33c01c7a10eb11572dfa9220c1079188ac6f24867347f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DNKonanov/Snapper
  tools:
  - Nanodisco
  - Python
  - Snapper
  - Tombo
  - ont_fast5_api
  - snapper-ont
  edam_operation: http://edamontology.org/operation_0415
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# sample-vs-wga-control-signal-comparison

## Summary

Detects methylated motifs by comparing raw Oxford Nanopore signal-level k-mer statistics between native (methylated) sample reads and whole-genome-amplified (WGA) control reads, where methylated positions show signal shifts absent in the WGA control. This sample-versus-control comparison underlies both Tombo's sample-comparison mode and Snapper's greedy motif enrichment.

## When to use

Use when you have matched ONT fast5 datasets from a native (methylated) sample and a WGA (unmethylated) control, resquiggled against a reference genome, and want to de novo identify modified-base motifs from raw signal differences.

## When NOT to use

- You have only a native sample with no WGA control — Snapper's sample comparison requires -control_fast5dir and cannot be applied
- Reads are not ONT raw-signal fast5 files (e.g. basecalled FASTQ or PacBio data) — the signal-level k-mer comparison depends on resquiggled nanopore signal
- The goal is per-read per-base modification calling rather than de novo motif discovery

## Inputs

- Native (sample) ONT fast5 reads, resquiggled and converted to multi-read fast5 format
- WGA (amplified) control ONT fast5 reads, resquiggled and converted to multi-read fast5 format
- Reference genome in FASTA format (e.g. J99.fasta)

## Outputs

- passed_motifs_[strand]_[contig_name].fasta files listing all k-mers most likely carrying a modified base, with per-k-mer confidence values
- final_motifs FASTA files for each contig
- Detected methylation motif set (e.g. the H.pylori J99 motif set recovered with 100x minimum coverage)

## How to apply

Prepare paired ONT fast5 datasets — the native sample and the WGA control for the same strain (e.g. H.pylori J99) — and a reference genome in FASTA format (e.g. J99.fasta). Resquiggle both fast5 sets against the reference using Tombo so raw signal is aligned to basecalls, then convert to multi-fast5 format with ont_fast5_api. Run snapper with -sample_fast5dir pointing to the sample multi-fast5 directory and -control_fast5dir to the WGA multi-fast5 directory; Snapper computes balanced statistics per k-mer (default k_size=15, long_k_size=29, ks_t=3 as -log KS-test p-value, min_conf=100, default coverage 40 — raised to 100x for the J99 analysis) and applies a greedy position-specific motif enrichment algorithm. Rationale: the WGA control lacks methylation, so k-mers whose sample signal distribution deviates from the WGA distribution (KS-test) are candidates for modified bases. Note that fully automated interpretation of motif collisions (e.g. merging CATG and GCGC into GCRYG in Nanodisco's automated mode) is a known failure mode; Snapper is designed to avoid manual control during enrichment.

## Related tools

- **Snapper** (Performs the sample-vs-WGA-control signal-level k-mer comparison and greedy motif enrichment to de novo detect methylated motifs; installed as pip package snapper-ont) — DNKonanov/Snapper
- **Tombo** (Resquiggles raw nanopore signal to the reference sequence, aligning signal needed for the sample-vs-control comparison) — https://github.com/nanoporetech/tombo
- **ont_fast5_api** (Converts resquiggled single-read fast5 files to the multi-read fast5 format required by snapper) — https://github.com/nanoporetech/ont_fast5_api
- **Nanodisco** (Alternative methylation-analysis tool with which the sample/control signal comparison approach is contrasted; its automated mode fails on J99 by merging CATG and GCGC into GCRYG)

## Examples

```
snapper -sample_fast5dir ../HelicobacterMod/fast5/J99_multi/ -control_fast5dir ../HelicobacterMod/fast5/J99_wga_multi/ -reference ../HelicobacterMod/genome/J99.fasta -coverage 100
```

## Evaluation signals

- passed_motifs_[strand]_[contig_name].fasta and final_motifs FASTA files exist for every contig in the reference
- With coverage threshold 100x and defaults otherwise, the recovered motif set on H.pylori J99 matches the earlier PacBio-based characterization (except TCNNGA, judged not an individual motif)
- Per-k-mer confidence values are present and confidence scores meet min_conf=100 for emitted motifs
- Motifs are not spuriously collided/merged — e.g. unlike Nanodisco automated mode on J99, CATG and GCGC are reported separately rather than collapsed into GCRYG
- KS-test statistic threshold respected: ks_t=3 corresponds to a -log10 p-value of 3 for the sample-vs-control signal comparison

## Limitations

- Snapper requires python 3.7 (later versions may be incompatible due to inner biopython dependencies)
- Tombo is deprecated and unsupported by Oxford Nanopore (replaced by Remora)
- No existing methylation motif detection algorithms are sensitive enough and return deterministic results; MEME does not extract all H.pylori methylation sites de novo even with Nanodisco's iterative approach
- Default coverage threshold is 40x and was raised to 100x for the J99 analysis; lower coverage may affect motif detection

## Evidence

- [methods] balanced k-mer statistics: "Firstly, fast5 files should be resquiggled using [Tombo](https://github.com/nanoporetech/tombo) software."
- [methods] multi-fast5 conversion: "After resquiggling, fast5 files should be converted to the multi-fast5 format using [ont_fast5_api](https://github.com/nanoporetech/ont_fast5_api)."
- [methods] snapper invocation: "snapper -sample_fast5dir ../HelicobacterMod/fast5/J99_multi/ -control_fast5dir ../HelicobacterMod/fast5/J99_wga_multi/ -reference ../HelicobacterMod/genome/J99.fasta"
- [intro] greedy enrichment: "The core feature of Snapper in comparison with other tools is a new high-sensitive greedy algorithm that is used 
for position-specific motif enrichment."
- [abstract] no manual control: "Snapper does not require manual control during the enrichment process and has enrichment sensitivity higher than MEME coupled with Tombo or Nanodisco instruments"
- [intro] k-mer stats: "Snapper uses balanced approach to compute statistics for each k-mer which is likely to be modified."
- [readme] tombo deprecation: [evidence span withheld — anchor status: external_doc]
- [readme] python 3.7 requirement: "python 3.7 (later versions might be incompatible because of inner biopython dependencies)"
- [readme] default coverage: "-coverage COVERAGE    minimal genome coverage depth (default 40)"
