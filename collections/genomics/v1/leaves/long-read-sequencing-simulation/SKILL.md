---
name: long-read-sequencing-simulation
description: Use when benchmarking a somatic SV caller against alternatives and no
  matched tumor/control long-read data with a validated truth set exists; simulation
  lets you control tumor purity (e.g. 0–100%) and sequence yield (e.g. 10–50x).
when_to_use_negative:
- You already have matched tumor/control long-read data with a trusted somatic SV
  truth set (e.g. manually curated or GIAB-style benchmarks) — real-data validation
  is preferable
- The caller under evaluation is a germline-only SV caller with no tumor/control paired
  mode or subtraction workflow
- You need to evaluate long repeat expansion events (e.g. VNTR/microsatellite expansions),
  which the reference-genome-based simulation framework may not capture reliably
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: SURVIVOR
  role: simSV module to inject germline and somatic SVs into GRCh38 and build the
    in-silico tumor/control genomes
  repo: https://github.com/fritzsedlazeck/SURVIVOR
- name: NanoSim
  role: v2.6.0 read_analysis.py to train ONT error models and simulator.py to generate
    Nanopore-like tumor/control reads
  repo: https://github.com/BirolLab/NanoSim
- name: minimap2
  role: aligns the simulated FASTQ files to GRCh38 to produce BAMs for subsampling
    and calling
  repo: https://github.com/lh3/minimap2
- name: nanomonsv
  role: 'method under evaluation: somatic SV caller run on every subsampled tumor/control
    BAM pair'
  repo: https://github.com/friend1ws/nanomonsv
- name: Sniffles2
  role: baseline separate-detection-and-subtraction caller run with '--minsupport
    1 --non-germline'
  repo: https://github.com/fritzsedlazeck/Sniffles
- name: cuteSV
  role: baseline subtraction caller with min_support=1, max_cluster_bias_INS/DEL 100,
    diff_ratio_merging 0.3
- name: Delly
  role: baseline 'lr' mode caller run on tumor and control separately
- name: SVIM
  role: baseline 'alignment' mode caller with '--skip_genotyping' run on tumor and
    control separately
- name: CAMPHORsomatic
  role: somatic long-read SV caller baseline run with default settings
  repo: https://github.com/afujimoto/CAMPHORsomatic
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkad526
    title: Precise characterization of somatic complex structural variations from
      tumor/control paired long-read sequencing data with nanomonsv
  source_content_hash: sha256:ce58ef8cc7b8d2d69faff9b2bd10f39fff002ea2d48589398bfd5c89be19cfc9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fritzsedlazeck/SURVIVOR
  tools:
  - CAMPHORsomatic
  - Delly
  - NanoSim
  - SURVIVOR
  - SVIM
  - Sniffles
  - Sniffles2
  - cuteSV
  - minimap2
  - nanomonsv
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# long-read-sequencing-simulation

## Summary

Generate in-silico tumor/control matched long-read sequencing benchmarks by injecting germline and somatic SVs with SURVIVOR, simulating Nanopore-like reads with NanoSim, and subsampling to controlled coverage and tumor purity, enabling precision/recall evaluation of somatic SV callers against a known truth set.

## When to use

Use when benchmarking a somatic SV caller against alternatives and no matched tumor/control long-read data with a validated truth set exists; simulation lets you control tumor purity (e.g. 0–100%) and sequence yield (e.g. 10–50x).

## When NOT to use

- You already have matched tumor/control long-read data with a trusted somatic SV truth set (e.g. manually curated or GIAB-style benchmarks) — real-data validation is preferable
- The caller under evaluation is a germline-only SV caller with no tumor/control paired mode or subtraction workflow
- You need to evaluate long repeat expansion events (e.g. VNTR/microsatellite expansions), which the reference-genome-based simulation framework may not capture reliably

## Inputs

- Reference genome FASTA (GRCh38, 22 autosomes + chromosome X)
- Real Nanopore reads (e.g. ENA PRJEB27698 COLO829BL ONT reads) for NanoSim model training
- SURVIVOR simSV germline and somatic SV injection parameters
- Simulated tumor and control FASTQ files aligned to GRCh38 as BAM (for subsampling)

## Outputs

- In-silico tumor and matched-control genomes with known SV coordinates
- Nanopore-like simulated FASTQ reads for tumor and control (~180 Gb yields)
- Subsampled BAM sets at 10x/20x/30x/40x/50x coverage and 0–100% tumor purity
- Somatic SV truth set VCF and per-method precision/recall tables (cf. Supplementary Figures S10–S12)

## How to apply

Build an in-silico diploid matched-control genome from GRCh38 (22 autosomes + chromosome X) by injecting germline SVs using SURVIVOR v1.0.6 simSV (2500 duplications, 5000 indels, 100 inversions, 50 inversion-deletions, 50 inversion-duplications), then inject somatic SVs (100 duplications, 200 indels, 100 translocations, 100 inversions) to create the in-silico tumor genome and convert somatic SV coordinates back to GRCh38. Train NanoSim 2.6.0 parameters on real Nanopore reads (e.g. COLO829BL ONT reads from ENA PRJEB27698 aligned to chromosome 22) with read_analysis.py, then use simulator.py to generate tumor and control reads (~180 Gb yields). Align simulated FASTQ files to GRCh38 with minimap2 and sub-sample the tumor/control BAMs across sequencing depths (10x, 20x, 30x, 40x, 50x) and tumor purities (0%, 20%, 40%, 60%, 80%, 100%). Run each SV caller on every subsampled tumor/control BAM pair and compute precision and recall against the simulated somatic SV truth set.

## Related tools

- **SURVIVOR** (simSV module to inject germline and somatic SVs into GRCh38 and build the in-silico tumor/control genomes) — https://github.com/fritzsedlazeck/SURVIVOR
- **NanoSim** (v2.6.0 read_analysis.py to train ONT error models and simulator.py to generate Nanopore-like tumor/control reads) — https://github.com/BirolLab/NanoSim
- **minimap2** (aligns the simulated FASTQ files to GRCh38 to produce BAMs for subsampling and calling) — https://github.com/lh3/minimap2
- **nanomonsv** (method under evaluation: somatic SV caller run on every subsampled tumor/control BAM pair) — https://github.com/friend1ws/nanomonsv
- **Sniffles2** (baseline separate-detection-and-subtraction caller run with '--minsupport 1 --non-germline') — https://github.com/fritzsedlazeck/Sniffles
- **cuteSV** (baseline subtraction caller with min_support=1, max_cluster_bias_INS/DEL 100, diff_ratio_merging 0.3)
- **Delly** (baseline 'lr' mode caller run on tumor and control separately)
- **SVIM** (baseline 'alignment' mode caller with '--skip_genotyping' run on tumor and control separately)
- **CAMPHORsomatic** (somatic long-read SV caller baseline run with default settings) — https://github.com/afujimoto/CAMPHORsomatic

## Examples

```
python simulator.py --model Nanopore_model training.bam.md genome.msi GRCh38_simulation_conf 1 60 --output tumor_sim
```

## Evaluation signals

- Precision and recall of each method at every coverage/purity point computed against the simulated somatic SV truth set (Supplementary Figures S10–S12)
- Validation criterion: nanomonsv's precision and recall must be superior to the subtraction approach and CAMPHORsomatic on the simulated data
- Expected error pattern: precision remains relatively stable while recall decreases at low tumor purity and low sequence yield
- Simulated germline SV counts must match the injection spec (2500 duplications, 5000 indels, 100 inversions, 50 inversion-deletions, 50 inversion-duplications) and somatic SV counts (100 duplications, 200 indels, 100 translocations, 100 inversions)
- Subsampled BAM coverage/purity grid must span 10x–50x and 0%–100% tumor purity as designed

## Limitations

- At low tumor purities and low sequence yields, recall of somatic SVs drops even when precision stays stable, so the simulation cannot rescue sensitivity at shallow/subclonal settings
- A reference-genome-based simulation framework may not appropriately capture long repeat expansion events (e.g. somatic VNTR/microsatellite expansions) because the reference genome is unreliable at such loci
- NanoSim parameter training requires real ONT reads; pretrained NanoSim models require scikit-learn <=0.22.1, and newer scikit-learn raises 'No module named sklearn.neighbors.kde'
- Detection of subclonal SVs present in a minority of cells remains challenging at modest sequencing depth, so simulation at low purity only partially reflects real subclonal calling difficulty

## Evidence

- [supplementary] On simulation data with varying tumor purity and sequence yields, nanomonsv showed superior precision and recall compared to other approaches; precision remained stable even at low tumor purities and low sequence yields, where recall dropped.: [evidence span withheld — anchor status: not_found]
- [readme] NanoSim v2.6 simulates ONT reads in FASTQ format with base quality learned from real data.: [evidence span withheld — anchor status: external_doc]
- [readme] SURVIVOR is a tool set for simulating and evaluating SVs.: [evidence span withheld — anchor status: external_doc]
- [intro] Separate detection and subtraction approaches generate many false positives, motivating simulation benchmarks of paired tumor/control callers.: "this approach can generate many false positives, such as germline SVs that pass the threshold in the tumor and narrowly miss it in the matched control"
- [readme] Pretrained NanoSim models require an older scikit-learn version.: [evidence span withheld — anchor status: external_doc]
