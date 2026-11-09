---
name: kmer-context-window-parsing
description: Use when per-read event observations from nanopolish eventalign must
  be grouped into mCaller classification windows (num_variables=6 → 11-mer) and filtered
  by skip count before or during m6A classification.
when_to_use_negative:
- Input is already classified mCaller output (e.g. the bed file from make_bed.py)
  rather than per-read eventalign observations
- The analysis goal is per-position methylation fraction summarization rather than
  per-read observation filtering
- Data were not event-aligned with nanopolish --scale-events, so no per-read event-to-kmer
  assignment exists to count skips from
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: mCaller
  role: Defines the skip_thresh rule (-s, number of skips to allow within an observation,
    default 0) and the num_variables context window; mCaller.py implements the filter
    during per-read classification
  repo: https://github.com/al-mcintyre/mCaller
- name: nanopolish
  role: Produces the eventalign TSV with scaled event values that supplies the per-read
    observations whose 11-mer windows are parsed for skips
  repo: https://github.com/jts/nanopolish
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-08289-9
    title: Single-molecule sequencing detection of N6-methyladenine in microbial reference
      materials
  source_content_hash: sha256:e9c1c6d2e145003090c8034673b8c269723a383d06199a5cc2bf58f09782d2dc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/al-mcintyre/mCaller
  tools:
  - mCaller
  - nanopolish
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# kmer-context-window-parsing

## Summary

Reconstructs mCaller's per-read k-mer context window from nanopolish eventalign TSV observations and counts skip events (6-mers with no recorded current) within each 11-mer window so that the skip_thresh rule from mCaller.py can be applied. It is the mechanism by which the NN_sk variant's tolerance of up to two skips per observation is enforced or tightened.

## When to use

Use when per-read event observations from nanopolish eventalign must be grouped into mCaller classification windows (num_variables=6 → 11-mer) and filtered by skip count before or during m6A classification.

## When NOT to use

- Input is already classified mCaller output (e.g. the bed file from make_bed.py) rather than per-read eventalign observations
- The analysis goal is per-position methylation fraction summarization rather than per-read observation filtering
- Data were not event-aligned with nanopolish --scale-events, so no per-read event-to-kmer assignment exists to count skips from

## Inputs

- nanopolish eventalign TSV produced with --scale-events (per-read event observations)
- Reference FASTA used for context reconstruction
- skip_thresh value (mCaller -s flag, default 0; NN_sk variant uses 2)
- num_variables setting (default 6, defining the 11-mer context window)

## Outputs

- skip_filtered_observations.tsv with retained per-read observations (chromosome, read name, position, context, features, strand)
- Per-observation skip counts within the 11-mer classification window

## How to apply

Load the nanopolish eventalign TSV (produced with --scale-events) and group per-read event observations at the target positions. For each observation, rebuild the k-mer context window using the mCaller num_variables setting — the default of 6 variables corresponds to an 11-mer context (6*2-1) — and count the skip events inside that window, where a skip is a 6-mer with no recorded current value. Apply the skip_thresh rule from mCaller.py (-s, number of skips to allow within an observation, default 0): discard any observation whose skip count exceeds the threshold; the relaxed NN_sk variant permits up to 2 skips. Write retained observations in the same per-read layout (chromosome, read name, position, context, features, strand) to skip_filtered_observations.tsv. The decision point is the threshold value: the default of 0 is the strict standard model, while 2 reproduces the NN_sk model's permissive handling of skipped 6-mers.

## Related tools

- **mCaller** (Defines the skip_thresh rule (-s, number of skips to allow within an observation, default 0) and the num_variables context window; mCaller.py implements the filter during per-read classification) — https://github.com/al-mcintyre/mCaller
- **nanopolish** (Produces the eventalign TSV with scaled event values that supplies the per-read observations whose 11-mer windows are parsed for skips) — https://github.com/jts/nanopolish

## Examples

```
python mCaller.py -m GATC -r reference.fasta -d r95_twobase_model_NN_6_m6A.pkl -e sample.eventalign.tsv -f sample.fastq -b A -s 0
```

## Evaluation signals

- Every retained observation contains at most skip_thresh skip events within its classification window (default 0, or 2 for the NN_sk variant)
- All observations whose 11-mer window skip count exceeds the threshold are absent from skip_filtered_observations.tsv
- Retained rows preserve the mCaller per-read layout: chromosome, read name, position, context, features, strand
- Context window length matches num_variables: 6 variables reconstructs an 11-mer (6*2-1) per-read context
- Observation counts before and after filtering can be reconciled by binning eventalign rows by their skip count

## Limitations

- Skips are only meaningful where nanopolish eventalign assigned events per read; missing or unscaled eventalign input prevents reliable skip counting
- The relaxed NN_sk setting (allowing 2 skips) relaxes the default strictness (0 skips) and may retain partially observed 11-mer windows, which the paper notes is a model variant rather than the default behavior
- Filtering at the observation level does not compensate for low read depth at a position; separate read-depth thresholds apply

## Evidence

- [methods] skip_thresh definition in mCaller.py options: "-s, --skip_thresh 
 number of skips to allow within an observation (default 0)"
- [methods] eventalign TSV generation with scaled events: "nanopolish eventalign -t <num_threads> --scale-events -n -r <filename>.fastq -b <filename>.sorted.bam -g <reference>.fasta"
- [readme] mCaller README command-line interface including skip threshold flag: "[-s SKIP_THRESH] [-q QUAL_THRESH] [-c CLASSIFIER]"
- [readme] mCaller README statement of the current-deviation calling principle underlying per-read observation classification: "This program is designed to call m6A from nanopore data using the differences between measured and expected currents."
