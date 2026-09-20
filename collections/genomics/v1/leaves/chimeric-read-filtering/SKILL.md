---
name: chimeric-read-filtering
description: Use when a Nanopore BAM with BC, U8 and GE tags is about to be collapsed
  into per-UMI consensus sequences and you must drop chimeric (clipped) reads and
  MAPQ=0 alignments beforehand.
when_to_use_negative:
- Input is raw Nanopore FASTQ before barcode/UMI assignment — chimeras are first split
  and reads stranded in Step 1 (NanoporeBC_UMI_finder.jar), not filtered via MAXCLIP
  on a BAM
- You only need short-read-free matrix generation (Step 4a) that skips consensus sequences
  per UMI and thus does not require the ComputeConsensus filtering chain
- Clipped reads are intentionally being exported for inspection (ExportClippedReads
  with MINCLIP) rather than discarded
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3308
tools:
- name: Sicelore-2.1.jar
  role: ComputeConsensus pipeline that applies the MAXCLIP and MAPQV0 filters and
    then builds per-UMI consensus sequences
  repo: https://github.com/ucagenomix/sicelore-2.1
- name: samtools
  role: BAM/SAM manipulation of the barcode- and UMI-tagged records fed into ComputeConsensus
  repo: https://github.com/samtools/samtools
- name: NanoporeBC_UMI_finder.jar
  role: 'Step 1 scanfastq: splits chimeric cDNA reads generated during library preparation
    before the BAM-level MAXCLIP filter'
  repo: https://github.com/ucagenomix/sicelore
- name: spoa
  role: Multiple alignment used downstream of filtering to build the per-UMI consensus
    from surviving reads
  repo: https://github.com/rvaser/spoa
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-17800-6
    title: High throughput error corrected Nanopore single cell transcriptome sequencing
  source_content_hash: sha256:8397093fe7c80d4eaee6e7f0c2b068edf563428291c1d8108a081078e472371b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ucagenomix/sicelore-2.1
  tools:
  - Java
  - NanoporeBC_UMI_finder.jar
  - Sicelore-2.1.jar
  - Sicelore-2.1.jar (ComputeConsensus pipeline)
  - samtools
  - spoa
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# chimeric-read-filtering

## Summary

Removes potentially chimeric cDNA reads (excessive terminal hard/soft clipping, MAXCLIP > 150 nt) and low-confidence alignments (minimap2 MAPQ = 0) from barcode-, UMI- and gene-tagged Nanopore SAM records so that downstream UMI-guided consensus generation in Sicelore's ComputeConsensus is not corrupted by chimeric or poorly mapped molecules.

## When to use

Use when a Nanopore BAM with BC, U8 and GE tags is about to be collapsed into per-UMI consensus sequences and you must drop chimeric (clipped) reads and MAPQ=0 alignments beforehand.

## When NOT to use

- Input is raw Nanopore FASTQ before barcode/UMI assignment — chimeras are first split and reads stranded in Step 1 (NanoporeBC_UMI_finder.jar), not filtered via MAXCLIP on a BAM
- You only need short-read-free matrix generation (Step 4a) that skips consensus sequences per UMI and thus does not require the ComputeConsensus filtering chain
- Clipped reads are intentionally being exported for inspection (ExportClippedReads with MINCLIP) rather than discarded

## Inputs

- barcode-, UMI- and gene-tagged Nanopore BAM file (passedParsedWithSequences.bam) with BC, U8, GE, TE and UE tags
- Sicelore-2.1.jar (ComputeConsensus pipeline)
- config parameters: MAXCLIP (default 150), MAPQV0 (default false)

## Outputs

- Filtered set of SAM records per cell barcode and UMI, with chimeric (>150 nt terminal clipping) and MAPQ=0 reads removed
- Molecule consensus FASTQ with '>BC-UMI-RN' headers (molecules.fastq)

## How to apply

Load the barcode-, UMI- and gene-annotated BAM (passedParsedWithSequences.bam from Step 4.b.1) and keep only SAM records carrying BC, U8 and GE tags, discarding any record lacking one of them. Discard reads with terminal hard- or soft-clipping exceeding the MAXCLIP threshold (default > 150 nt), since extensive non-matching sequence at either read end signals chimeric cDNA generated during library preparation; splitting of such chimeras already begins at Step 1 (NanoporeBC_UMI_finder.jar scanfastq). Additionally drop records with minimap2 MAPQ = 0 (MAPQV0=false default) so poorly mapped reads never contribute to a consensus. Only after these filters does Sicelore ComputeConsensus group records per cell barcode (BC) and UMI (U8), defining the cDNA span from the TSO end (TE tag) to the UMI end (UE tag) and building poa/racon consensus from the remaining reads.

## Related tools

- **Sicelore-2.1.jar** (ComputeConsensus pipeline that applies the MAXCLIP and MAPQV0 filters and then builds per-UMI consensus sequences) — https://github.com/ucagenomix/sicelore-2.1
- **samtools** (BAM/SAM manipulation of the barcode- and UMI-tagged records fed into ComputeConsensus) — https://github.com/samtools/samtools
- **NanoporeBC_UMI_finder.jar** (Step 1 scanfastq: splits chimeric cDNA reads generated during library preparation before the BAM-level MAXCLIP filter) — https://github.com/ucagenomix/sicelore
- **spoa** (Multiple alignment used downstream of filtering to build the per-UMI consensus from surviving reads) — https://github.com/rvaser/spoa

## Examples

```
java -jar Sicelore-2.1.jar ComputeConsensus I=chr1.bam O=molecules.fastq T=20 TMPDIR=<tmp>
```

## Evaluation signals

- Output FASTQ exists and every record header follows the '>BC-UMI-RN' naming scheme
- Record count in the output FASTQ equals the number of molecules surviving the BC/U8/GE-tag, MAXCLIP and MAPQV0 filters
- No retained SAM record has terminal hard/soft clipping > 150 nt or minimap2 MAPQ = 0
- Every surviving record carries all three of the BC, U8 and GE tags
- Each molecule's cDNA span is bounded by the TE (TSO end) and UE (UMI end) tags

## Limitations

- MAXCLIP default (>150 nt) may retain reads with moderate chimeric structure and discards some genuine reads with large clipped ends
- Chimera splitting in Step 1 relies on detecting polyA/T plus adapter features and is skipped when --noPolyARequired is set (e.g. RACE PCR with 5' barcoding)
- Dropping MAPQ=0 records (MAPQV0=false default) sacrifices molecules that could not be uniquely mapped

## Evidence

- [methods] SAM records with extensive non-matching sequences at either end, hard- or soft clipping are discarded (MAXCLIP parameter, defaults to > 150 nt ): "SAM records with extensive non-matching sequences at either end, hard- or soft clipping are discarded (MAXCLIP parameter, defaults to > 150 nt )"
- [readme] Scans for chimeric cDNA generated during library preparation and splits chimeric reads .: "Scans for chimeric cDNA generated during library preparation and splits chimeric reads ."
- [abstract] UMI guided error correction allows to generate high accuracy full length sequence information: "UMI guided error correction allows to generate high accuracy full length sequence information"
