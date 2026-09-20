---
name: multiple-sequence-alignment-consensus-calling
description: Use when Nanopore long reads already carry BC, U8 and GE tags (Step 3
  output BAM) and per-UMI consensus cDNA molecules are needed for downstream isoform/matrix
  building in Step 4.b.
when_to_use_negative:
- Input SAM records lack the BC, U8 or GE tags (Step 3 UMI assignment has not been
  run) — ComputeConsensus would have no molecules to group.
- The goal is a gene/isoform matrix directly from annotated long-reads — Step 4.a
  (IsoformMatrixReads) is the correct path, not UMI-guided consensus generation.
- Reads are already high-accuracy consensus molecules — re-running consensus would
  not improve accuracy.
edam_operation: http://edamontology.org/operation_0296
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: Sicelore-2.1.jar (ComputeConsensus)
  role: filters BC/U8/GE-tagged SAM records, applies MAXCLIP/MAPQV0 filters, groups
    by cell barcode and UMI, and builds per-molecule consensus sequences
  repo: https://github.com/ucagenomix/sicelore-2.1
- name: spoa
  role: partial-order alignment (POA) engine used to compute the multiple-alignment
    consensus for UMIs covered by more than 2 reads
  repo: https://github.com/rvaser/spoa
- name: racon
  role: polishes the spoa multiple-alignment consensus
- name: samtools
  role: BAM/SAM record handling in the consensus pipeline
  repo: https://github.com/samtools/samtools
- name: minimap2
  role: supplies the 'de' divergence tag used to select the best-mapping read for
    2-read UMIs and rank reads for consensus
  repo: https://github.com/lh3/minimap2
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-17800-6
    title: High throughput error corrected Nanopore single cell transcriptome sequencing
  source_content_hash: sha256:864e8c65f976e8a448e0293b2cc8374125afc60e09995acf5d9afac4a6a86eed
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ucagenomix/sicelore-2.1
  tools:
  - Java
  - Sicelore-2.1.jar (ComputeConsensus pipeline)
  - Sicelore-2.1.jar (ComputeConsensus)
  - minimap2
  - racon
  - samtools
  - spoa
  edam_operation: http://edamontology.org/operation_0296
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# multiple-sequence-alignment-consensus-calling

## Summary

Generates a single error-corrected consensus cDNA per UMI from Nanopore long-read alignments in the ScNaUmi-seq workflow, using Sicelore's ComputeConsensus with partial-order alignment (spoa) polished by racon to convert noisy Oxford Nanopore reads into high-accuracy molecule sequences.

## When to use

Use when Nanopore long reads already carry BC, U8 and GE tags (Step 3 output BAM) and per-UMI consensus cDNA molecules are needed for downstream isoform/matrix building in Step 4.b.

## When NOT to use

- Input SAM records lack the BC, U8 or GE tags (Step 3 UMI assignment has not been run) — ComputeConsensus would have no molecules to group.
- The goal is a gene/isoform matrix directly from annotated long-reads — Step 4.a (IsoformMatrixReads) is the correct path, not UMI-guided consensus generation.
- Reads are already high-accuracy consensus molecules — re-running consensus would not improve accuracy.

## Inputs

- barcode-, UMI- and gene-tagged BAM file (passedParsedWithSequences.bam from Step 4.b.1, with BC, U8, GE, TE and UE tags)
- Sicelore-2.1.jar (ComputeConsensus pipeline)
- spoa binary (required for step 4.b)
- racon (for consensus polishing)
- temporary working directory (TMPDIR)

## Outputs

- per-molecule consensus cDNA FASTQ (molecules.fastq) with records named '>BC-UMI-RN' (cell barcode, UMI, read count)

## How to apply

Start from the barcode-, UMI- and gene-tagged BAM (passedParsedWithSequences.bam) and keep only SAM records carrying the BC, U8 and GE tags. Filter out potentially chimeric reads whose terminal Soft/Hard clipping exceeds MAXCLIP=150 nt and drop MAPQ=0 records (MAPQV0=false default). Group surviving records per cell barcode (BC) and UMI (U8), defining each molecule's cDNA span from the TSO end (TE tag) to the UMI end (UE tag). For each molecule: a single read passes through as the consensus; 2-read UMIs take the cDNA of the read with the lowest minimap2 'de' divergence tag; UMIs with more than 2 reads get a spoa partial-order-alignment consensus built from the up to MAXREADS=20 lowest-divergence reads, optionally polished with racon. Run ComputeConsensus multi-threaded (e.g. T=20) with a TMPDIR, writing each molecule as a FASTQ record named '>BC-UMI-RN'.

## Related tools

- **Sicelore-2.1.jar (ComputeConsensus)** (filters BC/U8/GE-tagged SAM records, applies MAXCLIP/MAPQV0 filters, groups by cell barcode and UMI, and builds per-molecule consensus sequences) — https://github.com/ucagenomix/sicelore-2.1
- **spoa** (partial-order alignment (POA) engine used to compute the multiple-alignment consensus for UMIs covered by more than 2 reads) — https://github.com/rvaser/spoa
- **racon** (polishes the spoa multiple-alignment consensus)
- **samtools** (BAM/SAM record handling in the consensus pipeline) — https://github.com/samtools/samtools
- **minimap2** (supplies the 'de' divergence tag used to select the best-mapping read for 2-read UMIs and rank reads for consensus) — https://github.com/lh3/minimap2

## Examples

```
java -jar Sicelore-2.1.jar ComputeConsensus I=chr1.bam O=molecules.fastq T=20 TMPDIR=<tmp>
```

## Evaluation signals

- Output FASTQ exists and is non-empty.
- Every FASTQ record header follows the BC-UMI-RN naming scheme (cell barcode, UMI, read count).
- Number of FASTQ records equals the number of molecules surviving the BC/U8/GE-tag, MAXCLIP=150 and MAPQV0 filters.
- Each molecule's consensus cDNA span lies between the TE (TSO end) and UE (UMI end) tags.
- UMIs with 1 read yield the original read sequence; 2-read UMIs yield the sequence of the read with the lowest minimap2 'de' divergence.

## Limitations

- Reads with terminal Soft/Hard clipping greater than 150 nt (potentially chimeric) are discarded, so heavily clipped alignments cannot contribute to consensus.
- MAPQ=0 alignments are dropped by default (MAPQV0=false), removing ambiguously mapped reads.
- Only the top MAXREADS=20 lowest-divergence reads per UMI are used for the spoa consensus; UMIs with 1 or 2 reads receive no true error correction (single read passthrough or best-read selection).
- Consensus generation requires Java >= 13 and spoa installed for step 4.b; chimeric transcript splitting during Step 1 is skipped when no polyA is searched.

## Evidence

- [methods] SAM records with extensive non-matching sequences at either end, hard- or soft clipping are discarded (MAXCLIP parameter, defaults to > 150 nt ): "SAM records with extensive non-matching sequences at either end, hard- or soft clipping are discarded (MAXCLIP parameter, defaults to > 150 nt )"
- [readme] generates molecules consensus sequences (UMI-guided error-correction): [evidence span withheld — anchor status: external_doc]
- [readme] Spoa (SIMD POA) is a c++ implementation of the partial order alignment (POA) algorithm ... which is used to generate consensus sequences: [evidence span withheld — anchor status: external_doc]
- [readme] [spoa](https://github.com/rvaser/spoa) (require for step 4.b): "* [spoa](https://github.com/rvaser/spoa) (require for step 4.b)"
- [abstract] UMI guided error correction allows to generate high accuracy full length sequence information with the 10x Genomics single cell isolation system at high sequencing depths: "UMI guided error correction allows to generate high accuracy full length sequence information with the 10x Genomics single cell isolation system at high sequencing depths"
