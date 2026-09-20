---
name: chh-motif-kmer-enumeration
description: Use when assembling or auditing a nanopore 5mC training corpus for CHH
  methylation detection and you need to measure motif diversity — e.g., computing
  the fraction of 36,864 possible 9-mer CHH contexts with sufficient positive/negative
  support across species datasets.
when_to_use_negative:
- The methylation context of interest is CpG or CHG, not CHH — the 36,864 enumeration
  space and >=50/50 support threshold were designed for CHH scarcity
- You only need per-site methylation calls from nanopore reads (use DeepPlant or Dorado
  inference directly, not motif-coverage auditing)
- Sites lack strand/context annotation or the flanking sequence around each cytosine
  cannot be extracted from the reference, making 9-mer context reconstruction impossible
edam_operation: http://edamontology.org/operation_0232
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_0622
tools:
- name: DeepPlant
  role: The CHH methylation detection model whose training corpus this coverage audit
    validates; its extract_hc_sites utility produces the high-confidence positive/negative
    site lists that feed the 9-mer tallying
  repo: https://github.com/xiaochuanle/DeepPlant
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-58576-x
    title: Accurate cross-species 5mC detection for Oxford Nanopore sequencing in
      plants with DeepPlant
  source_content_hash: sha256:aefb40c7534c3f7848e20ef54002dc4712b4875c2090fdabbf1a270b4fb88707
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xiaochuanle/DeepPlant
  tools:
  - DeepPlant
  edam_operation: http://edamontology.org/operation_0232
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# chh-motif-kmer-enumeration

## Summary

Enumerates all possible 9-mer sequence contexts centered on a CHH motif (9 central trinucleotides x 4^6 flanking bases = 36,864) and quantifies how many are covered by high-confidence CHH methylation training samples, replicating the DeepPlant training-set coverage statistic of 97.2% at >=50 positive and >=50 negative samples per context.

## When to use

Use when assembling or auditing a nanopore 5mC training corpus for CHH methylation detection and you need to measure motif diversity — e.g., computing the fraction of 36,864 possible 9-mer CHH contexts with sufficient positive/negative support across species datasets.

## When NOT to use

- The methylation context of interest is CpG or CHG, not CHH — the 36,864 enumeration space and >=50/50 support threshold were designed for CHH scarcity
- You only need per-site methylation calls from nanopore reads (use DeepPlant or Dorado inference directly, not motif-coverage auditing)
- Sites lack strand/context annotation or the flanking sequence around each cytosine cannot be extracted from the reference, making 9-mer context reconstruction impossible

## Inputs

- High-confidence CHH positive site lists (from bisulfite-validated nanopore calls) per species
- High-confidence CHH negative site lists per species
- Reference genome sequences for extracting the 9-mer context around each cytosine

## Outputs

- chh_9mer_coverage_report.csv with per-9-mer positive/negative support counts
- Coverage fraction over the 36,864 possible 9-mer CHH contexts (e.g., 0.972)
- Count of covered 9-mers and average samples per context (e.g., >9225)

## How to apply

Load the high-confidence CHH positive and negative site lists from each species' nanopore dataset (e.g., S. miltiorrhiza, S. tuberosum, R. communis) with pandas/numpy, extracting the 9-base window centered on each cytosine. Enumerate the 36,864 candidate contexts programmatically: 9 CHH central trinucleotides (C with H in {A,C,T} at each of two positions) x 4^6 flanking combinations via itertools.product over A/C/G/T. Tally per-9-mer counts in the positive set and separately in the negative set, then flag a 9-mer as covered only when it reaches the minimum-support threshold of >=50 occurrences in BOTH sets. Divide the covered count by 36,864 to get the coverage fraction; expect ~97.2% for the combined three-species dataset with an average of over 9225 samples per context. Write the covered count, coverage fraction, and per-9-mer support counts to a summary CSV (e.g., chh_9mer_coverage_report.csv). The rationale is that scarce methylation-positive CHH samples limit model generalization, so coverage over diverse 9-mer motifs is the key training-corpus quality metric.

## Related tools

- **DeepPlant** (The CHH methylation detection model whose training corpus this coverage audit validates; its extract_hc_sites utility produces the high-confidence positive/negative site lists that feed the 9-mer tallying) — https://github.com/xiaochuanle/DeepPlant

## Examples

```
from itertools import product; ctxs = [''.join(f)+c+h1+h2 for f in product('ACGT', repeat=6) for c in 'C' for h1 in 'ACT' for h2 in 'ACT']; len(ctxs)  # 36864
```

## Evaluation signals

- The enumeration must produce exactly 36,864 unique 9-mer contexts (9 CHH central trinucleotides x 4^6 flanks); any other count indicates a bug
- Combined three-species coverage fraction should reproduce ~97.2% under the >=50 positive / >=50 negative support rule
- Average samples per covered context should exceed 9225 for the combined dataset
- Per-9-mer counts in the report CSV must sum consistently with the number of input CHH sites per species
- A 9-mer counted as covered must have >=50 occurrences in BOTH the positive and negative sets, never just one

## Limitations

- Coverage is defined only for the combined multi-species corpus; individual species datasets cover far fewer of the 36,864 contexts
- Even with screening of species with abundant high-methylation CHH sites, scarcity of CHH-positive samples remains a challenge for model training and generalization
- Reliance on species with naturally abundant CHH methylation may not capture the full diversity of methylation patterns across all plant species; in vitro enzyme-catalyzed methylation is ineffective because CHH methylation depends on RdDM (DRM1/DRM2)
- A 97.2% coverage figure says nothing about depth balance — some covered contexts may sit near the >=50 threshold and still be under-sampled

## Evidence

- [discussion] training dataset covers 97.2% of all possible 9-mer CHH contexts: "Our comprehensive training dataset now covers 97.2% of all possible 9-mer CHH contexts, averaging over 9225 samples per context"
- [abstract] motivation: CHH positive sample scarcity: "We address the scarcity of methylation-positive CHH training samples through screening species with abundant high-methylation CHH sites using bisulfite-sequencing"
- [intro] new Nanopore/BS-seq data for CHH-positive samples: "We generate new Nanopore and BS-seq data from selected species, significantly increasing the number of CHH-positive samples."
- [methods] extract_hc_sites CLI producing pos/neg k-mer lists: "Usage: extract_hc_sites [--help] [--version] pod5_dir bam_path reference_path ref_type write_dir pos neg kmer_size num_workers"
- [readme] README: diverse 9-mer motif coverage rationale: "we reanalyzed bisulfite sequencing (BS-seq) data to screen species with abundant high-methylation CHH sites, generating new datasets that cover diverse 9-mer motifs"
