---
name: mrna-length-integrity-profiling
description: Use when you have ONT cDNA reads aligned to an mRNA reference (sorted
  BAM) plus the transcript FASTA and need to quantify full-length fraction, fragmented
  fraction, and 5'/3' truncation proportions for mRNA vaccine quality control.
when_to_use_negative:
- Input is short-read (Illumina) data, which cannot span full mRNA length and therefore
  cannot resolve full-length vs. truncated transcript fractions
- The BAM contains only secondary alignments or unfiltered reads — extract primary
  aligned reads (SAMtools view -F 2048) first or length distributions will be biased
- Direct RNA sequencing data intended for chemistry/nucleoside-modification analysis
  (e.g., N1-methylpseudouridine detection) rather than cDNA-based length/integrity
  measurement
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3512
tools:
- name: Mana
  role: Command-line toolkit that analyses ONT alignments (.BAM) in --mrna mode to
    produce automated mRNA quality/integrity reports
  repo: https://github.com/scchess/Mana
- name: SAMtools
  role: Generates sorted and indexed BAM files and extracts primary aligned reads
    (view -F 2048) for read length distributions
- name: Minimap2
  role: Aligns ONT cDNA reads to the plasmid reference with -ax map-ont prior to Mana
    analysis
- name: Porechop
  role: Removes barcodes from nanopore reads before alignment
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-41354-y
    title: mRNA vaccine quality analysis using RNA sequencing
  source_content_hash: sha256:9fc990069ec1a6f1806d1009e7ed20bde4cc34580a0d77c624ffb6f7fa918546
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scchess/Mana
  tools:
  - Mana
  - Minimap2
  - Porechop
  - SAMtools
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# mrna-length-integrity-profiling

## Summary

Profiles the length distribution and integrity of a manufactured mRNA (e.g., eGFP mRNA, 1153 nt expected transcript) from ONT cDNA (VAX-seq) alignments using the Mana toolkit, quantifying full-length fraction and directional 5'/3' truncation.

## When to use

Use when you have ONT cDNA reads aligned to an mRNA reference (sorted BAM) plus the transcript FASTA and need to quantify full-length fraction, fragmented fraction, and 5'/3' truncation proportions for mRNA vaccine quality control.

## When NOT to use

- Input is short-read (Illumina) data, which cannot span full mRNA length and therefore cannot resolve full-length vs. truncated transcript fractions
- The BAM contains only secondary alignments or unfiltered reads — extract primary aligned reads (SAMtools view -F 2048) first or length distributions will be biased
- Direct RNA sequencing data intended for chemistry/nucleoside-modification analysis (e.g., N1-methylpseudouridine detection) rather than cDNA-based length/integrity measurement

## Inputs

- Sorted, indexed ONT cDNA alignment BAM file against the plasmid/transcript reference (e.g., alignment.bam)
- Reference transcript FASTA file (e.g., file.fasta with expected mRNA length such as 1153 nt)
- Optionally, Q>9-filtered FASTQ reads preprocessed with Porechop and aligned via Minimap2 -ax map-ont

## Outputs

- Mana mRNA quality report with mRNA length distribution relative to expected transcript length
- Full-length read fraction (e.g., 58.2% of reads encompassing full mRNA length)
- Fragmented fraction (e.g., 23%) and directional truncation proportions (7% 3'-truncated, 31.3% 5'-truncated, 3.5% both)

## How to apply

Pull the pre-built Mana docker image (docker pull scchess/mana && docker tag scchess/mana mana) so SAMtools, BCFtools, Pysam, and bedtools dependencies are resolved automatically. Mount the working directory with -v ${PWD}:/src and run Mana in --mrna mode with the alignment BAM (-b) and transcript FASTA (-f); the analysis takes about 10 minutes. Prior to Mana, nanopore reads with Q>9 should be barcode-trimmed with Porechop v0.2.4, aligned with Minimap2 -ax map-ont, and primary alignments extracted via SAMtools view -F 2048 for length distributions. Read the generated report and extract the length distribution relative to the expected transcript (e.g., 1153 nt eGFP mRNA, 77% within 5% of expected), full-length fraction (58.2%), fragmented fraction (23%), and truncation classes (7% 3'-truncated, 31.3% 5'-truncated, 3.5% both ends).

## Related tools

- **Mana** (Command-line toolkit that analyses ONT alignments (.BAM) in --mrna mode to produce automated mRNA quality/integrity reports) — https://github.com/scchess/Mana
- **SAMtools** (Generates sorted and indexed BAM files and extracts primary aligned reads (view -F 2048) for read length distributions)
- **Minimap2** (Aligns ONT cDNA reads to the plasmid reference with -ax map-ont prior to Mana analysis)
- **Porechop** (Removes barcodes from nanopore reads before alignment)

## Examples

```
docker run -v ${PWD}:/src -i -t mana python3 mana.py --mrna -b /src/alignment.bam -f /src/file.fasta -o /src/out
```

## Evaluation signals

- Reported length distribution shows ~77% of mRNA reads within 5% of the expected 1153 nt eGFP transcript length
- Full-length fraction matches published values (~58.2% of reads encompassing full mRNA length; 23% fragmented)
- Directional truncation classes sum consistently: 7% 3'-truncated, 31.3% 5'-truncated, 3.5% truncated at both ends
- Docker run completes within the expected ~10 minutes analysis time and writes the report to the -o output directory
- Input BAM is sorted/indexed and contains primary alignments (SAMtools view -F 2048) rather than secondary alignments

## Limitations

- Data are proof of concept with small sample sizes (n = 1 to n = 4); no statistical method was used to predetermine sample size and experiments were not randomised
- Building Mana from source instead of using the pre-built docker image takes >= 20 minutes and is not recommended
- cDNA sequencing was found more reliable than direct RNA sequencing; length/integrity conclusions depend on alignment to the plasmid/transcript reference

## Evidence

- [readme] Mana --mrna invocation: "docker run -v ${PWD}:/src -i -t mana \
               python3 mana.py --mrna \
               -b /src/alignment.bam \
               -f /src/file.fasta \
               -o /src/out"
- [abstract] Mana abstract description: "Command-line tool that analyses ONT alignments (.BAM) to report quality control statistics."
- [discussion] primary read extraction for length distributions: "SAMtools (v1.15) utilising SAMtools view -F 2048 was used to extract read length distributions of the primary aligned reads"
- [abstract] VAX-seq quality attributes: "VAX-seq can comprehensively measure key mRNA vaccine quality attributes, including sequence, length, integrity, and purity"
- [intro] Mana purpose: "To support VAX-seq, we also develop a software toolkit, Mana, that provides detailed and automated reports of mRNA quality."
