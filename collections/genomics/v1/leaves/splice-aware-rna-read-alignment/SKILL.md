---
name: splice-aware-rna-read-alignment
description: Use when raw nanopore fast5 direct RNA reads have been basecalled to
  FASTA and must be mapped to a genome (e.g. GRCh38 or TAIR10) for downstream per-site
  m6A prediction with mAFiA.
when_to_use_negative:
- Input reads are already aligned (a sorted/indexed BAM exists) — re-alignment is
  unnecessary; proceed directly to mAFiA read-level prediction.
- Reads are DNA nanopore reads or require isoform/transcript-level quantification
  rather than genomic site-level modification calling — this pipeline uses -ax splice
  -uf tuned for direct RNA orientation, not general-purpose DNA mapping.
- The downstream tool does not require the mapq≥50 filter or splice alignment (e.g.
  short-read or synthetic oligo analysis handled with different aligner settings).
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3170
tools:
- name: minimap2.22
  role: Splice-aware aligner mapping basecalled dRNA reads to GRCh38/TAIR10 with --secondary=no
    -ax splice -uf -k14 --cs
- name: samtools
  role: Filters BAM to mapq >= 50 (samtools view -bST -q50), sorts, and indexes the
    alignment for mAFiA
- name: RODAN
  role: Fully-convolutional basecaller that converts raw fast5 signal to FASTA reads
    before alignment (basecall.py, backbone.torch, batchsize 4096)
  repo: https://github.com/biodlab/RODAN
- name: mAFiA
  role: 'Downstream consumer of the filtered BAM: requires the q50-filtered, sorted,
    indexed BAM plus fast5 files for per-read m6A prediction'
  repo: https://github.com/dieterich-lab/mAFiA
provenance:
  source_task_ids:
  - task_007
  - task_001
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-47661-2
    title: Detecting m6A at single-molecular resolution via direct RNA sequencing
      and realistic training data
  source_content_hash: sha256:0636a8037cbd9b6476163c019d8d43ecece19291e118885391bdf3ac944b8429
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biodlab/RODAN
  tools:
  - RODAN
  - mAFiA
  - minimap2
  - minimap2.22
  - samtools
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# splice-aware-rna-read-alignment

## Summary

Align RODAN-basecalled Oxford Nanopore dRNA-Seq reads to a genome reference with splice-aware minimap2, then filter, sort, and index the BAM with samtools to produce a high-quality (mapq≥50) alignment ready for mAFiA modification calling.

## When to use

Use when raw nanopore fast5 direct RNA reads have been basecalled to FASTA and must be mapped to a genome (e.g. GRCh38 or TAIR10) for downstream per-site m6A prediction with mAFiA.

## When NOT to use

- Input reads are already aligned (a sorted/indexed BAM exists) — re-alignment is unnecessary; proceed directly to mAFiA read-level prediction.
- Reads are DNA nanopore reads or require isoform/transcript-level quantification rather than genomic site-level modification calling — this pipeline uses -ax splice -uf tuned for direct RNA orientation, not general-purpose DNA mapping.
- The downstream tool does not require the mapq≥50 filter or splice alignment (e.g. short-read or synthetic oligo analysis handled with different aligner settings).

## Inputs

- RODAN-basecalled FASTA of nanopore dRNA-Seq reads (rodan.fasta)
- Raw fast5 directory (e.g. fast5_chrX, HEK293 WT/KO, Arabidopsis col0/vir1)
- Genome reference FASTA (GRCh38_96, GRCh38, or TAIR10)

## Outputs

- Filtered, sorted, indexed BAM (minimap.q50.bam) with samtools index
- Reads with mapq >= 50 suitable as test_mAFiA input

## How to apply

Basecall the fast5 files first with the RODAN-based basecall.py (backbone.torch model, batchsize 4096) to obtain rodan.fasta. Align the basecalled reads with minimap2.22 using mRNA-oriented splice parameters (--secondary=no -ax splice -uf -k14 -t 36 --cs) against the genome reference (GRCh38_96 for human HEK293, TAIR10 for Arabidopsis). Pipe the SAM directly through 'samtools view -bST <ref> -q50' to discard reads with mapping quality below 50, then samtools sort and samtools index to produce minimap.q50.bam plus its .bai. The -q50 filter and minimum site coverage of 50 are critical because mAFiA only scores sites and reads passing these thresholds, and the sorted, indexed BAM is consumed by test_mAFiA/process_reads together with the fast5 directory.

## Related tools

- **minimap2.22** (Splice-aware aligner mapping basecalled dRNA reads to GRCh38/TAIR10 with --secondary=no -ax splice -uf -k14 --cs)
- **samtools** (Filters BAM to mapq >= 50 (samtools view -bST -q50), sorts, and indexes the alignment for mAFiA)
- **RODAN** (Fully-convolutional basecaller that converts raw fast5 signal to FASTA reads before alignment (basecall.py, backbone.torch, batchsize 4096)) — https://github.com/biodlab/RODAN
- **mAFiA** (Downstream consumer of the filtered BAM: requires the q50-filtered, sorted, indexed BAM plus fast5 files for per-read m6A prediction) — https://github.com/dieterich-lab/mAFiA

## Examples

```
minimap2 --secondary=no -ax splice -uf -k14 -t 36 --cs ${ref} ${basecall} | samtools view -bST ${ref} -q50 - | samtools sort - > ${bam}; samtools index ${bam}
```

## Evaluation signals

- Output BAM exists as minimap.q50.bam with a successful samtools index (.bai) and no read has mapping quality below 50.
- minimap2 is run with the exact splice-aware, mRNA-oriented flags (--secondary=no -ax splice -uf -k14) with --secondary=no so no secondary alignments are retained.
- Downstream test_mAFiA accepts the BAM and produces mAFiA.sites.bed in which every reported site has dRNA-Seq coverage above the minimum threshold of 50.
- Downstream mAFiA.reads.bam is generated with MM and ML tags, confirming the alignment is compatible with per-read modification calling.
- For the METTL3 KO vs WT comparison, site genomic coordinates from KO and WT alignments join cleanly, reflecting conserved chromosomal methylation site locations.

## Limitations

- Reads with mapping quality below 50 are discarded, which can reduce coverage at poorly mappable loci below the mAFiA minimum site coverage of 50.
- In case of discontinuous alignment to the reference, only the section of the read with the longest continuous mapping is chosen, potentially dropping usable signal elsewhere in the read.
- Alignment settings were validated against GRCh38 and TAIR10 with minimap2.22; other references or read types may require re-tuned parameters.

## Evidence

- [readme] Align basecalling results to reference genome. Filter, sort, and index BAM file.: "minimap2 --secondary=no -ax splice -uf -k14 -t 36 --cs ${ref} ${basecall} | samtools view -bST ${ref} -q50 - | samtools sort - > ${bam}"
- [results] Reads with mapq below 50 are filtered out before m6A calling.: "Reads with mapq below 50 are filtered out."
- [results] A minimum dRNA-Seq coverage of 50 is required downstream, constraining alignment quality needs.: "A minimum dRNA-Seq coverage of 50 is required for a site to be compared against GLORI measurements"
- [results] Discontinuous alignments are handled by choosing the longest continuous mapping section.: "In case of discontinuous alignment to the reference, the section of the read with the longest continuous mapping is chosen."
- [other] Arabidopsis extension confirms the same alignment parameters transfer across genomes.: [evidence span withheld — anchor status: not_found]
