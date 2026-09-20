---
name: edit-distance-tolerance-parameterization
description: Use when demultiplexing dual-indexed Oxford Nanopore reads with MiniBar
  and you need to set or vary the barcode (-e) and primer (-E) edit-distance tolerances
  to balance read yield against misassignment.
when_to_use_negative:
- Reads are not dual-indexed Nanopore amplicons (e.g. single-index Illumina data)
  — MiniBar's dual barcode-plus-primer matching at both read ends does not apply
- The goal is consensus calling or downstream phylogenetic analysis of already-binned
  reads rather than tuning demultiplexing stringency
- You need exact barcode matching or mismatch-tolerant schemes other than Levenshtein/edit
  distance, which is what the Edlib-backed matcher implements
edam_operation: http://edamontology.org/operation_0231
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3168
tools:
- name: MiniBar
  role: Demultiplexer whose -e (barcode edit distance, default 4) and -E (primer edit
    distance, default 11) flags are the parameters being tuned; runs with search length
    80 and method 3
  repo: https://github.com/calacademy-research/minibar
- name: Edlib
  role: Python library providing the fast Levenshtein edit-distance computation underlying
    MiniBar's barcode and primer matching
  repo: https://github.com/Martinsos/edlib
- name: Python 2.7
  role: Runtime for minibar.py (compatible with Python 3) in which the -e sweep and
    per-read header parsing are executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giz006
    title: Nanopore sequencing of long ribosomal DNA amplicons enables portable and
      simple biodiversity assessments with high phylogenetic resolution across broad
      taxonomic scale
  source_content_hash: sha256:56fa151cb6a1fd12a12b97280e75c02116bbcc2ac825796412088bea217da280
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/calacademy-research/minibar
  tools:
  - Edlib
  - MiniBar
  - Python 2.7
  edam_operation: http://edamontology.org/operation_0231
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# edit-distance-tolerance-parameterization

## Summary

Tunes the barcode and primer edit-distance thresholds (MiniBar's -e and -E flags) when demultiplexing dual-indexed Nanopore rDNA amplicon reads, trading read yield against misassignment. The article shows per-specimen read recovery rising steeply with the barcode edit distance (mean 355/647/1,051 reads at -e 2/3/4 with -E 11).

## When to use

Use when demultiplexing dual-indexed Oxford Nanopore reads with MiniBar and you need to set or vary the barcode (-e) and primer (-E) edit-distance tolerances to balance read yield against misassignment.

## When NOT to use

- Reads are not dual-indexed Nanopore amplicons (e.g. single-index Illumina data) — MiniBar's dual barcode-plus-primer matching at both read ends does not apply
- The goal is consensus calling or downstream phylogenetic analysis of already-binned reads rather than tuning demultiplexing stringency
- You need exact barcode matching or mismatch-tolerant schemes other than Levenshtein/edit distance, which is what the Edlib-backed matcher implements

## Inputs

- Dual-indexed Nanopore FASTQ or FASTA read file (e.g. PeperomiaTestSet.fasta), plain or gzipped
- MiniBar sample barcode combination file mapping sample IDs to fwd/rev index and primer sequences (e.g. IndexCombinationPeperomonia.txt / MinIndexCombinationPeperomonia.txt)
- Python 2.7 (or 3) environment with the Edlib module installed (pip install edlib)

## Outputs

- Annotated output file with sample ID and match-hit info (e.g. H-(0,6),H+(0,2) Jun_40) appended to each record header comment (e.g. PeperomiaTestSet_SampleIDs.fa)
- Per-run summary line such as '750 seqs: H 750 HH 679 Hh 62 hh 0 IDs 741 Mult_IDs 0'
- Per-specimen read counts at each tested index edit distance (2, 3, 4), assemblable into a tab-separated table with mean reads per specimen

## How to apply

Run minibar.py against a barcode-combination file (e.g. IndexCombinationPeperomonia.txt) and a FASTA/FASTQ read file, keeping the default search length (-l 80) and search method (-M 3) fixed while varying the barcode edit distance via -e across candidate values (e.g. 2, 3, 4); the primer edit distance defaults to -E 11. MiniBar matches barcodes and primers at both read ends using Edlib edit distance, counts perfect-perfect (HH), perfect-imperfect (Hh) and imperfect-imperfect (hh) hits, and appends the sample ID to each record header; records with no unique match become 'unk' or 'Multiple_Matches'. Lower -e values give conservative assignment (fewer, higher-confidence reads); higher values recover more reads per specimen but risk index-hopping style misassignment, so compare the summary line and per-specimen tallies across settings. Read counts increase significantly with larger edit distance, so report the sensitivity of assigned reads per specimen to -e rather than a single run. Verify by parsing the per-run summary line (seqs, HH/Hh/hh, IDs, Mult_IDs) and confirming nonzero IDs with Mult_IDs and unk reads recorded separately.

## Related tools

- **MiniBar** (Demultiplexer whose -e (barcode edit distance, default 4) and -E (primer edit distance, default 11) flags are the parameters being tuned; runs with search length 80 and method 3) — https://github.com/calacademy-research/minibar
- **Edlib** (Python library providing the fast Levenshtein edit-distance computation underlying MiniBar's barcode and primer matching) — https://github.com/Martinsos/edlib
- **Python 2.7** (Runtime for minibar.py (compatible with Python 3) in which the -e sweep and per-read header parsing are executed)

## Examples

```
minibar.py -e 2 IndexCombinationPeperomonia.txt PeperomiaTestSet.fasta >PeperomiaTestSet_SampleIDs.fa
```

## Evaluation signals

- The per-run summary line reports nonzero ID counts with Mult_IDs and unassigned (unk) reads tallied separately
- Mean reads-per-specimen at index edit distances 2, 3, and 4 with primer edit distance 11 reproduce the reported 355 / 647 / 1,051 pattern with counts increasing significantly at larger edit distances
- Annotated record headers end in a single sample ID with hit info before it (e.g. 'H-(0,6),H+(0,2) Jun_40'); headers lacking a unique match are classified unk or Multiple_Matches, not silently dropped
- HH + Hh + hh categories account for the seqs total in the summary line for each -e setting
- Changing -e while holding -E 11, -l 80 and -M 3 constant changes only the assignment stringency, verifiable by comparing summary lines across runs

## Limitations

- Higher barcode edit distances recover more reads per specimen but admit more ambiguous or incorrect sample assignments; the trade-off is not validated against a ground truth in the reported workflow
- Records with no or multiple sample matches (unk / Multiple_Matches) are rejected and reduce yield, especially at stricter settings
- The primer match can fail even when the barcode matches (lowercase 'h' with -1 in the parenthesis list, e.g. h-(3,-1)), limiting confident assignment; the 27-base primers are matched within the maximum 11 errors allowed
- Reported means are from the Peperomia dual-indexed rDNA test set; the optimal -e may differ for other barcode sets or amplicons

## Evidence

- [readme] -e <int> barcode edit distance value, overrides -p (4): "-e <int> barcode edit distance value, overrides -p (4)"
- [readme] -E <int> primer edit distance value (11): "-E <int> primer edit distance value (11)"
- [readme] 750 seqs: H 750 HH 679 Hh 62 hh 0 IDs 741 Mult_IDs 0 (0.1245s): "750 seqs: H 750 HH 679 Hh 62 hh 0 IDs 741 Mult_IDs 0 (0.1245s)"
- [readme] The -l and -M defaults anchor the sweep: "-l <int> length to search for index and primer at start and end of sequence (80)"
- [readme] The summary line reports IDs and Mult_IDs counts per run: "Sample ID is placed at end of header comment with match hit info before it."
- [methods] Article: identify reads by dual barcode indexes and primers: "Identify MinION sequence by dual barcode indexes and primers."
