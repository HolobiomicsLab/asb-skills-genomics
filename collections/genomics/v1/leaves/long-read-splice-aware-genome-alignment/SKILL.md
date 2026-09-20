---
name: long-read-splice-aware-genome-alignment
description: Use when PacBio full-length transcript reads (isoseq_flnc.Transcript.fastq)
  must be spliced-aligned to a reference genome before isoform annotation or single-cell
  quantification.
when_to_use_negative:
- Input reads are raw CCS BAM or untrimmed reads that still contain primers, cell
  barcodes, UMIs, or polyA tails — run classify_by_primer FLNC detection first.
- The goal is short-read or genomic (non-transcript) alignment where splice-aware
  '-ax splice' presets and '-uf' strand restriction are inappropriate.
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3170
tools:
- name: scISA-Tools
  role: Provides the pipeline context and the classify_by_primer step that produces
    the FLNC transcript fastq input for alignment
  repo: https://github.com/shizhuoxing/scISA-Tools
- name: minimap2
  role: Performs the splice-aware ('-ax splice -uf --secondary=no -C5') alignment
    of FLNC transcript reads to the reference genome, yielding isoseq_flnc.Transcript.sam
- name: sam2gff.pl
  role: Consumes the alignment SAM output downstream, converting it to GFF for gffcompare
    annotation matching
  repo: https://github.com/gpertea/gscripts/blob/master/sam2gff.pl
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-38324-9
    title: High-throughput and high-accuracy single-cell RNA isoform analysis using
      PacBio circular consensus sequencing
  source_content_hash: sha256:5386485303061dfaf3bf9b55154c90a42103c031993d1d73a09372975dea7eba
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shizhuoxing/scISA-Tools
  tools:
  - minimap2
  - sam2gff.pl
  - scISA-Tools
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# long-read-splice-aware-genome-alignment

## Summary

Align full-length non-concatemer (FLNC) transcript reads to a reference genome with minimap2 in splice-aware mode, a prerequisite step for downstream gene assignment (gffcompare) and single-cell count matrix generation in the scISA-Tools/HIT-scISOseq pipeline.

## When to use

Use when PacBio full-length transcript reads (isoseq_flnc.Transcript.fastq) must be spliced-aligned to a reference genome before isoform annotation or single-cell quantification.

## When NOT to use

- Input reads are raw CCS BAM or untrimmed reads that still contain primers, cell barcodes, UMIs, or polyA tails — run classify_by_primer FLNC detection first.
- The goal is short-read or genomic (non-transcript) alignment where splice-aware '-ax splice' presets and '-uf' strand restriction are inappropriate.

## Inputs

- isoseq_flnc.Transcript.fastq (full-length non-concatemer transcript reads after primer/cellBC/UMI/polyA trimming)
- Reference genome FASTA (e.g., ref.genome.fa, such as Ensembl Macaca_fascicularis_5.0.99)

## Outputs

- isoseq_flnc.Transcript.sam (splice-aware SAM alignments of FLNC reads to the reference genome)

## How to apply

After FLNC detection and cell barcode, UMI, and polyA tail trimming via classify_by_primer, align the remaining transcript reads (isoseq_flnc.Transcript.fastq) to the reference genome using minimap2 in splice mode with the parameters '-ax splice -uf --secondary=no -C5' to produce isoseq_flnc.Transcript.sam. The '-uf' flag restricts to strands consistent with transcript direction, '--secondary=no' suppresses secondary alignments so downstream gene assignment stays unambiguous, and '-C5' penalizes chimeric/missing transcript segments, which matters because HIT-scISOseq concatenated reads span multiple cDNAs. Export minimap2 (via SMRTlink) to the PATH and use multiple threads (-t) for the multi-GB FLNC fastq. The SAM output is then converted to GFF with sam2gff.pl and compared against a reference annotation GTF (e.g., ref.genes.gtf) with gffcompare, so alignment quality directly determines tmap match-code accuracy and the final scGene_matrix gene counts.

## Related tools

- **scISA-Tools** (Provides the pipeline context and the classify_by_primer step that produces the FLNC transcript fastq input for alignment) — https://github.com/shizhuoxing/scISA-Tools
- **minimap2** (Performs the splice-aware ('-ax splice -uf --secondary=no -C5') alignment of FLNC transcript reads to the reference genome, yielding isoseq_flnc.Transcript.sam)
- **sam2gff.pl** (Consumes the alignment SAM output downstream, converting it to GFF for gffcompare annotation matching) — https://github.com/gpertea/gscripts/blob/master/sam2gff.pl

## Examples

```
minimap2 -ax splice -t 6 -uf --secondary=no -C5 ref.genome.fa isoseq_flnc.Transcript.fastq > isoseq_flnc.Transcript.sam
```

## Evaluation signals

- The output file isoseq_flnc.Transcript.sam exists and contains splice-aware alignments (CIGAR strings with N operations) consistent with '-ax splice' mode.
- Only primary alignments are present, confirming '--secondary=no' was honored.
- Read names in the SAM match those of isoseq_flnc.Transcript.fastq, confirming FLNC reads (not raw CCS reads) were aligned.
- Downstream sam2gff.pl conversion produces a well-formed isoseq_flnc.Transcript.gff and gffcompare yields a .tmap file, indicating the SAM is structurally valid.
- Aligned read count approximately equals FLNC read count from classify_by_primer, with expected loss only from unmapped reads.

## Limitations

- Alignment accuracy depends on prior FLNC detection quality: reads lacking a 16 bp minimum primer alignment or 50 bp minimum transcript length are excluded upstream.
- Secondary alignments are suppressed, so multi-locus reads get a single best placement; ambiguous multimappers downstream are handled only by filter.tmap.pl unique-mapping selection.
- The pipeline assumes strand-consistent reads ('-uf'); incorrectly oriented reads will fail alignment.

## Evidence

- [readme] minimap2 -ax splice -t 6 -uf --secondary=no -C5 -t 5 ref.genome.fa isoseq_flnc.Transcript.fastq > isoseq_flnc.Transcript.sam: "minimap2 -ax splice -t 6 -uf --secondary=no -C5 -t 5 ref.genome.fa isoseq_flnc.Transcript.fastq > isoseq_flnc.Transcript.sam"
- [methods] Align FLNC transcripts to the reference genome with minimap2 in splice mode using the parameters '-ax splice -uf --secondary=no -C5', producing isoseq_flnc.Transcript.sam.: [evidence span withheld — anchor status: not_found]
- [readme] the remaining fraction of each isoforms FLNC reads were aligned to the reference genome with `minimap2`.: "After FLNC detection and primer, cellBC, UMI and polyA tail trimming, the remaining fraction of each isoforms FLNC reads were aligned to the reference genome with `minimap2`."
- [discussion] FLNC reads were generated with the following parameters: "-min_primerlen 16 -min_seqlen 50": [evidence span withheld — anchor status: not_found]
