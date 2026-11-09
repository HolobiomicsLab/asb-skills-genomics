---
name: compressed-read-kmer-alignment
description: Use when long (PacBio/3GS) reads have been compressed against SparseAssembler
  contigs and chimeric reads or false contig anchors must be detected before layout,
  especially at >10x long-read coverage.
when_to_use_negative:
- Long-read coverage is below 10x — the authors only suggest RemoveChimera 1 when
  you have >10x coverage
- The pipeline is still at Step1 (short-read contig construction with SparseAssembler)
  and no compressed reads exist yet
- Reads are raw uncompressed 3GS reads rather than DBG2OLC compressed reads, since
  overlap scores are computed on compressed representations
edam_operation: http://edamontology.org/operation_3180
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: DBG2OLC
  role: Performs the k-mer-based compressed-read alignment, overlap-score computation,
    and chimera removal via RemoveChimera, AdaptiveTh, KmerCovTh, and MinOverlap parameters
  repo: https://github.com/yechengxi/DBG2OLC
- name: SparseAssembler
  role: Produces the accurate DBG contigs (Contigs.txt) used to compress the long
    reads prior to chimera cleaning
  repo: https://github.com/yechengxi/SparseAssembler
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/srep31900
    title: 'DBG2OLC: Efficient Assembly of Large Genomes Using Long Erroneous Reads
      of the Third Generation Sequencing Technologies'
  source_content_hash: sha256:f8a932c6016e0a95fd2c2bb30784b06afdd342678e31511f1b98c81f2f6d4680
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yechengxi/DBG2OLC
  tools:
  - DBG2OLC
  - SparseAssembler
  edam_operation: http://edamontology.org/operation_3180
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# compressed-read-kmer-alignment

## Summary

Reconstructs DBG2OLC's chimera-cleaning stage, in which each compressed long read is aligned against candidate compressed reads via k-mer matching (k 17) to compute overlap scores, flagging chimeric reads and spurious contig identifiers for removal.

## When to use

Use when long (PacBio/3GS) reads have been compressed against SparseAssembler contigs and chimeric reads or false contig anchors must be detected before layout, especially at >10x long-read coverage.

## When NOT to use

- Long-read coverage is below 10x — the authors only suggest RemoveChimera 1 when you have >10x coverage
- The pipeline is still at Step1 (short-read contig construction with SparseAssembler) and no compressed reads exist yet
- Reads are raw uncompressed 3GS reads rather than DBG2OLC compressed reads, since overlap scores are computed on compressed representations

## Inputs

- Compressed long reads with candidate overlaps from the DBG2OLC overlap step
- SparseAssembler de Bruijn contigs file (Contigs.txt, FASTA)
- Long-read FASTA file (e.g. Pacbio_20x.fasta)

## Outputs

- Cleaned compressed-read set (chimera_cleaned_reads.txt)
- Log of removed chimeric reads and rejected contig anchors (chimera_removal_record.txt)
- DBG2OLC layout/backbone input free of chimeric reads

## How to apply

Load the compressed reads with candidate overlaps produced by the DBG2OLC overlap step together with the SparseAssembler contig file (Contigs.txt) and the long-read FASTA. For each pair of long reads, compute an overlap score by aligning the compressed reads and scoring with matching k-mers using the k-mer-based compressed-read representation (k 17). Apply the anchoring thresholds — AdaptiveTh 0.0001 (adaptive: a contig is rejected if M < AdaptiveTh*Contig_Length) and KmerCovTh 2 (fixed: rejected if M < KmerCovTh) — and the MinOverlap 20 overlap-score cutoff between read pairs. Run DBG2OLC with RemoveChimera 1, which the authors suggest when coverage exceeds 10x, so the multiple-sequence-alignment-based chimera detection flags chimeric reads and spurious contig identifiers. Remove flagged chimeric reads from the compressed-read set; at ~100x coverage, raise ChimeraTh and ContigTh to 2 for more stringent filtering in the multiple alignment.

## Related tools

- **DBG2OLC** (Performs the k-mer-based compressed-read alignment, overlap-score computation, and chimera removal via RemoveChimera, AdaptiveTh, KmerCovTh, and MinOverlap parameters) — https://github.com/yechengxi/DBG2OLC
- **SparseAssembler** (Produces the accurate DBG contigs (Contigs.txt) used to compress the long reads prior to chimera cleaning) — https://github.com/yechengxi/SparseAssembler

## Examples

```
./DBG2OLC k 17 AdaptiveTh 0.0001 KmerCovTh 2 MinOverlap 20 RemoveChimera 1 Contigs Contigs.txt f ../Pacbio_data/Pacbio_20x.fasta
```

## Evaluation signals

- chimera_cleaned_reads.txt exists and is a valid reads file after the run
- Every read flagged in chimera_removal_record.txt is absent from the cleaned compressed-read set
- Overlap scores between read pairs are computed from matching k-mers (k 17) of the compressed reads, consistent with the paper's MinOverlap 20 cutoff
- RemoveChimera was invoked with value 1, matching the README's suggestion for datasets with >10x coverage
- Tuning ranges are respected (e.g. KmerCovTh 2-5 and MinOverlap 10-30 for 10x/20x PacBio data) when parameters are adjusted

## Limitations

- RemoveChimera is only suggested when coverage exceeds 10x; below that the filter may not be appropriate
- At very high coverage (~100x), default ChimeraTh and ContigTh of 1 must be raised to 2, or problematic reads and false contig anchors may survive
- Parameters affect assembly quality and may need fine-tuning; the README advises following the prescribed steps before inventing your own pipeline

## Evidence

- [readme] RemoveChimera: remove chimeric reads in the dataset, suggest 1 if you have >10x coverage.: "RemoveChimera: remove chimeric reads in the dataset, suggest 1 if you have >10x coverage."
- [readme] AdaptiveTh and KmerCovTh anchor rejection criteria: "AdaptiveTh: adaptive k-mer matching threshold. If M < AdaptiveTh* Contig_Length, this contig cannot be used as an anchor to the long read."
- [readme] KmerCovTh fixed threshold: "KmerCovTh: fixed k-mer matching threshold. If M < KmerCovTh, this contig cannot be used as an anchor to the long read."
- [readme] Overlap score definition for compressed reads: "For each pair of long reads, an overlap score is calculated by aligning the compressed reads and score with the matching k-mers."
- [methods] MinOverlap parameter from the article's filter steps: "MinOverlap: minimum overlap score between a pair of long reads."
