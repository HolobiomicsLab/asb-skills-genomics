---
name: cross-platform-sequence-comparison
description: Use when comparing consensus sequences generated on one platform (e.g.
  Nanopore MinION long rDNA amplicons) against reference consensus sequences from
  another (e.g. Illumina short reads) to quantify cross-platform error rates.
when_to_use_negative:
- Comparing qualitative species richness or quantitative abundance recovery in bulk
  mock communities rather than consensus accuracy — the article notes long rDNA amplicon
  sequencing 'recover[s] highly skewed diversity estimates' and needs further research
  before bulk-sample use
- Raw-read error assessment — this skill operates on polished consensus sequences,
  not uncorrected MinION reads
- Input reads have not yet been demultiplexed and trimmed of barcodes/primers with
  minibar.py — binning and trimming (-T, -F) must precede consensus assembly
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: MiniBar
  role: Demultiplexes dual indexed MinION reads and trims barcodes/primers to produce
    per-sample FASTA files prior to consensus assembly
  repo: https://github.com/calacademy-research/minibar
- name: Edlib
  role: Python/C++ edit-distance alignment library required by minibar.py for barcode
    and primer matching
  repo: https://github.com/Martinsos/edlib
- name: Allele Wrangler
  role: Assembles per-specimen rDNA consensus sequences from trimmed Nanopore reads
    with a 30-reads-per-position minimum coverage threshold
- name: Racon
  role: Polishes Nanopore consensus sequences to reduce residual raw-read errors before
    cross-platform distance computation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giz006
    title: Nanopore sequencing of long ribosomal DNA amplicons enables portable and
      simple biodiversity assessments with high phylogenetic resolution across broad
      taxonomic scale
  source_content_hash: sha256:14c9d52de1635d4680e739e225ed67e5c4f377eba02e7d9206ef42b16c17e46f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/calacademy-research/minibar
  tools:
  - Allele Wrangler
  - Edlib
  - MiniBar
  - Python 2.7
  - Racon
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# cross-platform-sequence-comparison

## Summary

Quantifies Nanopore long-amplicon rDNA consensus accuracy by computing pairwise alignment distances between MinION-derived consensus sequences and Illumina-based consensus references, as done for the Peperomia test set at MiniBar index edit distances 2, 3, and 4.

## When to use

Use when comparing consensus sequences generated on one platform (e.g. Nanopore MinION long rDNA amplicons) against reference consensus sequences from another (e.g. Illumina short reads) to quantify cross-platform error rates.

## When NOT to use

- Comparing qualitative species richness or quantitative abundance recovery in bulk mock communities rather than consensus accuracy — the article notes long rDNA amplicon sequencing 'recover[s] highly skewed diversity estimates' and needs further research before bulk-sample use
- Raw-read error assessment — this skill operates on polished consensus sequences, not uncorrected MinION reads
- Input reads have not yet been demultiplexed and trimmed of barcodes/primers with minibar.py — binning and trimming (-T, -F) must precede consensus assembly

## Inputs

- Nanopore MinION FASTQ/FASTA reads of long dual-indexed rDNA amplicons
- Barcode/index combination file (e.g. IndexCombinationPeperomonia.txt / MinIndexCombinationPeperomonia.txt)
- Per-specimen Nanopore rDNA consensus sequences (post Allele Wrangler assembly and Racon polishing)
- Illumina-based consensus reference sequences for the same specimens (e.g. Peperomia consensus sequences)

## Outputs

- Per-sample demultiplexed FASTA files (e.g. Peperomia_Jun_38/39/40, unk)
- Per-specimen pairwise genetic distances between Nanopore and Illumina consensus sequences (percent)
- Summary table of average distances per MiniBar index edit distance (0.165%, 0.187%, 2.072%)

## How to apply

First demultiplex MinION reads with minibar.py using the dual-index barcode file (e.g. IndexCombinationPeperomonia.txt) with the -T trim and -F binning options (-P prefix) so barcodes and primers are removed and records are written to per-sample FASTA files. Assemble a per-specimen rDNA consensus with Allele Wrangler, enforcing a minimum coverage threshold of 30 reads per position, then polish each Nanopore consensus with Racon error correction. Finally, align each Nanopore rDNA consensus against the corresponding Illumina-based consensus (e.g. the Peperomia consensus sequences) using pairwise sequence alignment and tabulate per-specimen distances and averages. In the source study, average distances of 0.165% (edit distance 2), 0.187% (edit distance 3), and 2.072% (edit distance 4) demonstrated that consensus accuracy degrades sharply at high barcode edit distances.

## Related tools

- **MiniBar** (Demultiplexes dual indexed MinION reads and trims barcodes/primers to produce per-sample FASTA files prior to consensus assembly) — https://github.com/calacademy-research/minibar
- **Edlib** (Python/C++ edit-distance alignment library required by minibar.py for barcode and primer matching) — https://github.com/Martinsos/edlib
- **Allele Wrangler** (Assembles per-specimen rDNA consensus sequences from trimmed Nanopore reads with a 30-reads-per-position minimum coverage threshold)
- **Racon** (Polishes Nanopore consensus sequences to reduce residual raw-read errors before cross-platform distance computation)

## Examples

```
./minibar.py -T -F -P Peperomia_ IndexCombinationPeperomonia.txt PeperomiaTestSet.fasta
```

## Evaluation signals

- Computed average pairwise Nanopore-vs-Illumina distances reproduce the reported 0.165%, 0.187%, and 2.072% within rounding tolerance
- Every specimen consensus meets the minimum coverage threshold of 30 reads per position before distance computation
- minibar.py run summary line reports IDs assigned and Mult_IDs 0 (e.g. '750 seqs: H 750 HH 679 Hh 62 hh 0 IDs 741 Mult_IDs 0')
- Matched sample IDs and barcode/primer edit distances appear in record headers (e.g. 'H-(0,6),H+(0,2) Jun_40')

## Limitations

- Consensus accuracy depends strongly on MiniBar index edit distance: average distance to the Illumina consensus rises from ~0.165-0.187% (edit distances 2-3) to 2.072% at edit distance 4
- Records with a barcode hit but no primer match within the maximum 11 errors (counted as 'Hh') are lower-confidence assignments
- This cross-platform accuracy comparison does not extend to bulk-sample abundance analysis, which the article flags as biased and requiring further research

## Evidence

- [methods] Compute pairwise genetic distances between each Nanopore rDNA consensus and the corresponding Illumina-based Peperomia consensus sequences using pairwise sequence alignment.: [evidence span withheld — anchor status: not_found]
- [results] Consensus accuracy significantly increased for MiniBar index edit distances of two (0.165% average distance to Illumina-based consensus) and three (0.187% average distance): [evidence span withheld — anchor status: not_found]
- [intro] We compare the qualitative (species richness) and quantitative (species abundance) recovery of taxa in simple mock communities by long-amplicon sequencing with that based on short read Illumina: "We compare the qualitative (species richness) and quantitative (species abundance) recovery of taxa in simple mock communities by long-amplicon sequencing with that based on short read Illumina"
- [readme] Identify MinION sequence by dual barcode indexes and primers.: "Identify MinION sequence by dual barcode indexes and primers."
- [discussion] Further research into the sources of the observed bias is required before long rDNA amplicon sequencing can be utilized as a reliable resource for the analysis of bulk samples.: "Further research into the sources of the observed bias is required before long rDNA amplicon sequencing can be utilized as a reliable resource for the analysis of bulk samples."
