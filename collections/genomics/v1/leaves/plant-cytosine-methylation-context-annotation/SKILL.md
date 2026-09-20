---
name: plant-cytosine-methylation-context-annotation
description: Use when you have high-confidence methylation-positive/negative cytosine
  site lists from plant ONT R10.4 data and need to classify sites into CpG/CHG/CHH
  contexts or enumerate and count 9-mer flanking-context motifs for training-set coverage
  analysis.
when_to_use_negative:
- Sites come from CpG or CHG contexts and you only need motif frequency correlation
  against BS-seq — use the whole-genome methylation frequency comparison workflow
  instead
- You lack a reference genome, so flanking 9-mer sequence context cannot be extracted
  from site coordinates
- Your data are not plant ONT R10.4 5mC calls with bisulfite-derived high-confidence
  labels — context coverage statistics like the 97.2% figure are specific to that
  training-data construction
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_3295
tools:
- name: DeepPlant
  role: Produces the 5mC methylation calls whose CHH-positive/negative sites are annotated
    by sequence context; its training set is characterized by 9-mer CHH context coverage
  repo: https://github.com/xiaochuanle/DeepPlant
- name: extract_hc_sites
  role: DeepPlant subcommand that extracts high-confidence positive/negative sites
    from pod5/BAM/reference inputs, taking kmer_size as a parameter to set context
    length
  repo: https://github.com/xiaochuanle/DeepPlant
- name: Dorado
  role: Baseline ONT basecaller/modification caller for plant 5mC on R10.4, against
    which context-resolved CHH performance is compared
  repo: https://github.com/nanoporetech/dorado
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-58576-x
    title: Accurate cross-species 5mC detection for Oxford Nanopore sequencing in
      plants with DeepPlant
  source_content_hash: sha256:9a1d618f6bc6389b819fdff64fcf3157a7119403d5ca1288755d75beb3813b4c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xiaochuanle/DeepPlant
  tools:
  - DeepPlant
  - Dorado
  - extract_hc_sites
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# plant-cytosine-methylation-context-annotation

## Summary

Annotates plant nanopore methylation calls by cytosine sequence context (CpG, CHG, CHH) and flanking 9-mer motifs, enabling coverage accounting and context-specific model training/evaluation such as DeepPlant's 9-mer CHH context tally over 36,864 possible motifs.

## When to use

Use when you have high-confidence methylation-positive/negative cytosine site lists from plant ONT R10.4 data and need to classify sites into CpG/CHG/CHH contexts or enumerate and count 9-mer flanking-context motifs for training-set coverage analysis.

## When NOT to use

- Sites come from CpG or CHG contexts and you only need motif frequency correlation against BS-seq — use the whole-genome methylation frequency comparison workflow instead
- You lack a reference genome, so flanking 9-mer sequence context cannot be extracted from site coordinates
- Your data are not plant ONT R10.4 5mC calls with bisulfite-derived high-confidence labels — context coverage statistics like the 97.2% figure are specific to that training-data construction

## Inputs

- High-confidence methylation-positive and -negative cytosine site lists per species (from extract_hc_sites on ONT pod5/BAM data with BS-seq truth)
- Reference genome FASTA for extracting k-mer sequence context around each cytosine
- Context size parameter k (e.g. kmer_size=9 for 9-mer CHH contexts)

## Outputs

- Per-cytosine context annotations (CpG / CHG / CHH with 9-mer flanking motif)
- Per-9-mer positive and negative support-count tallies
- Coverage summary report CSV (e.g. chh_9mer_coverage_report.csv) with covered count, coverage fraction, and per-9-mer support counts

## How to apply

Load the merged high-confidence site lists (from S. miltiorrhiza, S. tuberosum, R. communis or similar plant species) with pandas/numpy and extract the k-mer sequence context centered on each cytosine from the reference. Classify each cytosine as CpG, CHG, or CHH using the canonical H={A,C,T} rule, and for CHH coverage analysis enumerate all 9-mer contexts programmatically as 9 CHH central trinucleotides x 4^6 flanking-base combinations via itertools.product over A/C/G/T (36,864 total). Tally each 9-mer's occurrence count separately in the positive and negative site sets, then flag a 9-mer as covered only when it reaches a minimum-support threshold of >=50 occurrences in BOTH sets. Divide the covered count by 36,864 to get the coverage fraction; DeepPlant's combined three-species training dataset achieves 97.2% coverage averaging 9225 samples per context. Write per-9-mer support counts and the summary coverage to a report CSV (e.g. chh_9mer_coverage_report.csv).

## Related tools

- **DeepPlant** (Produces the 5mC methylation calls whose CHH-positive/negative sites are annotated by sequence context; its training set is characterized by 9-mer CHH context coverage) — https://github.com/xiaochuanle/DeepPlant
- **extract_hc_sites** (DeepPlant subcommand that extracts high-confidence positive/negative sites from pod5/BAM/reference inputs, taking kmer_size as a parameter to set context length) — https://github.com/xiaochuanle/DeepPlant
- **Dorado** (Baseline ONT basecaller/modification caller for plant 5mC on R10.4, against which context-resolved CHH performance is compared) — https://github.com/nanoporetech/dorado

## Examples

```
extract_hc_sites pod5_dir bam_path reference_path plant pos.txt neg.txt 9 16  # then in python: kmers = [''.join(p) for p in itertools.product('ACGT', repeat=6)]
```

## Evaluation signals

- The enumerated context space equals exactly 36,864 9-mers (9 CHH central trinucleotides x 4^6 flanking combinations) — a count other than 36,864 indicates an enumeration bug
- A 9-mer is counted as covered only if it has >=50 occurrences in BOTH the positive and negative sets; covered_count / 36864 should reproduce ~97.2% for the combined S. miltiorrhiza + S. tuberosum + R. communis dataset
- Average samples per covered context should approximate the reported ~9225
- Every cytosine in the site list must resolve to exactly one of CpG/CHG/CHH (H = A, C, or T); unclassified sites indicate reference-coordinate or strand errors
- Output CSV must contain per-9-mer positive and negative support counts plus the overall coverage fraction

## Limitations

- Reliance on species with naturally abundant CHH methylation may not capture the full diversity of methylation patterns across all plant species
- In vitro enzyme-catalyzed DNA methylation is ineffective for generating CHH positive samples because CHH methylation depends on RNA-directed DNA methylation (RdDM)
- Scarcity of high-methylation CHH samples, although partially mitigated, continues to pose challenges for model training and generalization

## Evidence

- [discussion] Our comprehensive training dataset now covers 97.2% of all possible 9-mer CHH contexts, averaging over 9225 samples per context: "Our comprehensive training dataset now covers 97.2% of all possible 9-mer CHH contexts, averaging over 9225 samples per context"
- [intro] The new dataset covers 97.2% of all possible 9-mers centered with CHH: "We generate new Nanopore and BS-seq data from selected species, significantly increasing the number of CHH-positive samples. The new dataset covers 97.2% of all possible 9-mers centered with CHH"
- [methods] Usage: extract_hc_sites [--help] [--version] pod5_dir bam_path reference_path ref_type write_dir pos neg kmer_size num_workers: "Usage: extract_hc_sites [--help] [--version] pod5_dir bam_path reference_path ref_type write_dir pos neg kmer_size num_workers"
- [abstract] We address the scarcity of methylation-positive CHH training samples through screening species with abundant high-methylation CHH sites using bisulfite-sequencing and generate datasets that cover: "We address the scarcity of methylation-positive CHH training samples through screening species with abundant high-methylation CHH sites using bisulfite-sequencing and generate datasets that cover"
- [readme] reanalyzed bisulfite sequencing (BS-seq) data to screen species with abundant high-methylation CHH sites, generating new datasets that cover diverse 9-mer motifs: "reanalyzed bisulfite sequencing (BS-seq) data to screen species with abundant high-methylation CHH sites, generating new datasets that cover diverse 9-mer motifs"
