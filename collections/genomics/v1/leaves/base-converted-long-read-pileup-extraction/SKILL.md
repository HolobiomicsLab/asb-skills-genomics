---
name: base-converted-long-read-pileup-extraction
description: Use when you have C-to-T and G-to-A converted nanopore reads already
  mapped and -aligned (output_CT.sorted.bam, output_GA.sorted.bam) and need per-CpG
  methylation frequencies.
when_to_use_negative:
- Input is native (unconverted) nanopore signal data — use nanopolish call-methylation
  instead, which reads methylation directly from electric signals without enzymatic
  conversion
- The question requires distinguishing 5-methylcytosine from 5-hydroxymethylcytosine
  on the same molecule — nanoEM's conversion approach cannot separate mC and hmC
- Only short-read (non-converted) methylation data such as EMseq/bisulfite FASTQ is
  available — this pipeline's minimap2 map-ont alignment and dual-pileup logic applies
  to long converted reads
edam_operation: http://edamontology.org/operation_3206
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_0622
tools:
- name: sambamba
  role: Runs mpileup on the CT and GA sorted BAMs restricted to cpg_sites.bed to generate
    per-site pileup TSVs
- name: samtools
  role: Sorts, indexes, and views SAM/BAM files between pipeline stages (samtools
    sort -@ 8, samtools index)
- name: minimap2
  role: Maps converted long reads to the converted reference with -ax map-ont --eqx
    before pileup extraction
- name: pysam
  role: Used in call_methylation.py and downstream allele-assignment of long reads
    at heterozygous SNPs
- name: nanopolish
  role: Alternative direct signal-level methylation caller for native nanopore WGS,
    contrasted with the conversion-based pileup approach
  repo: github.com/jts/nanopolish
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkab397
    title: Long-read whole-genome methylation patterning using enzymatic base conversion
      and nanopore sequencing
  source_content_hash: sha256:61cf4626d5dd7faac79a0fa3c4900a616e117647afe07de9b3d73d8a193a8611
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/yos-sk/nanoEM
  tools:
  - Python3
  - minimap2
  - nanopolish
  - pysam
  - sambamba
  - samtools
  edam_operation: http://edamontology.org/operation_3206
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# base-converted-long-read-pileup-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extract per-CpG methylation observations from nanoEM data by pileup-ing the C-to-T and G-to-A converted BAM files at CpG sites, then combining the two strands' pileups into per-site methylation frequencies. This is the core methylation-calling step of the nanoEM enzymatic-conversion + nanopore pipeline.

## When to use

Use when you have C-to-T and G-to-A converted nanopore reads already mapped and best-aligned (output_CT.sorted.bam, output_GA.sorted.bam) and need per-CpG methylation frequencies.

## When NOT to use

- Input is native (unconverted) nanopore signal data — use nanopolish call-methylation instead, which reads methylation directly from electric signals without enzymatic conversion
- The question requires distinguishing 5-methylcytosine from 5-hydroxymethylcytosine on the same molecule — nanoEM's conversion approach cannot separate mC and hmC
- Only short-read (non-converted) methylation data such as EMseq/bisulfite FASTQ is available — this pipeline's minimap2 map-ont alignment and dual-pileup logic applies to long converted reads

## Inputs

- C-to-T converted FASTQ (*_CT.fq.gz) and G-to-A converted FASTQ (*_GA.fq.gz) from convert_reads.py
- Converted reference genome FASTA (output.fa from convert_ref.py)
- Best-alignment sorted/indexed BAMs: output_CT.sorted.bam and output_GA.sorted.bam
- CpG site coordinates in BED format (cpg_sites.bed)
- Reference genome FASTA (passed to sambamba mpileup via -f ref)

## Outputs

- pileup_CT.tsv and pileup_GA.tsv — sambamba mpileup outputs restricted to CpG sites
- frequency_methylation.tsv — per-CpG methylation frequencies produced by call_methylation.py
- Optionally allele-specific tables (e.g. allele_specific_methylation_GRB10_MEST.tsv) when reads are assigned to alleles with pysam

## How to apply

After base conversion of the reference (convert_ref.py) and reads (convert_reads.py), map each converted FASTQ to the converted reference with minimap2 -ax map-ont --eqx, sort and index with samtools, then run best_align.py to keep the best of the CT vs GA alignments per read, producing output_CT.sorted.bam and output_GA.sorted.bam. Define the CpG sites of interest in a BED file (cpg_sites.bed) and run sambamba mpileup separately on the CT and GA BAMs (8 threads, --samtools -f ref) to produce pileup_CT.tsv and pileup_GA.tsv. Feed both pileups to call_methylation.py, which aggregates converted-read observations and writes frequency_methylation.tsv with per-CpG methylation frequencies. Rationale: because unmethylated C is converted to T (and G to A on the opposite strand), residual C/G signal at CpGs in the two directional pileups indicates methylated cytosines, so the two pileups must be combined to recover strand-aware methylation calls. For locus-level or read-level questions (e.g. allele-specific methylation at imprinted loci), subset frequency_methylation.tsv or extract pileups at the locus BED and assign reads to alleles with pysam.

## Related tools

- **sambamba** (Runs mpileup on the CT and GA sorted BAMs restricted to cpg_sites.bed to generate per-site pileup TSVs)
- **samtools** (Sorts, indexes, and views SAM/BAM files between pipeline stages (samtools sort -@ 8, samtools index))
- **minimap2** (Maps converted long reads to the converted reference with -ax map-ont --eqx before pileup extraction)
- **pysam** (Used in call_methylation.py and downstream allele-assignment of long reads at heterozygous SNPs)
- **nanopolish** (Alternative direct signal-level methylation caller for native nanopore WGS, contrasted with the conversion-based pileup approach) — github.com/jts/nanopolish

## Examples

```
sambamba mpileup output_CT.sorted.bam -L cpg_sites.bed -o pileup_CT.tsv -t 8 --samtools -f ref && python src/call_methylation.py pileup_CT.tsv pileup_GA.tsv > frequency_methylation.tsv
```

## Evaluation signals

- pileup_CT.tsv and pileup_GA.tsv are non-empty and restricted to the CpG sites listed in cpg_sites.bed
- frequency_methylation.tsv contains one row per CpG with methylation frequencies consistent with the residual-C/residual-G counts in the two pileups
- At known imprinted loci (e.g. GRB10, MEST) the per-allele methylation frequencies are clearly separated, reproducing the published allele-specific pattern (one MEST allele methylated at the C of a heterozygous SNP, the other unmethylated)
- All BAMs (output_CT.sorted.bam, output_GA.sorted.bam) are sorted and indexed before mpileup, as the workflow requires samtools sort/index after best_align.py
- Genome-wide methylation patterns agree with the paper's expectation that CGIs show the expected methylation context in breast cancer cell lines such as MB231

## Limitations

- nanoEM cannot distinguish 5-methylcytosine from 5-hydroxymethylcytosine on the same DNA molecule, unlike direct nanopore WGS calling
- No software version numbers are given for minimap2, sambamba, samtools, or the pipeline scripts in the README or article
- The '--split-prefix' minimap2 option is only unnecessary for minimap2 built after April 7, 2023; older builds require it
- Short-read methylation analysis cannot resolve allele-specific methylation at imprinted loci the way long converted reads can, so pileups must come from long-read alignments

## Evidence

- [readme] sambamba mpileup over CT/GA BAMs at CpG sites followed by call_methylation.py: "sambamba mpileup output_CT.sorted.bam -L cpg_sites.bed -o pileup_CT.tsv -t 8 --samtools -f ref"
- [readme] call_methylation.py combines the two pileups into methylation frequencies: "python src/call_methylation.py pileup_CT.tsv pileup_GA.tsv > frequency_methylation.tsv"
- [abstract] Unmethylated cytosine is converted to thymine by the enzymatic conversion: "we combined recently developed enzymatic base conversion, where an unmethylated cytosine (C) should be converted to thymine (T), with nanopore sequencing"
- [methods] minimap2 ONT mapping of converted reads with --eqx: "minimap2 -t 8 --split-prefix temp_sam1 -ax map-ont output.fa  *_CT.fq.gz --eqx"
- [results] Allele-specific methylation at MEST resolved by allele assignment of long reads: [evidence span withheld — anchor status: not_found]
