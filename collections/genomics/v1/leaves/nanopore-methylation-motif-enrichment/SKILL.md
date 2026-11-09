---
name: nanopore-methylation-motif-enrichment
description: Use when you need to de novo discover DNA methylation motifs (6mA, 5mC,
  4mC) from native vs WGA nanopore signal data of a bacterium, and benchmark recovery
  against REBASE reference motifs.
when_to_use_negative:
- When methylation motifs are already known and the goal is typing/fine-mapping of
  candidate motifs with the pretrained Nanoident model (best_model/Nanoident.pt) rather
  than de novo discovery
- When input is modBAM/bedMethyl data intended for ONT modkit pileup-style workflows
  instead of raw nanopore signal
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3168
tools:
- name: Nanoident
  role: Provides the optimized de novo motif enrichment module and preprocessing scripts
    (Nanoident_preprocess.sh, Nanoident_enrich.sh)
  repo: https://github.com/cz-csu/Nanoident
- name: nanodisco
  role: Baseline automatic de novo motif discovery mode; its R environment (built
    via Singularity v3.2.1+) underlies preprocessing
  repo: https://github.com/fanglab/nanodisco
- name: Singularity
  role: Virtualization software used to build the nanodisco environment (v3.2.1 and
    above)
- name: R
  role: Environment for data preprocessing and motif enrichment
- name: bam-readcount
  role: Installed alongside nanodisco to support preprocessing/basecall feature extraction
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btaf397
    title: Identifying DNA methylation types and methylated base positions from bacteria
      using nanopore sequencing with multi-scale neural network
  source_content_hash: sha256:072a7ec989cd366e6060facc8dced30af724c8498539ea9901ac9fa85906a0cb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cz-csu/Nanoident
  tools:
  - Nanoident
  - R
  - Singularity
  - bam-readcount
  - nanodisco
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# nanopore-methylation-motif-enrichment

## Summary

A de novo methylation motif enrichment workflow for bacterial nanopore data that extends nanodisco preprocessing with four additional signal features (std, time, reference-base proportion, quality score) and compares enriched motifs against REBASE references. It enables recovery of more reference motifs than nanodisco's automatic mode.

## When to use

Use when you need to de novo discover DNA methylation motifs (6mA, 5mC, 4mC) from native vs WGA nanopore signal data of a bacterium, and benchmark recovery against REBASE reference motifs.

## When NOT to use

- When methylation motifs are already known and the goal is typing/fine-mapping of candidate motifs with the pretrained Nanoident model (best_model/Nanoident.pt) rather than de novo discovery
- When input is modBAM/bedMethyl data intended for ONT modkit pileup-style workflows instead of raw nanopore signal

## Inputs

- WGA nanopore raw signal dataset per strain
- Native nanopore raw signal dataset per strain
- Strain reference FASTA (e.g., NCBI accessions CP041693, CP041696, NC_008261.1)
- REBASE reference methylation motifs per strain
- nanodisco Singularity container (v3.2.1+) patched by update.sh

## Outputs

- Difference RDS files per strain (WGA vs NAT subtracted features)
- De novo enriched methylation motifs per strain
- Per-strain motif counts vs nanodisco baseline (e.g., 16 vs 12) written to motif_enrichment_comparison.csv

## How to apply

First run update.sh on the nanodisco Singularity container (v3.2.1+) so preprocessing handles four additional features (std, time, proportion of reference bases, average quality score) plus a base feature for positional correction. Then run Nanoident_preprocess.sh on the WGA and native datasets (passing dataset paths, sample names, reference FASTA, preprocess/diff/merge output dirs) to produce difference RDS files. Run Nanoident_enrich.sh per strain, passing the bacterial name, difference RDS file, motif-enrichment output path, and the strain's reference FASTA. As a baseline, run nanodisco's automatic motif discovery mode on the same preprocessed data. Finally, count motifs per strain, match them against REBASE reference motifs, and record agreement (e.g., per-strain counts of 16 vs 12 for H. pylori JP26, 8 vs 1 for N. gonorrhoeae FA 1090, 8 vs 8 for T. phaeum DSM 12270).

## Related tools

- **Nanoident** (Provides the optimized de novo motif enrichment module and preprocessing scripts (Nanoident_preprocess.sh, Nanoident_enrich.sh)) — https://github.com/cz-csu/Nanoident
- **nanodisco** (Baseline automatic de novo motif discovery mode; its R environment (built via Singularity v3.2.1+) underlies preprocessing) — https://github.com/fanglab/nanodisco
- **Singularity** (Virtualization software used to build the nanodisco environment (v3.2.1 and above))
- **R** (Environment for data preprocessing and motif enrichment)
- **bam-readcount** (Installed alongside nanodisco to support preprocessing/basecall feature extraction)

## Examples

```
bash ./process/code/update.sh -p /home/chenzh/Project1_1/nd_example2
```

## Evaluation signals

- Per-strain motif counts match reported Nanoident vs nanodisco pairs: 16 vs 12 (H. pylori JP26), 8 vs 1 (N. gonorrhoeae FA 1090), 8 vs 8 (T. phaeum DSM 12270)
- Discovered motifs agree with REBASE reference motifs when matched per strain
- motif_enrichment_comparison.csv contains the per-strain counts for all three strains
- Difference RDS files are produced for each strain from the WGA/native subtraction step
- update.sh has patched the nanodisco container so four additional features (std, time, reference-base proportion, quality score) are computed

## Limitations

- Preprocessing must be done in chunks and then merged because of memory constraints
- The original nanodisco handles only mean features; without update.sh the four additional features are unavailable
- Comparison against nanodisco's automatic mode depends on the same preprocessed data being used as baseline

## Evidence

- [abstract] Additionally, we optimize the pipeline for de novo methylation motif enrichment, enabling the discovery of novel methylation motifs.: "Additionally, we optimize the pipeline for de novo methylation motif enrichment, enabling the discovery of novel"
- [methods] Run the update.sh to add four additional features: "Run the update.sh. The original nanodisco could only handle mean features, but Nanoident can handle four additional features"
- [supplementary] Methylation motifs collected from REBASE database: "Methylation motifs of those bacteria were collected from the REBASE database"
- [other] In H. pylori JP26, nanodisco found 12 motifs vs Nanoident 16: [evidence span withheld — anchor status: not_found]
- [readme] Build nanodisco environment with Singularity v3.2.1 and above: "follow the nanodisco (https://github.com/fanglab/nanodisco/tree/master) to build the virtual environment using Singularity (v3.2.1 and above)"
