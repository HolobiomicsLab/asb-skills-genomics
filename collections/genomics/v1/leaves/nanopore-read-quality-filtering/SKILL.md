---
name: nanopore-read-quality-filtering
description: Use when running mCaller on ONT eventalign data and read-level noise
  from low-quality basecalls is degrading per-position methylation accuracy, and you
  want to restrict classification to reads above an average base-quality threshold.
when_to_use_negative:
- When low read depth is the limiting factor — filtering to high-quality reads removes
  observations and can interact with the separate minimum read depth for inclusion
  parameter
- When basecalled fastq lacks per-base quality scores or event data (e.g. non-Albacore
  basecallers that do not save event data), since average quality cannot be computed
  or eventalign will fail
- When evaluating accuracy at contexts dissimilar to training examples, where mCaller
  accuracy is dominated by training bias rather than read quality
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: mCaller
  role: Runs classification with the -q/--qual_thresh option that applies the read
    quality threshold, producing filtered per-read m6A calls
  repo: github.com/al-mcintyre/mCaller
- name: biopython
  role: Parses the fastq to compute each read's average base quality before/while
    filtering
- name: nanopolish
  role: Generates the eventalign TSV input with scaled event values that mCaller classifies
    under the quality filter
  repo: https://github.com/jts/nanopolish
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-08289-9
    title: Single-molecule sequencing detection of N6-methyladenine in microbial reference
      materials
  source_content_hash: sha256:f62565839e47b74d038ea884018c5638d15d6e6f31daa6b47bd6c665c62cb5b6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - biopython
  - mCaller
  - nanopolish
  - numpy
  - pandas
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# nanopore-read-quality-filtering

## Summary

Restricts mCaller m6A classification to nanopore reads whose average base quality exceeds a cutoff via the -q/qual_thresh option, improving per-position accuracy by excluding noisy reads. For R9 data in this study, the 'NN_hq' variant classified only reads with average base quality > 9.

## When to use

Use when running mCaller on ONT eventalign data and read-level noise from low-quality basecalls is degrading per-position methylation accuracy, and you want to restrict classification to reads above an average base-quality threshold.

## When NOT to use

- When low read depth is the limiting factor — filtering to high-quality reads removes observations and can interact with the separate minimum read depth for inclusion parameter
- When basecalled fastq lacks per-base quality scores or event data (e.g. non-Albacore basecallers that do not save event data), since average quality cannot be computed or eventalign will fail
- When evaluating accuracy at contexts dissimilar to training examples, where mCaller accuracy is dominated by training bias rather than read quality

## Inputs

- nanopolish eventalign TSV (e.g. testdata/masonread1.eventalign.tsv) with scaled event values per read
- fastq of basecalled ONT reads (e.g. testdata/masonread1.fastq) used to compute average base quality
- positions file with chromosome, position, strand, and label (e.g. testdata/test_positions_m6A.txt) with PacBio-derived methylation labels
- reference fasta (e.g. testdata/pb_ecoli_polished_assembly.fasta)
- trained model pickle (e.g. r95_twobase_model_NN_6_m6A.pkl)

## Outputs

- quality-filtered per-read m6A predictions in the eventalign diffs file (e.g. testdata/masonread1.eventalign.diffs.6) with chromosome, read name, genomic position, k-mer context, features, strand, label, and methylation probability columns
- per-position accuracy of filtered calls versus PacBio-derived labels (expected ~84.2% for R9 E. coli data)

## How to apply

Compute each read's average base quality from the fastq (e.g. with Biopython over testdata/masonread1.fastq), then re-run mCaller.py with the same arguments as the unfiltered baseline (-p positions file, -r reference fasta, -e eventalign tsv, -d model pkl, -f fastq, -b A) plus -q set to the R9 read-quality cutoff so only reads exceeding the threshold are classified. The article's 'NN_hq' model variant used average base quality > 9 for R9 data. Score per-position accuracy of the filtered calls against PacBio-derived labels in the positions file; for R9 E. coli data, filtered accuracy reproduces the reported ~84.2%. Write the quality-filtered per-read predictions to the eventalign diffs output (e.g. testdata/masonread1.eventalign.diffs.6) retaining chromosome, read name, genomic position, k-mer context, features, strand, label, and methylation probability.

## Related tools

- **mCaller** (Runs classification with the -q/--qual_thresh option that applies the read quality threshold, producing filtered per-read m6A calls) — github.com/al-mcintyre/mCaller
- **biopython** (Parses the fastq to compute each read's average base quality before/while filtering)
- **nanopolish** (Generates the eventalign TSV input with scaled event values that mCaller classifies under the quality filter) — https://github.com/jts/nanopolish

## Examples

```
mCaller.py -p testdata/test_positions_m6A.txt -r testdata/pb_ecoli_polished_assembly.fasta -e testdata/masonread1.eventalign.tsv -d r95_twobase_model_NN_6_m6A.pkl -f testdata/masonread1.fastq -b A -q 9
```

## Evaluation signals

- Filtered per-position accuracy on R9 E. coli positions reproduces the reported ~84.2% ('NN_hq' variant)
- Every read in the filtered diffs output has average base quality > the applied qual_thresh (e.g. > 9 for R9 data)
- Number of per-read observations in the filtered diffs output is strictly smaller than the unfiltered run (no -q) for the same positions file
- Output diffs file retains all expected columns: chromosome, read name, genomic position, k-mer context, features, strand, label, methylation probability
- mCaller runs identically apart from the -q flag, so per-read predictions for reads above the threshold should match the unfiltered baseline run

## Limitations

- The default qual_thresh is 'none'; without -q, reads at all quality levels are classified, which lowers accuracy
- Training biases can decrease accuracy at contexts with lower similarity to the training examples, independent of read quality
- mCaller showed notable failure at one S. aureus motif, so quality filtering alone does not guarantee correct calls at all motifs
- Restricting to high-quality reads trades accuracy for reduced coverage at positions where only low-quality reads exist

## Evidence

- [methods] NN_hq restricted classification to only high quality reads, defined as reads with average base quality > 9, in place of using reads at all quality levels.: [evidence span withheld — anchor status: not_found]
- [readme] qual_thresh option in mCaller README: "-q, --qual_thresh
 quality threshold for reads (default none)"
- [other] high-quality-read accuracy reproduces reported value: [evidence span withheld — anchor status: not_found]
- [discussion] filtering rationale: exclude low-quality reads to improve accuracy: "For mCaller, training biases could decrease accuracy at contexts with lower similarity to the training examples."
