---
name: strand-specific-methylation-bias-analysis
description: Use when per-site methylation calls (read_id, chromosome, pos_in_reference,
  strand, methylation_rate) from nanopore 5mC detection need to be intersected with
  element annotations (TE classes, gene regions) to test for strand-biased methylation
  in a specific context such as CHH.
when_to_use_negative:
- Input is raw POD5/BAM data without methylation calls — run DeepPlant's extract_and_call_mods
  first to produce the per-site TSV
- Methylation calls were produced by a tool without validated plant CHH performance
  (e.g., default Dorado models) — DeepPlant improves whole-genome CHH correlation
  with BS-seq by 23.4–117.6% over Dorado
- The question requires absolute methylation levels across species rather than within-genome
  strand-bias comparisons; DeepPlant's CHH model was trained on species screened for
  abundant high-methylation CHH sites and may not capture full pattern diversity across
  all plants
edam_operation: http://edamontology.org/operation_3207
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0622
tools:
- name: DeepPlant
  role: Produces the per-read CHH methylation calls (call_mods TSV) on ONT R10.4 plant
    data that feed the strand-bias analysis
  repo: https://github.com/xiaochuanle/DeepPlant
- name: pandas
  role: Loads and groups the call_mods TSV by element type and strand to average methylation_rate
- name: pybedtools/bedtools
  role: Performs interval-overlap intersection of CHH cytosine positions with TE and
    protein-coding annotations
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-58576-x
    title: Accurate cross-species 5mC detection for Oxford Nanopore sequencing in
      plants with DeepPlant
  source_content_hash: sha256:89ada34dc7351345334cf6f515ebc7946f3002a96174e01e5d15958a34d63cce
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xiaochuanle/DeepPlant
  tools:
  - DeepPlant
  - pandas
  - pybedtools/bedtools
  edam_operation: http://edamontology.org/operation_3207
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# strand-specific-methylation-bias-analysis

## Summary

Quantifies strand-specific CHH methylation frequency within annotated genomic elements (e.g., centromeric transposon types and protein-coding controls) using per-read methylation calls from DeepPlant, revealing element-specific forward/reverse methylation bias in plant genomes.

## When to use

Use when per-site methylation calls (read_id, chromosome, pos_in_reference, strand, methylation_rate) from nanopore 5mC detection need to be intersected with element annotations (TE classes, gene regions) to test for strand-biased methylation in a specific context such as CHH.

## When NOT to use

- Input is raw POD5/BAM data without methylation calls — run DeepPlant's extract_and_call_mods first to produce the per-site TSV
- Methylation calls were produced by a tool without validated plant CHH performance (e.g., default Dorado models) — DeepPlant improves whole-genome CHH correlation with BS-seq by 23.4–117.6% over Dorado
- The question requires absolute methylation levels across species rather than within-genome strand-bias comparisons; DeepPlant's CHH model was trained on species screened for abundant high-methylation CHH sites and may not capture full pattern diversity across all plants

## Inputs

- DeepPlant call_mods TSV with columns read_id, chromosome, pos_in_reference, strand, methylation_rate (CHH context calls from ONT R10.4 data)
- Genomic annotation intervals (BED/GFF) for element types, e.g., centromeric TE classes (Ac/Ds, Mariner, LINE, Ty1-copia) and protein-coding regions
- Reference genome assembly (e.g., O. sativa T2T-NIP) for coordinate consistency

## Outputs

- Per-element-type, per-strand average CHH methylation frequency table (element type × strand)
- Strand-bias direction (forward vs. reverse) for each element type
- Validation comparison against published source data (e.g., Fig. 3e source data at Zenodo 10.5281/zenodo.15062213)

## How to apply

Load the DeepPlant call_mods TSV output — containing read_id, chromosome, pos_in_reference, strand, and methylation_rate — with pandas, filtering for the methylation context of interest (e.g., CHH sites on the O. sativa T2T-NIP genome). Intersect the cytosine positions with element annotation intervals (e.g., seven centromeric TE types including Ac/Ds, Mariner, LINE, and Ty1-copia, plus non-TE protein-coding control regions) using interval-overlap logic such as pybedtools/bedtools intersect. Group the overlapped sites by element type and aligned strand (+/-) and average methylation_rate within each group to obtain strand-specific methylation frequency per element. Determine the forward-vs-reverse bias direction per element type. Rationale: DeepPlant's per-read, per-strand methylation calls on R10.4 data enable methylation profiling of repetitive centromeric/TE regions where short-read BS-seq is limited; validation should compare the recomputed bias pattern against published source data (e.g., Fig. 3e, Zenodo 10.5281/zenodo.15062213).

## Related tools

- **DeepPlant** (Produces the per-read CHH methylation calls (call_mods TSV) on ONT R10.4 plant data that feed the strand-bias analysis) — https://github.com/xiaochuanle/DeepPlant
- **pandas** (Loads and groups the call_mods TSV by element type and strand to average methylation_rate)
- **pybedtools/bedtools** (Performs interval-overlap intersection of CHH cytosine positions with TE and protein-coding annotations)

## Examples

```
import pandas as pd; df = pd.read_csv('call_mods_chh.tsv', sep='\t'); sites = df[df.context=='CHH']; grouped = sites.groupby(['element_type','strand'])['methylation_rate'].mean()
```

## Evaluation signals

- Grouped methylation frequencies reproduce the published direction pattern: Ac/Ds and Mariner show higher CHH methylation on the forward strand, while LINE and Ty1-copia show higher methylation on the reverse strand
- No overall forward-vs-reverse strand bias across all seven centromeric TE types combined
- Recomputed values match the Fig. 3e source data from Zenodo 10.5281/zenodo.15062213
- Every overlapped CHH site falls within exactly one defined element interval per annotation pass (no coordinate-system mismatches between call_mods positions and annotations)
- Both + and - strand groups are non-empty for each element type (sufficient sites per strand for a reliable frequency estimate)

## Limitations

- DeepPlant is less computationally efficient than Dorado — it took much longer to call methylation on the same dataset (Supplementary Data 10), so generating input calls at scale is costly
- Findings are tied to the reference annotation quality; strand assignment depends on the alignment strand recorded in call_mods output, so alignment/annotation strand conventions must be consistent
- Short-read BS-seq cannot reliably profile complex repetitive regions such as centromeres and TEs, so independent bisulfite validation of centromeric strand bias is limited

## Evidence

- [other] Load the DeepPlant call_mods TSV output containing read_id, chromosome, pos_in_reference, strand, and methylation_rate for CHH sites on the O. sativa T2T-NIP genome using pandas: [evidence span withheld — anchor status: not_found]
- [other] Intersect CHH cytosine positions with centromeric annotation intervals: [evidence span withheld — anchor status: not_found]
- [other] Ac/Ds and Mariner forward-strand bias finding: [evidence span withheld — anchor status: not_found]
- [intro] BS-seq limitation in repetitive regions motivating nanopore-based methylation profiling: [evidence span withheld — anchor status: not_found]
- [intro] DeepPlant outperforms Dorado on CHH methylation frequency correlation: "DeepPlant, which outperforms Dorado, achieving a correlation improvement ranging from 0.135 to 0.381 with BS-seq in whole-genome CHH methylation frequency quantification"
