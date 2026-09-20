---
name: long-read-genotyping
description: Use when you have a long-read bam/cram plus its reference genome and
  a fasta of insertion sequences (e.g. Alu, L1, SVA, HERV) and need to call non-reference
  TE insertions with haplotype/genotype support in VCF.
when_to_use_negative:
- Input is short-read data — sTELLeR is designed and tested for long-read data such
  as PacBio or ONT
- You need only reference-known TE calls from an annotated reference rather than non-reference
  insertion detection
- Coverage exceeds the --maxreads cap (default 100); calls in regions exceeding given
  coverage are avoided, so re-tune parameters or use a different caller
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0797
tools:
- name: sTELLeR
  role: 'Main caller: extracts split-read/insertion signatures, clusters candidates
    (DBSCAN), refines consensus, filters, and writes haplotype-aware VCF'
  repo: https://github.com/kristinebilgrav/sTELLeR
- name: minimap2
  role: Aligns candidate insertion sequences to the TE fasta (SAM intermediates ID_chr_repeats.sam)
- name: pysam
  role: Parses the input bam to extract split reads and insertion sequences spanning
    candidate TE insertions
- name: samtools
  role: Dependency for bam handling during candidate extraction
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btae686
    title: Detecting transposable elements in long-read genomes using sTELLeR
  source_content_hash: sha256:72ab2ca7de490813b0f59d139ee49932d5fae0d7fafec9b9b66077f2f217f1b8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kristinebilgrav/sTELLeR
  tools:
  - minimap2
  - pysam
  - python3
  - sTELLeR
  - samtools
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# long-read-genotyping

## Summary

Detect and genotype non-reference transposable element insertions from long-read (PacBio/ONT) data using sTELLeR, which clusters split-read insertion signatures, aligns candidate insertions to a TE fasta with minimap2, and emits haplotype-aware VCF calls.

## When to use

Use when you have a long-read bam/cram plus its reference genome and a fasta of insertion sequences (e.g. Alu, L1, SVA, HERV) and need to call non-reference TE insertions with haplotype/genotype support in VCF.

## When NOT to use

- Input is short-read data — sTELLeR is designed and tested for long-read data such as PacBio or ONT
- You need only reference-known TE calls from an annotated reference rather than non-reference insertion detection
- Coverage exceeds the --maxreads cap (default 100); calls in regions exceeding given coverage are avoided, so re-tune parameters or use a different caller

## Inputs

- bam or cram file of long reads (PacBio/ONT)
- reference genome fasta used for alignment (e.g. HG38_chr22.fa)
- fasta file of insertion sequences to detect (--TE_fasta, e.g. TEfastasequences.fa with L1 and Alu sequences)

## Outputs

- VCF file of TE insertion calls (variant name INS:ME, format TE:TS:HT:GT), e.g. testdata_res_repeats.vcf
- optional intermediate files with --keep_intermediates: per-chromosome candidate fasta (ID_chr_reads.fasta) and candidate-vs-TE alignments (ID_chr_repeats.sam)

## How to apply

Run steller.py with the reference genome (--ref), the TE fasta (--TE_fasta, e.g. fasta/TEfastasequences.fa containing Alu and L1 sequences), the long-read bam (--bam), and the platform style (--style ont or pb). sTELLeR parses split reads and insertion sequences with pysam, clusters candidate insertion signatures with DBSCAN, and aligns candidate insertion sequences to the TE fasta with minimap2. Apply the filter defaults: minimum 3 supporting split reads/insertions (--sr), mapping quality >= 20 to the specific TE sequence (--mq), and a maximum of 100 supporting reads to avoid calls in high-coverage regions (--maxreads). Inspect intermediate files (ID_chr_reads.fasta and ID_chr_repeats.sam) only if --keep_intermediates is set, and read the final testdata_res_repeats.vcf where variants are named INS:ME with format fields TE:TS:HT:GT (haplotag 0 if unphased, genotype 1/0, 1/1 unphased or 1|0, 0|1, 1|1 if phased).

## Related tools

- **sTELLeR** (Main caller: extracts split-read/insertion signatures, clusters candidates (DBSCAN), refines consensus, filters, and writes haplotype-aware VCF) — https://github.com/kristinebilgrav/sTELLeR
- **minimap2** (Aligns candidate insertion sequences to the TE fasta (SAM intermediates ID_chr_repeats.sam))
- **pysam** (Parses the input bam to extract split reads and insertion sequences spanning candidate TE insertions)
- **samtools** (Dependency for bam handling during candidate extraction)

## Examples

```
python steller/steller.py --ref HG38_chr22.fa --TE_fasta fasta/TEfastasequences.fa --bam testdata.bam --sr 4 --style pb -o testdata_res -mr 80
```

## Evaluation signals

- Run on the sTELLeR_supplementary test data (HG38_chr22.fa, fasta/TEfastasequences.fa, testdata.bam) and confirm a total of 7 variants are identified
- Compare the output testdata_res_repeats.vcf against the provided testbamTRUTH.txt for matching TE positions
- Check VCF variants are named INS:ME and format fields follow TE:TS:HT:GT
- Verify HT is 0 and GT is 1/0 or 1/1 for unphased input; HT is phased and GT is 1|0, 0|1 or 1|1 for phased input
- Confirm call counts change sensibly when varying --sr (default 3), --mq (default 20), and --maxreads (default 100)

## Limitations

- --maxreads (default 100) avoids calls in regions exceeding given coverage, so very high-coverage loci will be suppressed
- Haplotype (HT) information requires phased input; otherwise HT is 0 and only genotype 1/0 or 1/1 is reported
- Supporting reads threshold (--sr, default 3) depends on the coverage of the sample and the desired sensitivity
- The reference used in the supplementary test is a masked chr22 zip that must be unpacked (tar -xvzf) for correct reference bases

## Evidence

- [intro] a tool for detection of non-reference transposable element using long-read data such as PacBio or ONT: "a tool for detection of non-reference transposable element using long-read data such as PacBio or ONT"
- [methods] filter defaults: "-r , --sr         Minimum number of supporting split reads/insertions to call a variant (default 3)"
- [methods] mapping quality threshold: "-m , --mq         Mapping quality (default 20)"
- [methods] test expectation: "correct results can be verified by comparing to the provided file testbamTRUTH.txt"
- [readme] output format: "The variant is named INS:ME and in the info section the TE type along with start and end positions are given."
- [readme] haplotype handling: [evidence span withheld — anchor status: external_doc]
- [readme] example command: "python steller/steller.py --ref HG38_chr22.fa --TE_fasta fasta/TEfastasequences.fa --bam testdata.bam --sr 4 --style pb -o testdata_res -mr 80"
