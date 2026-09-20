---
name: umi-based-read-grouping
description: Use when you have a barcode-, UMI- and gene-tagged Nanopore BAM (with
  BC, U8, GE tags) and need one consensus cDNA per cell-UMI molecule for downstream
  isoform or variant analysis (Sicelore Step 4.b).
when_to_use_negative:
- Input SAM records lack BC, U8 or GE tags (e.g. unannotated BAM from before Step
  3 UMI assignment) — no molecules can be formed
- You want to build matrices directly from cellBC-UMI annotated long-reads without
  error correction (Sicelore Step 4 option a) rather than from consensus molecules
- UMIs are covered by fewer reads than needed for consensus (only single/double-read
  molecules exist), where consensus building reduces to read selection, not poa/racon
  error correction
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3308
tools:
- name: Sicelore-2.1.jar (ComputeConsensus)
  role: Filters chimeric/MAPQ=0 reads, groups SAM records per BC+U8, and builds per-molecule
    consensus cDNA sequences
  repo: https://github.com/ucagenomix/sicelore-2.1
- name: spoa
  role: Partial order alignment (POA) multiple alignment used to generate the consensus
    sequence for UMIs covered by more than 2 reads (required for Step 4.b)
  repo: https://github.com/rvaser/spoa
- name: racon
  role: Polishes the poa multiple-alignment consensus for multi-read molecules
- name: samtools
  role: BAM/SAM manipulation and indexing of the barcode- and UMI-tagged alignment
    records feeding ComputeConsensus
  repo: https://github.com/samtools/samtools
- name: minimap2
  role: Provides the alignment and 'de' divergence tags used to pick the best-mapping
    read for 2-read UMIs and to rank top MAXREADS=20 reads
  repo: https://github.com/lh3/minimap2
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-17800-6
    title: High throughput error corrected Nanopore single cell transcriptome sequencing
  source_content_hash: sha256:8c8934048aa18a012d009c5c6438aed483b8288a58ce4308ae32c32820a34a63
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# umi-based-read-grouping

## Summary

Groups Nanopore SAM records per cell barcode and UMI and computes a single error-corrected consensus cDNA per molecule using Sicelore's ComputeConsensus (poa multiple alignment polished with racon). This is the UMI-guided error-correction core of ScNaUmi-seq that turns noisy long reads into high-accuracy molecule sequences.

## When to use

Use when you have a barcode-, UMI- and gene-tagged Nanopore BAM (with BC, U8, GE tags) and need one consensus cDNA per cell-UMI molecule for downstream isoform or variant analysis (Sicelore Step 4.b).

## When NOT to use

- Input SAM records lack BC, U8 or GE tags (e.g. unannotated BAM from before Step 3 UMI assignment) — no molecules can be formed
- You want to build matrices directly from cellBC-UMI annotated long-reads without error correction (Sicelore Step 4 option a) rather than from consensus molecules
- UMIs are covered by fewer reads than needed for consensus (only single/double-read molecules exist), where consensus building reduces to read selection, not poa/racon error correction

## Inputs

- Barcode-, UMI- and gene-tagged Nanopore BAM with BC, U8, GE, TE, UE tags (passedParsedWithSequences.bam from Sicelore Step 4.b.1)
- 10x Genomics barcode whitelist (e.g. 3M-february-2018.txt.gz) already used upstream for barcode assignment
- minimap2 'de' divergence tags carried on the aligned records

## Outputs

- Molecule-level FASTQ with one error-corrected consensus cDNA per BC-UMI molecule, headers formatted '>BC-UMI-RN' (molecules.fastq)
- Consensus sequences ready for Step 4.b isoform/gene matrix generation

## How to apply

Load the barcode/UMI/gene-annotated BAM (passedParsedWithSequences.bam from Step 4.b.1) and retain only SAM records carrying BC, U8 and GE tags. Filter out potentially chimeric reads whose terminal soft/hard clipping exceeds 150 nt (MAXCLIP=150) and drop minimap2 MAPQ=0 records (MAPQV0=false default). Group surviving records per cell barcode (BC) and UMI (U8), defining each molecule's cDNA span from the TSO end (TE tag) to the UMI end (UE tag). For 1-read UMIs assign the single read's cDNA directly; for 2-read UMIs take the cDNA of the best-mapping read judged by minimap2's 'de' divergence tag; for >2 reads build a spoa partial-order-alignment consensus from up to the top MAXREADS=20 lowest-divergence reads, then polish with racon. Run ComputeConsensus with multi-threading, writing each molecule as '>BC-UMI-RN'.

## Related tools

- **Sicelore-2.1.jar (ComputeConsensus)** (Filters chimeric/MAPQ=0 reads, groups SAM records per BC+U8, and builds per-molecule consensus cDNA sequences) — https://github.com/ucagenomix/sicelore-2.1
- **spoa** (Partial order alignment (POA) multiple alignment used to generate the consensus sequence for UMIs covered by more than 2 reads (required for Step 4.b)) — https://github.com/rvaser/spoa
- **racon** (Polishes the poa multiple-alignment consensus for multi-read molecules)
- **samtools** (BAM/SAM manipulation and indexing of the barcode- and UMI-tagged alignment records feeding ComputeConsensus) — https://github.com/samtools/samtools
- **minimap2** (Provides the alignment and 'de' divergence tags used to pick the best-mapping read for 2-read UMIs and to rank top MAXREADS=20 reads) — https://github.com/lh3/minimap2

## Examples

```
java -jar Sicelore-2.1.jar ComputeConsensus I=passedParsedWithSequences.bam O=molecules.fastq T=20 TMPDIR=/tmp/consensus
```

## Evaluation signals

- Output FASTQ exists and every record header follows the '>BC-UMI-RN' naming scheme
- Record count in molecules.fastq equals the number of molecules surviving the BC/U8/GE-tag, MAXCLIP=150 and MAPQV0 filters
- Molecules with >2 reads yield consensus sequences that differ from any single input read (evidence of poa/racon consensus building rather than copy-through)
- cDNA span of each molecule lies within the TE (TSO end) to UE (UMI end) boundaries defined in the input tags
- No consensus derives from a record with MAPQ=0 or terminal clipping >150 nt

## Limitations

- Single-read and 2-read UMIs get no true error correction (read copied directly or best-mapping read chosen by 'de' divergence), so their accuracy is limited by raw Nanopore error rates
- Chimeric cDNA generated during PCR can survive splitting only up to the MAXCLIP (>150 nt terminal clipping) heuristic; chimeras below that clip threshold may pass into consensus
- Requires Java >= 13 and spoa installed; ComputeConsensus is memory/CPU intensive and needs explicit TMPDIR and thread settings (T=20, -Xmx tuning as in the sicelore quick-run setup)

## Evidence

- [methods] SAM records with extensive non-matching sequences at either end, hard- or soft clipping are discarded (MAXCLIP parameter, defaults to > 150 nt ): "SAM records with extensive non-matching sequences at either end, hard- or soft clipping are discarded (MAXCLIP parameter, defaults to > 150 nt )"
- [readme] generation of molecules consensus sequences (UMI-guided error-correction): [evidence span withheld — anchor status: external_doc]
- [readme] [spoa](https://github.com/rvaser/spoa) (require for step 4.b): "requires: * Java >= 13 * [samtools](https://github.com/samtools/samtools) * [minimap2](https://github.com/lh3/minimap2) * [spoa](https://github.com/rvaser/spoa) (require for step 4.b)"
- [other] Spoa (SIMD POA) is a c++ implementation of the partial order alignment (POA) algorithm: [evidence span withheld — anchor status: external_doc]
