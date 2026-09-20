---
name: plasmodium-snp-database-annotation
description: Use when MinION-targeted amplicon reads (e.g., P. falciparum drug-resistance
  loci) have been mapped to a target with LAST via NanoPipe and the per-position polymorphisms
  need to be called and annotated against PlasmoDB SNP data.
when_to_use_negative:
- The input is metagenomic or mixed-species MinION data — NanoPipe does not yet support
  metagenomics analysis; it is designed for targeted read-to-reference alignment and
  SNP detection
- Positions have fewer than 10 reads — the consensus assigns a gap below this minimum,
  so polymorphism calls would be unreliable
- The reads are not aligned against a target database yet (no calc.lastalign.maf/calc.bam)
  — this skill operates downstream of the lastal/last-split mapping step
edam_operation: http://edamontology.org/operation_0331
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: NanoPipe
  role: Pipeline that computes nucleotide counts, majority-rule consensus, filtered
    polymorphisms, and annotates them against the local PlasmoDB SNP database (nanopipe_calc_polymorphism.py
    -s plasf)
  repo: https://github.com/IOB-Muenster/nanopipe2
- name: LAST
  role: Aligns MinION reads to the target (lastal version 946 with MinION-tuned parameters
    piped into last-split) to produce the MAF input for polymorphism calling
- name: samtools
  role: Converts the MAF to BAM (view -bS, sort, index) to supply sorted, indexed
    alignments for the polymorphism step and IGV browsing
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giy169
    title: NanoPipe—a web server for nanopore MinION sequencing data analysis
  source_content_hash: sha256:af57700b80ec9639f3daf7c59c7ee8f4aabde75200868577fcdb403fbc914a83
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IOB-Muenster/nanopipe2
  tools:
  - LAST
  - NanoPipe
  - samtools
  edam_operation: http://edamontology.org/operation_0331
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# plasmodium-snp-database-annotation

## Summary

Computes per-position nucleotide counts and majority-rule consensus from MinION amplicon alignments (NanoPipe/LAST), calls SNP candidates under NanoPipe's coverage and transition/transversion filters, and annotates them against a local PlasmoDB P. falciparum 3D7 SNP database using nanopipe_calc_polymorphism.py with -s plasf.

## When to use

Use when MinION-targeted amplicon reads (e.g., P. falciparum drug-resistance loci) have been mapped to a target with LAST via NanoPipe and the per-position polymorphisms need to be called and annotated against PlasmoDB SNP data.

## When NOT to use

- The input is metagenomic or mixed-species MinION data — NanoPipe does not yet support metagenomics analysis; it is designed for targeted read-to-reference alignment and SNP detection
- Positions have fewer than 10 reads — the consensus assigns a gap below this minimum, so polymorphism calls would be unreliable
- The reads are not aligned against a target database yet (no calc.lastalign.maf/calc.bam) — this skill operates downstream of the lastal/last-split mapping step

## Inputs

- calc.lastalign.maf (LAST/last-split alignment output from nanopipe_calc.pl)
- calc.bam and calc.bam.bai (samtools view -bS, sort, index)
- Local PlasmoDB Plasmodium falciparum 3D7 SNP database for all 14 chromosomes
- MinION query reads in FASTA/FASTQ (e.g., nanopipe2 testcase input.query or ENA study PRJEB30433)
- Target FASTA (e.g., Plasmodium 3D7 target package, indexed with lastdb)

## Outputs

- calc.nuccounts.1 (per-position nucleotide counts)
- calc.nuccounts.1.poly (filtered polymorphism table with PlasmoDB match counts, e.g. 'PlasmoDB local: 3 match(es); 0 artifact(s)')
- Majority-rule consensus sequence with IUPAC ambiguity codes
- SNP candidate list annotated with transitions/transversions weighting and p-error quality assessment

## How to apply

After nanopipe_calc.pl produces calc.lastalign.maf (lastal with -A 15 -B 4 -P 2 -Q 0 -S 1 -a 15 -b 3 -k 2 and the trained input.substmatrix) and calc.bam, download the PlasmoDB Plasmodium falciparum 3D7 SNP data for all 14 chromosomes locally. Run nanopipe_calc_polymorphism.py with the -s plasf flag to compute per-position nucleotide counts, a majority-rule consensus (minimum 10 reads per position, otherwise a gap; IUPAC codes when the top two nucleotide counts differ by <=20%), and the filtered polymorphism table (alternate coverage >=20% of all reads, read coverage >=30% of the maximum contig coverage, transitions weighted 2x versus transversions, p-error alignment-quality assessment). Judge success by the PlasmoDB match line in calc.nuccounts.1.poly, e.g. 'PlasmoDB local: 3 match(es); 0 artifact(s)', confirming the expected PfCRT/K-propeller mutations are annotated and not flagged as artifacts.

## Related tools

- **NanoPipe** (Pipeline that computes nucleotide counts, majority-rule consensus, filtered polymorphisms, and annotates them against the local PlasmoDB SNP database (nanopipe_calc_polymorphism.py -s plasf)) — https://github.com/IOB-Muenster/nanopipe2
- **LAST** (Aligns MinION reads to the target (lastal version 946 with MinION-tuned parameters piped into last-split) to produce the MAF input for polymorphism calling)
- **samtools** (Converts the MAF to BAM (view -bS, sort, index) to supply sorted, indexed alignments for the polymorphism step and IGV browsing)

## Examples

```
nanopipe_calc_polymorphism.py -q input.query -t input.target -s plasf
```

## Evaluation signals

- Alignment rate: ~99.9% of query reads mapped in the P. falciparum test case
- calc.nuccounts.1.poly reports the expected PfCRT/K-propeller mutations with a PlasmoDB annotation line such as 'PlasmoDB local: 3 match(es); 0 artifact(s)'
- Every reported SNP candidate has alternate-allele coverage >=20% of all reads and read coverage >=30% of the maximum contig coverage
- Every consensus position either has >=10 reads or is emitted as a gap; positions with top-two nucleotide counts differing by <=20% carry IUPAC ambiguity codes
- Run of the P. falciparum case completes in minutes (21 MB FASTA query against the P. falciparum genome took less than 4 minutes)

## Limitations

- SNP detection statistics are not sophisticated; more sophisticated statistics for SNP detection and evaluation are planned future work
- Positions with fewer than 10 reads are assigned a gap, and near-tie nucleotide counts (<=20% difference) yield ambiguous IUPAC consensus codes rather than a definitive call
- Candidate filtering weights transitions 2x versus transversions, so transversion candidates are penalized relative to transitions

## Evidence

- [methods] The minimum nucleotide count per position should be at least 10 (i.e., at least 10 reads), otherwise a gap will be assigned: "The minimum nucleotide count per position should be at least 10 (i.e., at least 10 reads), otherwise a gap will be assigned"
- [methods] Alternate coverage threshold: "suggests a polymorphism candidate if an alternate nucleotide has coverage of at least 20% of all reads"
- [methods] Transition/transversion weighting: "transversions are weighted by the factor of 1 and transitions by the factor of 2"
- [methods] PlasmoDB SNP database download: "choose Plasmodium falciparum 3D7 and download the SNP data for all 14 chromosomes"
- [results] Test-case validation: "NanoPipe succeeded in mapping 99.9% of all query reads and detecting the expected mutations"
