---
name: nanopore-5mc-modification-calling
description: Use when profiling plant 5mC from ONT R10.4 POD5/BAM data, especially
  CHH methylation in repetitive regions (centromeres, transposable elements) where
  short-read BS-seq coverage is poor.
when_to_use_negative:
- Data were generated on the R9.4 FlowCell — Tombo, Megalodon, and DeepSignal-Plant
  models built for R9.4 cannot be directly applied to R10.4, and DeepPlant is trained/tested
  on R10.4 data
- The goal is CpG or CHG detection only and a faster runtime is required — DeepPlant
  is less computationally efficient than Dorado on the same dataset
- Input is bulk whole-genome methylation frequency where BS-seq already achieves near-complete
  coverage in non-centromeric regions (~98%)
edam_operation: http://edamontology.org/operation_0004
edam_topics:
- http://edamontology.org/topic_0157
- http://edamontology.org/topic_0622
tools:
- name: DeepPlant
  role: Primary 5mC detection model (Bi-LSTM + Transformer) run via 'extract_and_call_mods'
    on POD5/BAM/reference to emit per-cytosine methylation_rate calls
  repo: https://github.com/xiaochuanle/DeepPlant
- name: Dorado
  role: Benchmark baseline for plant 5mC detection on the R10.4 platform against which
    DeepPlant CHH correlations are compared
  repo: https://github.com/nanoporetech/dorado
- name: Oxford Nanopore Technologies (ONT) R10.4 FlowCell
  role: Sequencing platform producing the raw signal (POD5) data that DeepPlant is
    designed for
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-58576-x
    title: Accurate cross-species 5mC detection for Oxford Nanopore sequencing in
      plants with DeepPlant
  source_content_hash: sha256:c0de9126fc47234c5887f536fabfb3bb0521a6ec4623cd072ae8f0fda1fc5445
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xiaochuanle/DeepPlant
  tools:
  - DeepPlant
  - Dorado
  - Oxford Nanopore Technologies (ONT) R10.4 FlowCell
  edam_operation: http://edamontology.org/operation_0004
  edam_topics:
  - http://edamontology.org/topic_0157
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# nanopore-5mc-modification-calling

## Summary

Detect 5-methylcytosine (5mC) at single-molecule resolution in plant CpG, CHG, and CHH contexts from Oxford Nanopore R10.4 sequencing data using DeepPlant, a Bi-LSTM/Transformer deep learning model that outperforms Dorado on cross-species CHH methylation quantification.

## When to use

Use when profiling plant 5mC from ONT R10.4 POD5/BAM data, especially CHH methylation in repetitive regions (centromeres, transposable elements) where short-read BS-seq coverage is poor.

## When NOT to use

- Data were generated on the R9.4 FlowCell — Tombo, Megalodon, and DeepSignal-Plant models built for R9.4 cannot be directly applied to R10.4, and DeepPlant is trained/tested on R10.4 data
- The goal is CpG or CHG detection only and a faster runtime is required — DeepPlant is less computationally efficient than Dorado on the same dataset
- Input is bulk whole-genome methylation frequency where BS-seq already achieves near-complete coverage in non-centromeric regions (~98%)

## Inputs

- ONT R10.4 POD5 signal directory (e.g., ~43x O. sativa nanopore reads)
- Sorted BAM alignment file against a reference genome (e.g., T2T-NIP40)
- Reference genome FASTA (e.g., T2T-NIP, ref_type DNA)
- Pre-trained DeepPlant Bi-LSTM model directory (/DeepPlant/model/bilstm)
- Optional: centromeric/TE annotation intervals (BED/GFF) for region-restricted methylation summarization

## Outputs

- Per-cytosine methylation TSV with read_id, chromosome, pos_in_reference, strand, and methylation_rate
- Region-level methylation frequency summaries (e.g., CHH coverage fractions per centromeric/non-centromeric region; strand-specific frequencies per TE type)

## How to apply

Pull the chenhx26/deepplant:1.1.0 Docker image, verify GPU access with nvidia-smi inside the container, and mount the data directory read-write. Run 'DeepPlant extract_and_call_mods' on the POD5 directory, sorted BAM, and reference FASTA (ref_type DNA) using the pre-trained Bi-LSTM model at /DeepPlant/model/bilstm with chh_kmer_size 13, producing a per-cytosine TSV with read_id, chromosome, pos_in_reference, strand, and methylation_rate. Restrict calls to contexts of interest (e.g., CHH sites within centromeric coordinates of T2T-NIP chromosomes 1-12) and apply a minimum read coverage threshold of 10 per site before computing methylation frequencies. Aggregate per-site methylation_rate values into region-level frequencies (e.g., strand-specific averages per centromeric TE class such as Ac/Ds, Mariner, LINE, Ty1-copia) and validate against deposited BS-seq or Fig. 3b/3e source data values.

## Related tools

- **DeepPlant** (Primary 5mC detection model (Bi-LSTM + Transformer) run via 'extract_and_call_mods' on POD5/BAM/reference to emit per-cytosine methylation_rate calls) — https://github.com/xiaochuanle/DeepPlant
- **Dorado** (Benchmark baseline for plant 5mC detection on the R10.4 platform against which DeepPlant CHH correlations are compared) — https://github.com/nanoporetech/dorado
- **Oxford Nanopore Technologies (ONT) R10.4 FlowCell** (Sequencing platform producing the raw signal (POD5) data that DeepPlant is designed for)

## Examples

```
docker run -i -t -v data_path:data_path:rw --gpus all chenhx26/deepplant:1.1.0 /bin/bash -c 'DeepPlant extract_and_call_mods $DATA/pod5 $DATA/sorted.bam $DATA/ref.fa DNA $DATA/out /DeepPlant/model/bilstm'
```

## Evaluation signals

- Whole-genome CHH methylation frequency correlations with BS-seq fall in the reported 0.705–0.838 range across nine species (correlation improvement of 0.135–0.381 over Dorado)
- On O. sativa T2T-NIP centromeres with ≥10x coverage, DeepPlant CHH coverage ratio reproduces the deposited Fig. 3b value of 88.0% vs BS-seq 37.7%, with non-centromeric coverage ~98%
- Strand-bias pattern of centromeric TE methylation matches Fig. 3e: Ac/Ds and Mariner higher on the forward strand; LINE and Ty1-copia higher on the reverse strand
- Output TSV schema contains read_id, chromosome, pos_in_reference, strand, and methylation_rate for each called cytosine
- GPU availability confirmed inside the container via nvidia-smi before running inference

## Limitations

- DeepPlant is less computationally efficient than Dorado — it took much longer to call methylation on the same dataset (Supplementary Data 10)
- Scarcity of high-methylation CHH training samples continues to pose challenges for model training and generalization; reliance on species with naturally abundant CHH methylation may not capture the full diversity of methylation patterns
- Potential for overfitting to specific sequence patterns underscores the need for careful model optimization and validation
- In vitro enzyme-catalyzed methylation is ineffective for generating CHH positive samples because CHH methylation depends on RdDM (DRM1/DRM2), and PCR amplification with modified base substitutions struggles to produce high-purity 5mC

## Evidence

- [abstract] DeepPlant achieves whole-genome CHH correlation improvement: "Evaluated across nine species, DeepPlant achieves high whole-genome methylation frequency correlations (0.705-0.838) with BS-seq data on CHH, improved by 23.4- 117.6% compared to Dorado"
- [methods] DeepPlant command-line interface usage: "Usage: extract_and_call_mods [--help] [--version] pod5_dir bam_path reference_path ref_type write_dir model_dir"
- [methods] Docker container launch command: "docker run -i -t -v data_path:data_path:rw --gpus all chenhx26/deepplant:1.1.0 /bin/bash"
- [intro] R10.4 incompatibility of prior tools: "methylation detection tools developed for the R9.4 platform, such as Tombo17, Megalodon18, and DeepSignal-Plant16, cannot be directly applied to the R10.4 FlowCell"
- [discussion] Training dataset motif coverage: "Our comprehensive training dataset now covers 97.2% of all possible 9-mer CHH contexts, averaging over 9225 samples per context"
- [readme] DeepPlant purpose statement from README: "Nanopore sequencing enables comprehensive detection of 5-methylcytosine (5mC), particularly in transposable elements and centromeric regions. However, CHH methylation detection in plants is limited"
