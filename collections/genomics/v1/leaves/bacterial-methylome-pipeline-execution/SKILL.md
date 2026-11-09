---
name: bacterial-methylome-pipeline-execution
description: Use when raw WGA/native nanopore signal datasets for bacterial strains
  must be turned into de novo methylation motif calls and compared against REBASE
  reference motifs or a baseline method such as nanodisco automatic mode.
when_to_use_negative:
- Input is already a modBAM/bedMethyl from a basecaller's modified-base model (use
  modkit pileup instead of de novo discovery)
- The analysis needs only motif typing/fine mapping of known candidate motifs rather
  than de novo motif discovery
- The genome is not bacterial or no paired WGA/native nanopore data exist, since the
  enrichment module relies on WGA-vs-native signal differences
edam_operation: http://edamontology.org/operation_3898
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3674
tools:
- name: Nanoident
  role: Core pipeline whose optimized enrichment module performs de novo methylation
    motif discovery; its preprocess/enrich shell scripts run the WGA-vs-native comparison
  repo: https://github.com/cz-csu/Nanoident
- name: nanodisco
  role: Baseline automatic motif discovery mode and the container/environment foundation
    for R preprocessing
  repo: https://github.com/fanglab/nanodisco
- name: Singularity
  role: Virtualization software (v3.2.1 and above) used to build the nanodisco container
- name: bam-readcount
  role: Installed to obtain basecall features during preprocessing
- name: R
  role: Environment for data preprocessing and motif enrichment steps
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btaf397
    title: Identifying DNA methylation types and methylated base positions from bacteria
      using nanopore sequencing with multi-scale neural network
  source_content_hash: sha256:8b224e000e2f7dc2fdddfdbac8f39989f4d0a513249d18fc33967b19830fd928
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
  edam_operation: http://edamontology.org/operation_3898
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3674
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# bacterial-methylome-pipeline-execution

## Summary

Execute the Nanoident/nanodisco bacterial methylome pipeline: preprocess WGA and native nanopore signal into difference RDS files with added features (std, time, reference-base proportion, quality score), run de novo methylation motif enrichment, and benchmark recovered motifs against REBASE references.

## When to use

Use when raw WGA/native nanopore signal datasets for bacterial strains must be turned into de novo methylation motif calls and compared against REBASE reference motifs or a baseline method such as nanodisco automatic mode.

## When NOT to use

- Input is already a modBAM/bedMethyl from a basecaller's modified-base model (use modkit pileup instead of de novo discovery)
- The analysis needs only motif typing/fine mapping of known candidate motifs rather than de novo motif discovery
- The genome is not bacterial or no paired WGA/native nanopore data exist, since the enrichment module relies on WGA-vs-native signal differences

## Inputs

- WGA nanopore signal dataset (raw fast5/pod5 per strain, e.g., from SRA BioProject PRJNA559199)
- Native (non-amplified) nanopore signal dataset for the same strain
- Reference FASTA per strain (NCBI accessions CP041693, CP041696, NC_008261.1, CP014225.1, CP023448.1, NC_007796.1, NC_002946.2, CP041695, CP003732)
- nanodisco Singularity container image
- REBASE reference methylation motifs per strain

## Outputs

- Preprocessed difference RDS files (WGA minus NAT signal features per motif)
- De novo enriched methylation motifs per strain with typing and methylated base positions
- motif_enrichment_comparison.csv with per-strain counts (Nanoident vs nanodisco: 16 vs 12, 8 vs 1, 8 vs 8)
- Motif-to-REBASE agreement records per strain

## How to apply

Build the nanodisco Singularity container (v3.2.1+) for R-based preprocessing/enrichment and a conda environment from Nanoident/env/freeze.yml for model work. Run update.sh (passing the nanodisco container path with -p) so nanodisco gains four additional features beyond mean: std, time, reference-base proportion, and average quality score, plus a base feature for positional correction. For each strain run Nanoident_preprocess.sh with the WGA path, native path, sample names, reference FASTA, preprocess/diff/merge output directories — chunked preprocessing is required because of memory constraints — producing difference RDS files. Then run Nanoident_enrich.sh, passing the bacterial name, the strain's difference RDS file, the motif-enrichment output path, and the reference FASTA. For benchmarking, run nanodisco's automatic motif discovery on the same preprocessed data as the baseline, count discovered motifs per strain, match them against REBASE reference motifs, and record per-strain counts (e.g., H. pylori JP26: Nanoident 16 vs nanodisco 12).

## Related tools

- **Nanoident** (Core pipeline whose optimized enrichment module performs de novo methylation motif discovery; its preprocess/enrich shell scripts run the WGA-vs-native comparison) — https://github.com/cz-csu/Nanoident
- **nanodisco** (Baseline automatic motif discovery mode and the container/environment foundation for R preprocessing) — https://github.com/fanglab/nanodisco
- **Singularity** (Virtualization software (v3.2.1 and above) used to build the nanodisco container)
- **bam-readcount** (Installed to obtain basecall features during preprocessing)
- **R** (Environment for data preprocessing and motif enrichment steps)

## Examples

```
bash ./process/code/update.sh -p /home/chenzh/Project1_1/nd_example2; chmod +x ./process/code/Nanoident_preprocess.sh; ./process/code/Nanoident_enrich.sh <strain> <diff.RDS> <enrich_out/> <ref.fasta>
```

## Evaluation signals

- Motif counts per strain reproduce the reported pairs: Nanoident 16 vs nanodisco 12 for H. pylori JP26, 8 vs 1 for N. gonorrhoeae FA 1090, 8 vs 8 for T. phaeum DSM 12270
- motif_enrichment_comparison.csv contains the three per-strain count pairs with correct strain labels
- Discovered motifs match the strain's REBASE reference motifs (H. pylori JP26 has 20 REBASE reference motifs, with Nanoident recovering more than nanodisco)
- Difference RDS files contain the five Nanoident features (mean plus std, time, reference-base proportion, average quality score) confirming update.sh ran successfully
- Preprocess/merge steps completed chunked output without memory failure for each strain

## Limitations

- Preprocessing must be chunked due to memory constraints ('Because of memory constraints, we need to preprocess in chunks first')
- The original nanodisco could only handle mean features; the update.sh patch is required before Nanoident preprocessing
- Both R (via nanodisco/Singularity) and Python (via conda freeze.yml) environments must exist; Singularity must be v3.2.1 or above

## Evidence

- [readme] Run the update.sh. The original nanodisco could only handle mean features, but Nanoident can handle four additional features (std, time,the proportion of reference bases, average quality score).: "Run the update.sh. The original nanodisco could only handle mean features, but Nanoident can handle four additional features"
- [readme] Because of memory constraints, we need to preprocess in chunks first: "Because of memory constraints, we need to preprocess in chunks first"
- [methods] follow the nanodisco (https://github.com/fanglab/nanodisco/tree/master) to build the virtual environment using Singularity (v3.2.1 and above): "follow the nanodisco (https://github.com/fanglab/nanodisco/tree/master) to build the virtual environment using Singularity (v3.2.1 and above)"
- [methods] an R environment for data preprocessing and motif enrichment: "an R environment for data preprocessing and motif enrichment"
- [other] In H. pylori JP26, nanodisco's automatic mode found 12 motifs while Nanoident's optimized enrichment module identified 16: [evidence span withheld — anchor status: not_found]
