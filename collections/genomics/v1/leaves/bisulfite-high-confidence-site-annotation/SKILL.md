---
name: bisulfite-high-confidence-site-annotation
description: 'Use when benchmarking nanopore 5mC callers on plant genomes and balanced
  ground-truth labels are needed: called cytosines must be intersected with BS-seq-derived
  positive (100%) and negative (0%) CHH site lists at >=5x coverage.'
when_to_use_negative:
- Whole-genome methylation frequency correlation with BS-seq is the goal rather than
  site-level ground truthing — DeepPlant's whole-genome correlation workflow (0.705-0.838
  on CHH) is the appropriate alternative
- Datasets contain no BS-seq data for the species, so balanced 100%/0% high-confidence
  CHH site lists cannot be derived (scarcity of high-methylation CHH samples remains
  a challenge)
- CpG or CHG contexts with different k-mer requirements — DeepPlant uses default CpG/CHG
  kmer sizes of 51 rather than the CHH kmer size of 13
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: DeepPlant
  role: Provides the extract_hc_sites command used to build the high-confidence site
    lists from BS-seq-derived positives/negatives, and extract_and_call_mods for the
    methylation calls being labeled
  repo: github.com/xiaochuanle/DeepPlant
- name: Dorado
  role: Baseline ONT R10.4 5mC methylation caller whose calls are intersected with
    the same high-confidence site lists for side-by-side comparison
  repo: https://github.com/nanoporetech/dorado
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-58576-x
    title: Accurate cross-species 5mC detection for Oxford Nanopore sequencing in
      plants with DeepPlant
  source_content_hash: sha256:b12b604806dbbb1c578067b018e5dc96b3becc58c50bdea68b3e8ec60cf8cd4e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepPlant
  - Dorado
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# bisulfite-high-confidence-site-annotation

## Summary

Constructs high-confidence ground-truth CHH methylation site lists from BS-seq data — fully methylated (100%) positives and unmethylated (0%) negatives — and intersects them with nanopore methylation calls from tools like DeepPlant and Dorado to compute single-molecule accuracy, precision, recall, F1, AUC, and AP across plant species.

## When to use

Use when benchmarking nanopore 5mC callers on plant genomes and balanced ground-truth labels are needed: called cytosines must be intersected with BS-seq-derived positive (100%) and negative (0%) CHH site lists at >=5x coverage.

## When NOT to use

- Whole-genome methylation frequency correlation with BS-seq is the goal rather than site-level ground truthing — DeepPlant's whole-genome correlation workflow (0.705-0.838 on CHH) is the appropriate alternative
- Datasets contain no BS-seq data for the species, so balanced 100%/0% high-confidence CHH site lists cannot be derived (scarcity of high-methylation CHH samples remains a challenge)
- CpG or CHG contexts with different k-mer requirements — DeepPlant uses default CpG/CHG kmer sizes of 51 rather than the CHH kmer size of 13

## Inputs

- BS-seq datasets (publicly available bisulfite sequencing data) for the species of interest
- Oxford Nanopore POD5 read directory
- Sorted alignment BAM file
- Reference genome FASTA and reference type
- Per-read, per-site methylation rate TSVs from DeepPlant (extract_and_call_mods) and Dorado R10.4 5mC calls
- Positive and negative high-confidence CHH site lists (>=5x coverage, 100% and 0% methylation)

## Outputs

- High-confidence CHH site lists (positive fully-methylated and negative unmethylated sets) per species
- Balanced ground-truth labels assigned to each called cytosine per species
- Single-molecule accuracy, precision, recall, F1, AUC, and AP per species and tool (e.g., single_molecule_metrics.tsv), stratified by CHH 9-mer context where applicable

## How to apply

Reanalyze publicly available bisulfite sequencing (BS-seq) data to screen species with abundant high-methylation CHH sites, then derive per-species lists of high-confidence CHH sites: positives are fully methylated (100%) sites and negatives are unmethylated (0%) sites, each with >=5x BS-seq coverage. Use DeepPlant's extract_hc_sites command, supplying the POD5 directory, sorted BAM, reference genome, reference type, output directory, positive and negative site files, a k-mer size (13 for CHH, versus 51 for CpG/CHG), and a number of workers. Intersect methylation calls from DeepPlant (extract_and_call_mods with chh_kmer_size=13, batch_size=1024, Bi-LSTM model at /DeepPlant/model/bilstm) and from ONT Dorado's R10.4 5mC model with these balanced positive/negative site lists to label every called cytosine. The rationale is that CHH-positive training/evaluation samples are scarce in plants, so bisulfite screening of species with naturally abundant high-methylation CHH sites yields the balanced labels needed for trustworthy single-molecule metric estimation.

## Related tools

- **DeepPlant** (Provides the extract_hc_sites command used to build the high-confidence site lists from BS-seq-derived positives/negatives, and extract_and_call_mods for the methylation calls being labeled) — github.com/xiaochuanle/DeepPlant
- **Dorado** (Baseline ONT R10.4 5mC methylation caller whose calls are intersected with the same high-confidence site lists for side-by-side comparison) — https://github.com/nanoporetech/dorado

## Examples

```
extract_hc_sites pod5_dir sorted.bam ref.fa genome write_dir pos_hc_sites.tsv neg_hc_sites.tsv 13 8
```

## Evaluation signals

- Positive and negative CHH site lists are restricted to sites with >=5x BS-seq coverage and exactly 100% / 0% methylation
- Called cytosines from both DeepPlant and Dorado successfully intersect with the balanced ground-truth site lists for every evaluated species
- Reproduced per-species single-molecule accuracy and F1 scores are consistent with the reported Table 2 pattern (DeepPlant superior to Dorado; F1 gains of 6.8% for O. sativa, 5.94% for B. vulgaris, 5.48% for S. lycopersicum)
- CHH k-mer size is set to 13 (not the default CpG/CHG size of 51) so site contexts are extracted with the correct 9-mer-centered windows
- The labeled site sets are balanced between fully methylated and unmethylated classes, avoiding metric inflation from class imbalance

## Limitations

- Scarcity of high-methylation CHH samples, although partially mitigated, continues to pose challenges for model training and generalization
- Reliance on species with naturally abundant CHH methylation may not capture the full diversity of methylation patterns across all plant species
- In vitro enzyme-catalyzed DNA methylation is ineffective for generating CHH positive samples because CHH methylation depends on RdDM with non-coding RNA guidance and DRM1/DRM2 methyltransferases; PCR amplification with modified base substitutions also has difficulties producing high-purity 5mC and alters base contexts
- No dataset accession IDs for the BS-seq data or generated datasets are provided in the methods

## Evidence

- [methods] extract_hc_sites usage with pos/neg site files, kmer_size and num_workers arguments: "Usage: extract_hc_sites [--help] [--version] pod5_dir bam_path reference_path ref_type write_dir pos neg kmer_size num_workers"
- [abstract] BS-seq screening motivation for high-methylation CHH site datasets: "We address the scarcity of methylation-positive CHH training samples through screening species with abundant high-methylation CHH sites using bisulfite-sequencing"
- [methods] Reanalysis of BS-seq data to generate high-confidence site datasets: "we reanalyzed bisulfite sequencing (BS-seq) data to screen species with abundant high-methylation CHH sites, generating new datasets"
- [discussion] Training coverage of CHH 9-mer contexts underpinning the kmer_size choice: "Our comprehensive training dataset now covers 97.2% of all possible 9-mer CHH contexts, averaging over 9225 samples per context"
