---
name: somatic-mutation-catalog-parsing
description: Use when raw somatic mutation calls in MAF or VCF format must be converted
  into a 96-channel (or SBS/DBS/ID) mutation count matrix, e.g. before de novo or
  InverseNMF signature refitting.
when_to_use_negative:
- Input is already a mutation count/context matrix (originalGenomes) — re-parsing
  from VCF/MAF is unnecessary
- The goal is de novo signature extraction or gene-signature association (CCA) rather
  than converting raw variant calls into count matrices
- Variants are not somatic (e.g. germline calls) or lack reference/alternate allele
  and genomic coordinate fields needed to infer trinucleotide context
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0199
tools:
- name: RNMF
  role: R package that parses VCF/MAF somatic variant files into SBS/DBS/ID mutation
    count matrices (SigsInput function)
  repo: https://github.com/zhenzhang-li/RNMF
- name: BSgenome.Hsapiens.UCSC.hg19
  role: Bioconductor genome library used to extract trinucleotide context (Ch37 build)
    when building the count matrix
- name: BSgenome.Hsapiens.UCSC.hg38
  role: Alternative Bioconductor genome library for hg38-build variant catalogs
- name: R
  role: Language in which RNMF is implemented and the parsing workflow is executed
  repo: https://github.com/zhenzhang-li/RNMF
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/cam4.4717
    title: A practical framework <scp>RNMF</scp> for exploring the association between
      mutational signatures and genes using gene cumulative contribution abundance
  source_content_hash: sha256:35456a0b2ab16367c498714dbc0e382c12d4822ef9fe8f26b7b29e81a1583901
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhenzhang-li/RNMF
  tools:
  - BSgenome.Hsapiens.UCSC.hg19
  - BSgenome.Hsapiens.UCSC.hg38
  - MutationalPatterns
  - R
  - RNMF
  - deconstructSigs
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# somatic-mutation-catalog-parsing

## Summary

Parse somatic variant catalogs (VCF or MAF files) into SBS/DBS/ID mutation type information matrices using the RNMF R package with Bioconductor BSgenome.Hsapiens.UCSC.hg19/hg38, producing the observed mutation count matrix V required for downstream NMF signature analysis.

## When to use

Use when raw somatic mutation calls in MAF or VCF format must be converted into a 96-channel (or SBS/DBS/ID) mutation count matrix, e.g. before de novo or InverseNMF signature refitting.

## When NOT to use

- Input is already a mutation count/context matrix (originalGenomes) — re-parsing from VCF/MAF is unnecessary
- The goal is de novo signature extraction or gene-signature association (CCA) rather than converting raw variant calls into count matrices
- Variants are not somatic (e.g. germline calls) or lack reference/alternate allele and genomic coordinate fields needed to infer trinucleotide context

## Inputs

- Somatic variant catalog in MAF format (e.g. ESCC.exon.ALL.input.maf with Tumor_Sample_Barcode, Chromosome, Start_position, Reference_Allele, Tumor_Seq_Allele2, Hugo_Symbol columns)
- Somatic variant catalog in VCF format
- Reference genome build (BSgenome.Hsapiens.UCSC.hg19 / Ch37 or BSgenome.Hsapiens.UCSC.hg38)

## Outputs

- Mutation type information/count matrix V of SBS, DBS, and ID mutation channels per sample (originalGenomes)
- sampleNames and subtypes vectors aligning samples to mutation subtype channels
- Parsed per-sample mutation channel counts ready for denovoNMF or InverseNMF refitting

## How to apply

Import somatic variants into RNMF from a Variant Call Format (VCF) file or a Mutation Annotation Format (MAF) file, mapping sample, chromosome, position, reference and alternate allele columns (e.g. Tumor_Sample_Barcode, Chromosome, Start_position, Reference_Allele, Tumor_Seq_Allele2 in MAF). Use the Bioconductor BSgenome.Hsapiens.UCSC.hg19 (Ch37) or hg38 library to retrieve trinucleotide context and classify each variant into SBS, DBS, or ID mutation types, yielding the information/count matrix V. Invoke SigsInput with Filetype='MAF', AnalCOSMICSigType (e.g. 'SBS'), and genome.build='Ch37' as shown in the README, then expose the resulting matrices as originalGenomes, sampleNames, and subtypes for the downstream denovoNMF or InverseNMF steps. Validate by checking that the count matrix dimensions match the number of samples and the expected 96/153/83 channel layout for SBS/DBS/ID respectively, and that totals equal the number of non-silent input variants.

## Related tools

- **RNMF** (R package that parses VCF/MAF somatic variant files into SBS/DBS/ID mutation count matrices (SigsInput function)) — https://github.com/zhenzhang-li/RNMF
- **BSgenome.Hsapiens.UCSC.hg19** (Bioconductor genome library used to extract trinucleotide context (Ch37 build) when building the count matrix)
- **BSgenome.Hsapiens.UCSC.hg38** (Alternative Bioconductor genome library for hg38-build variant catalogs)
- **R** (Language in which RNMF is implemented and the parsing workflow is executed) — https://github.com/zhenzhang-li/RNMF

## Examples

```
library(RNMF); SigsInput(file='ESCC.exon.ALL.input.maf', Filetype='MAF', AnalCOSMICSigType='SBS', genome.build="Ch37", sample.id='Tumor_Sample_Barcode', chr='Chromosome', pos='Start_position', ref='Reference_Allele', alt='Tumor_Seq_Allele2')
```

## Evaluation signals

- Count matrix row/column dimensions match the number of input samples and the expected mutation channel layout (e.g. 96 SBS channels)
- Total parsed mutation counts reconcile with the number of non-silent variants in the source MAF/VCF file
- Genome build used (Ch37 vs hg38) matches the coordinate system of the input variant file to avoid context misassignment
- Downstream InverseNMF/denovoNMF runs converge without dimension errors on the parsed matrix (within the 100,000-iteration cap)
- Refitted contribution scores are consistent with baseline tools such as deconstructSigs and MutationalPatterns on the same catalog

## Limitations

- Due to inconsistent sequencing background of the 1073-sample ESCC data, only coding regions were uniformly analyzed, which constrains how mixed WGS/WES catalogs can be combined
- Parsing depends on correct MAF column mapping and genome build annotation; mismatches silently corrupt trinucleotide context assignment

## Evidence

- [methods] Somatic variant import from VCF or MAF file: "somatic variants can be imported from a Variant Call Format (VCF) file or a Mutation Annotation Format (MAF) file"
- [methods] Bioconductor genome dependency: "Then it relies on the Bioconductor library, such as BS.genome.Hsapiens.UCSC.hg19 or BS.genome.Hsapiens.UCSC.hg38"
- [methods] Mutation type matrix generation: "to acquire an information matrix of mutation types (single‐base substitutions [SBS], double‐base substitutions [DBS], and insertion and deletion mutation [ID])"
- [methods] RNMF count-matrix capability: "our designed RNMF software provides the ability to obtain the count matrix of SBS, DBS, and ID directly from mutation data sets"
- [readme] README SigsInput invocation: "SigsInput(file, Filetype = 'MAF', AnalCOSMICSigType = 'SBS', genome.build = "Ch37", sample.id = 'Tumor_Sample_Barcode'"
