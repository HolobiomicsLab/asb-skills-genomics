---
name: nanopore-barcode-kit-generation
description: Use when designing or expanding a DNA barcode kit for nanopore sequencing
  at a fixed barcode length (e.g., 24 bp) and error tolerance (e.g., 2 edit errors),
  or when comparing a generated kit's demultiplexing accuracy to official ONT kits.
when_to_use_negative:
- 'PacBio sequencing data: the article states ''no tests on PacBio data have been
  performed yet'', so kit designs are only validated on ONT data.'
- The goal is demultiplexing existing official ONT kit reads without generating a
  new kit — use the demultiplexing mode alone rather than this generation-and-benchmark
  skill.
- 'Barcodes of unequal length: the NP-complete formulation and probability-divergence
  design assume all candidate sequences have the same length.'
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: PRO
  role: Generates the barcode kit via farthest-point sampling under probability divergence
    (pro_FPS_select.py) and demultiplexes simulated reads
  repo: https://github.com/renztr/PRO.git
- name: Guppy
  role: Official ONT demultiplexing baseline used to benchmark the PRO-generated kit
    against official kits
- name: edlib
  role: Computes semiglobal-alignment edit distances to remove preselected candidates
    within edit distance 1 of flanking sequences
- name: ONT-sequencing-data-library-preparation-pipeline
  role: Simulates library preparation, ligating PRO and official ONT barcode kits
    onto raw DNA sequences with custom flanking sequences
  repo: https://github.com/JustLeeee/ONT-sequencing-data-library-preparation-pipeline.git
- name: deepsimulator 1.5
  role: Simulates the ONT sequencing process to obtain FASTQ reads after library preparation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.fmre.2024.04.014
    title: Generating barcodes for nanopore sequencing data with PRO
  source_content_hash: sha256:213b88174d5cafb29ba804a823cef41fbb57265b27c83964fa6516cd5f8e7134
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/rztongr/PRO
  repo_url: https://github.com/renztr/PRO.git
  tools:
  - Anaconda3
  - Guppy
  - ONT-sequencing-data-library-preparation-pipeline
  - PRO
  - Python
  - deepsimulator 1.5
  - edlib
  - tqdm
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# nanopore-barcode-kit-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generate error-tolerant DNA barcode kits for ONT nanopore sequencing using PRO's farthest-point sampling under probability divergence, then benchmark demultiplexing accuracy (with PRO and Guppy 6.5.7) against official ONT kits via simulated library preparation and sequencing.

## When to use

Use when designing or expanding a DNA barcode kit for nanopore sequencing at a fixed barcode length (e.g., 24 bp) and error tolerance (e.g., 2 edit errors), or when comparing a generated kit's demultiplexing accuracy to official ONT kits.

## When NOT to use

- PacBio sequencing data: the article states 'no tests on PacBio data have been performed yet', so kit designs are only validated on ONT data.
- The goal is demultiplexing existing official ONT kit reads without generating a new kit — use the demultiplexing mode alone rather than this generation-and-benchmark skill.
- Barcodes of unequal length: the NP-complete formulation and probability-divergence design assume all candidate sequences have the same length.

## Inputs

- Flanking-sequence FASTQ file used in library preparation
- User-specified candidate DNA sequences (txt) or full 4^n k-mer space for barcode length n
- Raw DNA reference reads (e.g., 104 E. coli whole-genome shotgun sequences and Enterobacteria phage lambda genome)
- Official ONT barcode kits and Guppy config files (barcode_arrs_nb12.cfg, barcode_arrs_16s.cfg, barcode_arrs_pcr96.cfg)
- PRO-generated barcode set (select_barcode.txt)

## Outputs

- select_barcode.txt with the generated barcode kit (e.g., 96 barcodes of 24 bp)
- pre_processing_barcodes.txt and pre_processing_barcodes_number.txt (preprocessed candidates and hash integers)
- Simulated ONT FASTQ reads per kit from deepsimulator 1.5
- demux_accuracy_comparison.csv comparing PRO vs Guppy 6.5.7 demultiplexing accuracies

## How to apply

Preprocess the 4^n candidate space (e.g., 4^24 24-mers) by hashing each candidate into a 64-bit integer (N(G)=0, N(A)=1, N(T)=2, N(C)=3), sorting in descending order, and randomly preselecting from equal bins; drop sequences whose semiglobal-alignment edit distances to both flanking sequences are <= 1 using edlib. Run PRO's pro_FPS_select.py with -l 24, -th 2 (barcodes tolerating two edit errors), -a 0.88 (sequencing error rate), -f flanking-sequence fastq, and the preselected size (-q) so adapted farthest-point sampling under probability divergence outputs the desired barcode count into select_barcode.txt. Simulate library preparation and ONT sequencing by ligating the PRO kit (and official kits such as B1 EXP-NBD104, B2 SQK-16S024, B3 EXP-PBC096) onto E. coli WGS and lambda phage reads with the JustLeeee library-preparation pipeline and deepsimulator 1.5. Demultiplex simulated reads with PRO (probability divergence, narrowed flanking alignment ranges sequence'[35:77] and sequence'[35+startf:120]) and Guppy 6.5.7 (official configs barcode_arrs_nb12.cfg, barcode_arrs_16s.cfg, barcode_arrs_pcr96.cfg), then compile accuracies for comparison. Rationale: the underlying barcode-generation problem is NP-complete, so farthest-point sampling with probability divergence gives a tractable, sufficiently distinct barcode set.

## Related tools

- **PRO** (Generates the barcode kit via farthest-point sampling under probability divergence (pro_FPS_select.py) and demultiplexes simulated reads) — https://github.com/renztr/PRO.git
- **Guppy** (Official ONT demultiplexing baseline used to benchmark the PRO-generated kit against official kits)
- **edlib** (Computes semiglobal-alignment edit distances to remove preselected candidates within edit distance 1 of flanking sequences)
- **ONT-sequencing-data-library-preparation-pipeline** (Simulates library preparation, ligating PRO and official ONT barcode kits onto raw DNA sequences with custom flanking sequences) — https://github.com/JustLeeee/ONT-sequencing-data-library-preparation-pipeline.git
- **deepsimulator 1.5** (Simulates the ONT sequencing process to obtain FASTQ reads after library preparation)

## Examples

```
python pro_FPS_select.py -l 24 -th 2 -a 0.88 -f flanking.fastq -q 100000 -o output_dir
```

## Evaluation signals

- select_barcode.txt contains exactly the requested number of barcodes (e.g., 96 for the official ONT-size comparison) at the requested length (e.g., 24 bp).
- All pairwise probability-divergence distances within the kit meet the threshold t set for tolerating two edit errors (-th 2).
- Generated barcodes have semiglobal-alignment edit distances > 1 from both flanking sequences.
- Demultiplexing comparison reproduces the reported pattern: PRO averages ~4.34% higher accuracy than Guppy (up to 4.73% on B4) on the same-size 96-barcode 24 bp kit.
- Official ONT kits B1 (EXP-NBD104), B2 (SQK-16S024), and B3 (EXP-PBC096) show high accuracies with Guppy (100.00%, 99.71%, 99.66%) while PRO underperforms on these kits.

## Limitations

- PRO underperforms Guppy on official kits B1, B2, and B3 because the flanking sequences are similar to the barcodes and ONT's design strategy does not ensure large probability divergence between barcodes.
- No tests on PacBio data have been performed, though the approach could in principle apply to PacBio sequencing.
- Finding the optimal largest barcode set in a same-length sequence space is NP-complete, so PRO's farthest-point sampling provides a heuristic, not the optimal, barcode set.

## Evidence
