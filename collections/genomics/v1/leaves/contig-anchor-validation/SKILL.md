---
name: contig-anchor-validation
description: Use when running DBG2OLC Step2 (overlap and layout) on PacBio/ONT long
  reads against short-read Contigs.txt and you must decide which contigs anchor long
  reads, or when chimera removal is needed at >10x long-read coverage.
when_to_use_negative:
- Long-read coverage is below 10x — the README only suggests RemoveChimera 1 if you
  have >10x coverage
- Input contigs come from a short-read assembler after repeat resolving, gap closing,
  or scaffolding — the README warns such heuristics introduce errors that propagate
  to poor final results
- The goal is the non-hybrid NGS-only assembly mode, which does not use long-read/contig
  anchoring
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: DBG2OLC
  role: 'Performs the overlap-and-layout step: compresses long reads to k=17 contig-ID
    space, applies KmerCovTh/AdaptiveTh anchor thresholds and MinOverlap overlap scores,
    and removes chimeric reads with RemoveChimera 1'
  repo: https://github.com/yechengxi/DBG2OLC
- name: SparseAssembler
  role: Produces the raw DBG Contigs.txt that are validated as anchors to the long
    reads
  repo: https://github.com/yechengxi/SparseAssembler
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/srep31900
    title: 'DBG2OLC: Efficient Assembly of Large Genomes Using Long Erroneous Reads
      of the Third Generation Sequencing Technologies'
  source_content_hash: sha256:da9a0548b4b674198a93fbcf9b1655c124b336085ae5a6f7c22adf9bab9611c2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yechengxi/DBG2OLC
  tools:
  - DBG2OLC
  - SparseAssembler
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# contig-anchor-validation

## Summary

Validates which SparseAssembler/DBG contigs may serve as anchors to compressed long reads during DBG2OLC's overlap-and-layout step, using k-mer matching thresholds (AdaptiveTh, KmerCovTh) computed on the k=17 compressed-read representation, and concurrently removes chimeric reads via multiple sequence alignment of compressed reads.

## When to use

Use when running DBG2OLC Step2 (overlap and layout) on PacBio/ONT long reads against short-read Contigs.txt and you must decide which contigs anchor long reads, or when chimera removal is needed at >10x long-read coverage.

## When NOT to use

- Long-read coverage is below 10x — the README only suggests RemoveChimera 1 if you have >10x coverage
- Input contigs come from a short-read assembler after repeat resolving, gap closing, or scaffolding — the README warns such heuristics introduce errors that propagate to poor final results
- The goal is the non-hybrid NGS-only assembly mode, which does not use long-read/contig anchoring

## Inputs

- Contigs.txt — raw DBG contigs in FASTA from SparseAssembler Step1 (no scaffolding/gap closing)
- Long-read FASTA (e.g. Pacbio_20x.fasta) to be compressed into contig-ID space
- Overlap scores / candidate overlaps between compressed long reads from the k=17 matching step

## Outputs

- Validated contig-anchor assignments embedded in the DBG2OLC layout (backbone_raw.fasta)
- Chimera-cleaned compressed-read set feeding the layout step
- DBG2OLC log listing removed chimeric reads and rejected contig anchors

## How to apply

Compress each long read into its constituent DBG contig IDs using a k=17 k-mer representation, then for every contig-long-read pair compute M, the number of matched k-mers. Reject a contig as an anchor when M < KmerCovTh (fixed threshold, e.g. KmerCovTh 2) or M < AdaptiveTh * Contig_Length (adaptive threshold, e.g. AdaptiveTh 0.0001 for 20x data up to 0.01-0.02 for 50x-100x data). Simultaneously compute the MSA-based overlap score between pairs of compressed reads and drop pairs scoring below MinOverlap (e.g. 20 for 10x-20x, 50-150 for 50x-100x); enable RemoveChimera 1 (suggested at >10x coverage) so chimeric reads and spurious contig identifiers are cleaned via this multiple alignment, raising ChimeraTh/ContigTh to 2 only at ~100x coverage. Rationale: base-level errors are skipped and structural errors (chimeras, false anchors) are what must be detected and corrected, per the paper's second design principle. Tune KmerCovTh 2-5, MinOverlap 10-30, AdaptiveTh 0.001-0.01 for 10x/20x PacBio data before proceeding to consensus, checking the N50 of backbone_raw.fasta as the gate.

## Related tools

- **DBG2OLC** (Performs the overlap-and-layout step: compresses long reads to k=17 contig-ID space, applies KmerCovTh/AdaptiveTh anchor thresholds and MinOverlap overlap scores, and removes chimeric reads with RemoveChimera 1) — https://github.com/yechengxi/DBG2OLC
- **SparseAssembler** (Produces the raw DBG Contigs.txt that are validated as anchors to the long reads) — https://github.com/yechengxi/SparseAssembler

## Examples

```
./DBG2OLC k 17 AdaptiveTh 0.0001 KmerCovTh 2 MinOverlap 20 RemoveChimera 1 Contigs Contigs.txt f ../Pacbio_data/Pacbio_20x.fasta
```

## Evaluation signals

- Every contig used as an anchor satisfies M >= KmerCovTh and M >= AdaptiveTh * Contig_Length matched k-mers under the k=17 representation
- All read pairs retained in the layout have overlap scores >= MinOverlap (20 for 10x/20x PacBio data)
- Chimeric reads flagged by the compressed-read MSA are absent from the layout used to build backbone_raw.fasta
- N50 of backbone_raw.fasta is acceptable (e.g. ~583 kbp on 20x S.cer w303 test data) before proceeding to consensus — the README says otherwise keep tuning
- Raising thresholds per the suggested tuning ranges (KmerCovTh 2-5, MinOverlap 10-30 at 10x/20x) improves rather than collapses the layout

## Limitations

- RemoveChimera is only recommended when coverage exceeds 10x; at lower coverage chimeras may not be reliably detected
- At ~100x coverage default ChimeraTh/ContigTh of 1 must be raised to 2, otherwise problematic reads and false contig anchors may survive the MSA filtering
- Contigs from assemblers that used repeat-resolving heuristics (gap closing, scaffolding) degrade the final assembly, so validation assumes raw DBG contigs only

## Evidence

- [readme] AdaptiveTh: adaptive k-mer matching threshold. If M < AdaptiveTh* Contig_Length, this contig cannot be used as an anchor to the long read.: "AdaptiveTh: adaptive k-mer matching threshold. If M < AdaptiveTh* Contig_Length, this contig cannot be used as an anchor to the long read."
- [methods] KmerCovTh: fixed k-mer matching threshold. If M < KmerCovTh, this contig cannot be used as an anchor to the long read.: "KmerCovTh: fixed k-mer matching threshold. If M < KmerCovTh, this contig cannot be used as an anchor to the long read."
- [readme] RemoveChimera: remove chimeric reads in the dataset, suggest 1 if you have >10x coverage.: "RemoveChimera: remove chimeric reads in the dataset, suggest 1 if you have >10x coverage."
