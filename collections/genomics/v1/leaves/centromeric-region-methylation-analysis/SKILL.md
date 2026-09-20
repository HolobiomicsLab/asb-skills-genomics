---
name: centromeric-region-methylation-analysis
description: Use when profiling CHH methylation in plant centromeres or transposable
  elements from ONT R10.4 nanopore data, especially where short-read BS-seq coverage
  is poor and per-site, strand-aware methylation frequencies are needed.
when_to_use_negative:
- Input is short-read BS-seq data rather than ONT R10.4 nanopore reads — DeepPlant
  is a nanopore methylation caller, not a bisulfite analyzer
- The methylation caller must be R9.4-compatible only — DeepPlant targets the R10.4
  FlowCell, and older tools (Tombo, Megalodon, DeepSignal-Plant) do not directly apply
  to R10.4
- Computational runtime is the dominant constraint — DeepPlant is less efficient than
  Dorado (much longer calling time on the same dataset, Supplementary Data 10)
edam_topics:
- http://edamontology.org/topic_3295
tools:
- name: DeepPlant
  role: Performs CHH 5mC methylation calling via 'extract_and_call_mods' with the
    pre-trained Bi-LSTM model; consumed as Docker image chenhx26/deepplant:1.1.0
  repo: github.com/xiaochuanle/DeepPlant
- name: Oxford Nanopore Technologies (ONT) R10.4 FlowCell
  role: Sequencing platform whose long reads enable methylation profiling of centromeric
    and repetitive regions inaccessible to short-read BS-seq
- name: Dorado
  role: Baseline plant 5mC caller on R10.4 used for benchmark comparison of CHH correlation
    and runtime
  repo: nanoporetech/dorado
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-58576-x
    title: Accurate cross-species 5mC detection for Oxford Nanopore sequencing in
      plants with DeepPlant
  source_content_hash: sha256:7de60d9fe79348739fe3fa23105539ea830aad1259343c9454be1331d70407d2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepPlant
  - Dorado
  - Oxford Nanopore Technologies (ONT) R10.4 FlowCell
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# centromeric-region-methylation-analysis

## Summary

Profiles CHH 5-methylcytosine in centromeric and repetitive plant regions using DeepPlant's Bi-LSTM/Transformer models on ONT R10.4 nanopore reads, exploiting long reads to reach regions short-read BS-seq undercovers (88.0% vs 37.7% centromeric CHH site coverage in O. sativa).

## When to use

Use when profiling CHH methylation in plant centromeres or transposable elements from ONT R10.4 nanopore data, especially where short-read BS-seq coverage is poor and per-site, strand-aware methylation frequencies are needed.

## When NOT to use

- Input is short-read BS-seq data rather than ONT R10.4 nanopore reads — DeepPlant is a nanopore methylation caller, not a bisulfite analyzer
- The methylation caller must be R9.4-compatible only — DeepPlant targets the R10.4 FlowCell, and older tools (Tombo, Megalodon, DeepSignal-Plant) do not directly apply to R10.4
- Computational runtime is the dominant constraint — DeepPlant is less efficient than Dorado (much longer calling time on the same dataset, Supplementary Data 10)

## Inputs

- Oxford Nanopore POD5 read directory (R10.4 FlowCell, e.g., ~43x O. sativa)
- Sorted BAM alignment of the nanopore reads
- Reference genome FASTA (e.g., T2T-NIP40 O. sativa assembly)
- Centromeric interval annotation BED for chromosomes 1-12
- Transposable element annotation (seven centromeric TE types: Ac/Ds, Mariner, LINE, Ty1-copia, etc.) and protein-coding region annotation
- Published whole-genome BS-seq dataset for the same species (for comparison)

## Outputs

- Per-cytosine TSV from DeepPlant call_mods with read_id, chromosome, pos_in_reference, strand, and methylation_rate for CHH sites
- Fraction of CHH sites with coverage >= 10 in centromeric vs non-centromeric regions (target: 88.0% vs ~37.7% centromeric; ~98% non-centromeric)
- Strand-specific CHH methylation frequency table per TE class and non-TE protein-coding control regions

## How to apply

Run the chenhx26/deepplant:1.1.0 Docker container with NVIDIA Container Toolkit (--gpus all) and mount the data directory read-write, then invoke 'DeepPlant extract_and_call_mods' on the POD5 directory, sorted BAM, and reference FASTA using the pre-trained Bi-LSTM model at /DeepPlant/model/bilstm with chh_kmer_size 13 and ref_type DNA, producing a per-cytosine TSV of methylation_rate values. Restrict CHH calls to centromeric coordinates (e.g., chromosomes 1-12 of the T2T-NIP O. sativa assembly) using interval overlap (bedtools/pybedtools intersect) with TE and protein-coding annotations, applying a minimum per-site read coverage threshold of 10. Rationale: BS-seq short reads underrepresent centromeres and TEs, whereas DeepPlant's long-read calls reach these regions and improve whole-genome CHH correlation with BS-seq (0.705-0.838 across nine species) over Dorado. Compare site-coverage fractions against the deposited Fig. 3b source data (Zenodo 10.5281/zenodo.15062213), and group by element type and aligned strand to detect strand-specific CHH bias.

## Related tools

- **DeepPlant** (Performs CHH 5mC methylation calling via 'extract_and_call_mods' with the pre-trained Bi-LSTM model; consumed as Docker image chenhx26/deepplant:1.1.0) — github.com/xiaochuanle/DeepPlant
- **Oxford Nanopore Technologies (ONT) R10.4 FlowCell** (Sequencing platform whose long reads enable methylation profiling of centromeric and repetitive regions inaccessible to short-read BS-seq)
- **Dorado** (Baseline plant 5mC caller on R10.4 used for benchmark comparison of CHH correlation and runtime) — nanoporetech/dorado

## Examples

```
docker run -i -t -v /data:/data:rw --gpus all chenhx26/deepplant:1.1.0 /bin/bash -c "DeepPlant extract_and_call_mods /data/pod5 /data/aligned.bam /data/T2T-NIP.fa DNA /data/out /DeepPlant/model/bilstm"
```

## Evaluation signals

- Reproduced centromeric CHH coverage ratio for DeepPlant matches the deposited Fig. 3b value of 88.0%, and BS-seq matches 37.7%, with non-centromeric coverage around 98%
- Per-site coverage threshold of 10 applied consistently to both DeepPlant calls and BS-seq before computing coverage fractions
- Recomputed strand-bias pattern matches Fig. 3e source data: Ac/Ds and Mariner higher CHH methylation on the forward strand; LINE and Ty1-copia higher on the reverse strand
- Output TSV contains the expected columns (read_id, chromosome, pos_in_reference, strand, methylation_rate) and CHH calls restricted to chromosomes 1-12 centromeric coordinates
- Whole-genome CHH methylation frequency correlation with BS-seq falls in the reported 0.705-0.838 range across species

## Limitations

- DeepPlant is less computationally efficient than Dorado — it took much longer to call methylation on the same dataset (Supplementary Data 10)
- Scarcity of high-methylation CHH-positive training samples continues to pose challenges for model training and generalization across species
- Reliance on species with naturally abundant CHH methylation may not capture the full diversity of plant methylation patterns; in vitro enzyme-catalyzed methylation is ineffective for CHH because CHH methylation depends on RdDM

## Evidence

- [readme] Nanopore sequencing enables comprehensive detection of 5-methylcytosine (5mC), particularly in transposable elements and centromeric regions.: "Nanopore sequencing enables comprehensive detection of 5-methylcytosine (5mC), particularly in transposable elements and centromeric regions."
- [intro] BS-seq short-read limitation: [evidence span withheld — anchor status: not_found]
- [methods] Docker run command: "docker run -i -t -v data_path:data_path:rw --gpus all chenhx26/deepplant:1.1.0 /bin/bash"
- [methods] extract_and_call_mods usage: "extract_and_call_mods [--help] [--version] pod5_dir bam_path reference_path ref_type write_dir model_dir"
- [discussion] Runtime limitation: "The computational benchmark showed that DeepPlant is less computationally efficient than Dorado as it took much longer to call methylation on the same dataset (Supplementary Data 10)"
- [abstract] CHH correlation improvement: "DeepPlant achieves high whole-genome methylation frequency correlations (0.705-0.838) with BS-seq data on CHH, improved by 23.4- 117.6% compared to Dorado"
