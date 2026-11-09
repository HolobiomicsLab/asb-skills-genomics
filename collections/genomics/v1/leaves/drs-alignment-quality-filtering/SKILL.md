---
name: drs-alignment-quality-filtering
description: Use when quantifying Oxford Nanopore direct-RNA sequencing reads against
  a transcriptome with multi-mapping uncertainty, and implausible alignments (e.g.
  3' ends far from transcript ends, weak secondary AS scores) must be removed before
  EM abundance estimation.
when_to_use_negative:
- Input is short-read RNA-seq data — the filters target DRS artefacts (3' truncation,
  strand and polyA-proximity) and NanoCount is developed for DRS only
- Reads are aligned to the genome rather than a transcriptome with retained secondary
  mappings (-p 0); without multi-mapping alignments the EM step cannot resolve isoforms
- The goal is alignment QC visualisation only, not quantification — BamSlam summarises/plots
  alignments without the EM abundance step
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0102
tools:
- name: NanoCount
  role: Applies the DRS-specific alignment filters and then quantifies transcripts
    via expectation-maximisation over the filtered read-transcript compatibility index
  repo: https://github.com/a-slide/NanoCount
- name: minimap2
  role: Aligns DRS FASTQ reads to the transcriptome with '-ax map-ont' and '-p 0'
    to retain up to 100 secondary mappings per read for downstream filtering
  repo: https://github.com/lh3/minimap2
- name: samtools
  role: Converts minimap2 SAM output to BAM (samtools view -bh) feeding aligned_reads.bam
    into the filtering step
- name: BamSlam
  role: R script for summary plots/QC of the aligned reads before or after filtering;
    can also consume BAM output from NanoCount
  repo: https://github.com/josiegleeson/BamSlam
- name: nanopore-rna-analysis
  role: Prior art by Jared Simpson whose EM transcript abundance estimator inspired
    NanoCount's approach
  repo: https://github.com/jts/nanopore-rna-analysis
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkab1129
    title: Accurate expression quantification from nanopore direct RNA sequencing
      with NanoCount
  source_content_hash: sha256:b2b8fa910b10ae15973b25fcbb25f8a19b9b3845eabc4dc267b0198ce04d4837
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/a-slide/NanoCount
  tools:
  - BamSlam
  - NanoCount
  - minimap2
  - nanopore-rna-analysis
  - samtools
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# drs-alignment-quality-filtering

## Summary

A DRS-specific alignment filtering step, implemented in NanoCount, that discards implausible alignments (short/partial alignments, wrong-strand, 3'-end mismatch, weak secondary hits) before EM-based transcript quantification to improve isoform assignment from nanopore direct-RNA reads.

## When to use

Use when quantifying Oxford Nanopore direct-RNA sequencing reads against a transcriptome with multi-mapping uncertainty, and implausible alignments (e.g. 3' ends far from transcript ends, weak secondary AS scores) must be removed before EM abundance estimation.

## When NOT to use

- Input is short-read RNA-seq data — the filters target DRS artefacts (3' truncation, strand and polyA-proximity) and NanoCount is developed for DRS only
- Reads are aligned to the genome rather than a transcriptome with retained secondary mappings (-p 0); without multi-mapping alignments the EM step cannot resolve isoforms
- The goal is alignment QC visualisation only, not quantification — BamSlam summarises/plots alignments without the EM abundance step

## Inputs

- Direct-RNA sequencing FASTQ reads (e.g. SIRV mix E0/E2 spike-in reads for SRR6058583 from ENA)
- Transcriptome reference FASTA (e.g. SIRV transcripts or GENCODE v31)
- BAM of minimap2 transcriptome alignments retaining secondary mappings (aligned_reads.bam)

## Outputs

- Filtered read-transcript compatibility index of surviving alignments
- Per-transcript counts table (sirv_transcript_counts.tsv) with transcript_name, raw abundance (summing to 1), est_count (raw abundance x number of primary alignments) and tpm (raw abundance x 1M)
- Filtered BAM of alignments passing DRS-specific criteria (NanoCount output BAM)

## How to apply

Align direct-RNA FASTQ reads to a transcriptome reference (e.g. SIRV or GENCODE) with minimap2 using '-ax map-ont' and '-p 0' so all secondary mappings are retained, converting SAM to BAM with samtools view -bh. Then apply the NanoCount DRS-specific filters: discard alignments with aligned fraction <0.5, alignment length <50 nt, negative-strand orientation, or 3' ends >50 nt from the transcript 3' end; for secondary alignments keep only those with an AS score at least 95% of the best alignment's AS, as weaker hits likely represent misalignments or reads from unannotated isoforms. Build the read-transcript compatibility index from surviving alignments and run the expectation-maximisation loop to convergence to resolve multi-mapping reads. The 3' filtering is essential because many incomplete DRS reads are consigned to 3'UTRs and cannot specify the expressed isoform, so only polyA-proximal, confidently oriented alignments are informative for quantification.

## Related tools

- **NanoCount** (Applies the DRS-specific alignment filters and then quantifies transcripts via expectation-maximisation over the filtered read-transcript compatibility index) — https://github.com/a-slide/NanoCount
- **minimap2** (Aligns DRS FASTQ reads to the transcriptome with '-ax map-ont' and '-p 0' to retain up to 100 secondary mappings per read for downstream filtering) — https://github.com/lh3/minimap2
- **samtools** (Converts minimap2 SAM output to BAM (samtools view -bh) feeding aligned_reads.bam into the filtering step)
- **BamSlam** (R script for summary plots/QC of the aligned reads before or after filtering; can also consume BAM output from NanoCount) — https://github.com/josiegleeson/BamSlam
- **nanopore-rna-analysis** (Prior art by Jared Simpson whose EM transcript abundance estimator inspired NanoCount's approach) — https://github.com/jts/nanopore-rna-analysis

## Examples

```
minimap2 -ax map-ont -p 0 -N 100 sirv_transcriptome.fasta SRR6058583.fastq | samtools view -bh > aligned_reads.bam && nanocount -i aligned_reads.bam -o sirv_transcript_counts.tsv
```

## Evaluation signals

- sirv_transcript_counts.tsv exists with transcript_name/raw/est_count/tpm fields present
- Raw abundance values across the 69 SIRV transcripts sum to 1
- Output covers all 69 SIRV transcripts in the reference set
- Minimap2 run used -p 0 so multiple secondary alignments per read are present in aligned_reads.bam (required for the 95%-of-best-AS secondary filter to act)
- Aligned fraction, alignment length, strand and 3'-end-distance columns verify each retained alignment passes the <0.5, <50 nt, negative-strand and >50 nt cutoffs

## Limitations

- Modest DRS sequencing depth limits detection and quantification of lowly expressed genes and isoforms
- Many incomplete reads are largely consigned to 3'UTRs and cannot specify the expressed isoform, so filtered-out reads carry no isoform information
- Read truncation due to software limitations remains an avenue for improvement
- Caution extrapolating validation results from spike-ins (sequins/SIRVs) because their transcriptome complexity is lower than the human one
- Secondary alignments with AS <95% of the best may include genuine reads from unannotated isoforms, not only misalignments

## Evidence

- [results] 3' end distance filter: "discard read alignments with 3′ ends >50nt away from the transcript 3′ end"
- [results] secondary AS filter: "and secondary alignments with an AS <95% that of the best alignment (highest AS), as these likely represent misalignments or reads from unannotated isoforms"
- [results] EM quantification after filtering: "NanoCount performs DRS specific filtering steps to improve transcript assignment prior to quantifying DRS reads using an expectation-maximisation algorithm"
- [discussion] 3'UTR truncation rationale: "many incomplete reads are largely consigned to 3′UTRs and cannot specify the expressed isoform"
- [readme] EM tool purpose: "NanoCount estimates transcripts abundance from Oxford Nanopore direct-RNA sequencing datasets, using an expectation-maximization approach like RSEM, Kallisto, salmon, etc to handle the uncertainty of"
