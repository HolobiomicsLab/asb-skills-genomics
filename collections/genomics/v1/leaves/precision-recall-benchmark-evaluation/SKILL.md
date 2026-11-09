---
name: precision-recall-benchmark-evaluation
description: Use when you must quantitatively compare somatic SV callers (e.g. nanomonsv
  vs detection-subtraction baselines or CAMPHORsomatic) against a known truth set
  of simulated variants.
when_to_use_negative:
- Evaluation is on real tumor data without a known somatic SV truth set — precision/recall
  cannot be computed and comparison must use orthogonal validation instead
- The question is SV breakpoint resolution or structural characterization rather than
  caller sensitivity/precision comparison
- Short-read sequencing data — this benchmark design is built around ONT long-read
  simulation (NanoSim) and long-read callers (Sniffles2, cuteSV, Delly lr, SVIM)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0091
tools:
- name: SURVIVOR
  role: simSV injection of germline SVs into the in-silico control genome (v1.0.6)
  repo: https://github.com/fritzsedlazeck/SURVIVOR
- name: NanoSim
  role: ONT read model training (read_analysis.py) and simulation of Nanopore-like
    tumor/control reads (simulator.py, v2.6.0)
  repo: https://github.com/BirolLab/NanoSim
- name: minimap2
  role: alignment of simulated FASTQ reads to GRCh38 before sub-sampling BAMs
- name: nanomonsv
  role: 'method under test: somatic SV calling on every tumor/control BAM pair'
  repo: https://github.com/friend1ws/nanomonsv
- name: Sniffles2
  role: detection-subtraction baseline run on tumor and control separately with '--minsupport
    1 --non-germline'
  repo: https://github.com/fritzsedlazeck/Sniffles
- name: CAMPHORsomatic
  role: paired tumor/control baseline run with default settings
  repo: https://github.com/afujimoto/CAMPHORsomatic
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkad526
    title: Precise characterization of somatic complex structural variations from
      tumor/control paired long-read sequencing data with nanomonsv
  source_content_hash: sha256:e190d42d234a69fabcbbfcb6c8abdad2fd23b86dc3169f6f1672eb5b1bfe4ce2
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# precision-recall-benchmark-evaluation

## Summary

Benchmark somatic SV callers by tabulating precision and recall against a simulated truth set across tumor purities and sequencing depths, using NanoSim-simulated ONT tumor/control reads with germline and somatic SVs injected via SURVIVOR.

## When to use

Use when you must quantitatively compare somatic SV callers (e.g. nanomonsv vs detection-subtraction baselines or CAMPHORsomatic) against a known truth set of simulated variants.

## When NOT to use

- Evaluation is on real tumor data without a known somatic SV truth set — precision/recall cannot be computed and comparison must use orthogonal validation instead
- The question is SV breakpoint resolution or structural characterization rather than caller sensitivity/precision comparison
- Short-read sequencing data — this benchmark design is built around ONT long-read simulation (NanoSim) and long-read callers (Sniffles2, cuteSV, Delly lr, SVIM)

## Inputs

- Reference genome GRCh38 FASTA (22 autosomes + chromosome X)
- Real ONT long-read data for model training (e.g. COLO829BL Nanopore FASTQ/BAM, ENA PRJEB27698)
- Subsampled tumor/control BAM pairs at 10x/20x/30x/40x/50x coverage and 0%–100% tumor purity
- Simulated somatic SV truth set (duplications, indels, translocations, inversions in GRCh38 coordinates)

## Outputs

- Precision and recall values per caller per coverage/purity point (tabulated, cf. Supplementary Figures S10–S12)
- Somatic SV call sets (VCF-like) from each evaluated caller on every subsampled BAM pair

## How to apply

First build an in-silico diploid matched-control genome from GRCh38 (22 autosomes + chromosome X) by injecting germline SVs with SURVIVOR v1.0.6 simSV (2500 duplications, 5000 indels, 100 inversions, 50 inversion-deletions, 50 inversion-duplications), then inject somatic SVs (100 duplications, 200 indels, 100 translocations, 100 inversions) and convert somatic coordinates back to GRCh38. Train NanoSim 2.6.0 on real ONT reads (e.g. COLO829BL Nanopore reads from ENA PRJEB27698 aligned to chromosome 22 with read_analysis.py) and generate tumor/control Nanopore-like FASTQ (~180 Gb) with simulator.py. Align simulated reads to GRCh38 with minimap2 and sub-sample the tumor/control BAMs to coverages of 10x–50x and tumor purities of 0%–100%. Run the method under test on every subsampled tumor/control BAM pair, and run baselines identically (Sniffles2 '--minsupport 1 --non-germline'; cuteSV '--min_support = 1' with max_cluster_bias_INS/DEL 100 and diff_ratio_merging 0.3; Delly 'lr'; SVIM 'alignment' with '--skip_genotyping', subtracting control calls with 200 bp margins; CAMPHORsomatic defaults). Compute precision and recall at every coverage/purity point against the simulated somatic SV truth set; validate that precision stays stable at low purity/depth while recall drops, and that the method under test beats the subtraction approach and CAMPHORsomatic.

## Related tools

- **SURVIVOR** (simSV injection of germline SVs into the in-silico control genome (v1.0.6)) — https://github.com/fritzsedlazeck/SURVIVOR
- **NanoSim** (ONT read model training (read_analysis.py) and simulation of Nanopore-like tumor/control reads (simulator.py, v2.6.0)) — https://github.com/BirolLab/NanoSim
- **minimap2** (alignment of simulated FASTQ reads to GRCh38 before sub-sampling BAMs)
- **nanomonsv** (method under test: somatic SV calling on every tumor/control BAM pair) — https://github.com/friend1ws/nanomonsv
- **Sniffles2** (detection-subtraction baseline run on tumor and control separately with '--minsupport 1 --non-germline') — https://github.com/fritzsedlazeck/Sniffles
- **CAMPHORsomatic** (paired tumor/control baseline run with default settings) — https://github.com/afujimoto/CAMPHORsomatic

## Examples

```
sniffles -i tumor.subsampled.30x.purity40.bam -v tumor.sniffles2.vcf --minsupport 1 --non-germline
```

## Evaluation signals

- Precision of nanomonsv is superior to the separate detection-subtraction approach and CAMPHORsomatic at every simulated coverage/purity point
- Precision remains relatively stable even at low tumor purity and low sequence yield, while recall decreases under those conditions
- Truth-set concordance: recall is computed against the exact set of injected somatic SVs (100 duplications, 200 indels, 100 translocations, 100 inversions) after coordinate conversion back to GRCh38
- Subtraction baseline calls are removed only after checking 200 bp overlap margins against control calls — verify the subtraction step uses the documented margins
- Results reproduce the pattern reported in Supplementary Figures S10–S12 across the full grid (5 coverages × 6 purities)

## Limitations

- Recall drops at low tumor purity and low sequence yield even when precision is stable — the benchmark does not rescue subclonal SV detection
- The reference-genome-based framework may not capture long repeat expansion events, so such SV classes are outside the truth set
- Detections where both breakpoints lie in poorly characterized highly repetitive regions (e.g. centromeres) cannot be recovered, biasing recall estimates for those loci

## Evidence

- [intro] this approach can generate many false positives, such as germline SVs that pass the threshold in the tumor and narrowly miss it in the matched control: "this approach can generate many false positives, such as germline SVs that pass the threshold in the tumor and narrowly miss it in the matched control"
- [abstract] Canonical SV module identifies somatic SVs captured by short-read technologies with higher precision and recall than existing methods.: "we demonstrate that Canonical SV module can identify somatic SVs that can be captured by short-read technologies with higher precision and recall than existing methods"
- [supplementary] Sniffles2 baseline invocation with somatic minimum support of 1.: [evidence span withheld — anchor status: not_found]
- [readme] NanoSim captures ONT technology-specific read features used for the simulated benchmark reads.: [evidence span withheld — anchor status: external_doc]
- [readme] Sniffles2 quick-start command used for the baseline SV calls.: [evidence span withheld — anchor status: external_doc]
