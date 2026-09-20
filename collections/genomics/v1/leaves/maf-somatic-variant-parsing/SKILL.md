---
name: maf-somatic-variant-parsing
description: Use when the input is raw somatic mutation data in MAF or VCF format
  (e.g. EXON1073 or WGS508 ESCC files) and it must be converted into RNMF's SBS/ID
  count matrices or joined with signature-contribution tables by Tumor_Sample_Barcode
  and Hugo_Symbol.
when_to_use_negative:
- Input is already a mutation-type count matrix or signature contribution table (Sfile)
  — parsing is unnecessary and could duplicate normalization
- Data are non-somatic germline variants or lack the required MAF fields (Tumor_Sample_Barcode,
  Reference_Allele, Tumor_Seq_Allele2), since RNMF's column mapping cannot key records
- Analysis targets whole-genome trinucleotide contexts when only inconsistent exon/coding-region
  data are available — the article notes coding-region restriction introduces limitations
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0199
tools:
- name: RNMF
  role: Provides SigsInput to import MAF/VCF somatic variants and build the SBS/DBS/ID
    count matrices used by denovoNMF, InverseNMF, and the sam*/genePerMutSigs association
    functions
  repo: https://github.com/zhenzhang-li/RNMF
- name: R
  role: Language in which RNMF is implemented and invoked as a library
- name: BSgenome.Hsapiens.UCSC.hg19
  role: Bioconductor genome build (Ch37) used to reconstruct mutation contexts from
    parsed variant coordinates
- name: BSgenome.Hsapiens.UCSC.hg38
  role: Alternative Bioconductor genome build (Ch38) for context reconstruction
provenance:
  source_task_ids:
  - task_004
  - task_003
  - task_001
  source_papers:
  - doi: 10.1002/cam4.4717
    title: A practical framework <scp>RNMF</scp> for exploring the association between
      mutational signatures and genes using gene cumulative contribution abundance
  source_content_hash: sha256:29bbf7f23f547f0f12b64daf967dada7f7f51e6a55a94322bc3e005b14bc6fc2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhenzhang-li/RNMF
  tools:
  - BSgenome.Hsapiens.UCSC.hg19
  - BSgenome.Hsapiens.UCSC.hg38
  - Bioconductor
  - R
  - RNMF
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# maf-somatic-variant-parsing

## Summary

Parse somatic variants from a MAF (or VCF) file and build the mutation-type information matrix (SBS, DBS, ID) that RNMF requires as input, keyed on Tumor_Sample_Barcode and Hugo_Symbol, before any signature extraction or gene-association analysis in the ESCC cohorts.

## When to use

Use when the input is raw somatic mutation data in MAF or VCF format (e.g. EXON1073 or WGS508 ESCC files) and it must be converted into RNMF's SBS/ID count matrices or joined with signature-contribution tables by Tumor_Sample_Barcode and Hugo_Symbol.

## When NOT to use

- Input is already a mutation-type count matrix or signature contribution table (Sfile) — parsing is unnecessary and could duplicate normalization
- Data are non-somatic germline variants or lack the required MAF fields (Tumor_Sample_Barcode, Reference_Allele, Tumor_Seq_Allele2), since RNMF's column mapping cannot key records
- Analysis targets whole-genome trinucleotide contexts when only inconsistent exon/coding-region data are available — the article notes coding-region restriction introduces limitations

## Inputs

- MAF-format somatic mutation file (e.g. ESCC.exon.ALL.input.maf) with Tumor_Sample_Barcode, Chromosome, Start_position, Reference_Allele, Tumor_Seq_Allele2, Hugo_Symbol columns
- VCF-format somatic variant file (alternative input supported by RNMF)
- Genome build specification (Ch37/Ch38) via BSgenome.Hsapiens.UCSC.hg19 or BSgenome.Hsapiens.UCSC.hg38

## Outputs

- Mutation-type information count matrices (SBS, DBS, ID) feeding denovoNMF or InverseNMF
- originalGenomes matrix, sampleNames, and subtypes objects consumed by denovoNMF
- MAF records joined to gene symbols for samPerMutSigs/genePerMutSigs association analyses

## How to apply

Import the somatic variant file into the RNMF R package using SigsInput with Filetype='MAF', supplying the column mapping sample.id='Tumor_Sample_Barcode', chr='Chromosome', pos='Start_position', ref='Reference_Allele', alt='Tumor_Seq_Allele2', and Hugo='Hugo_Symbol' for downstream gene joins. Specify AnalCOSMICSigType ('SBS' or 'ID') so RNMF generates the correct mutation-type information matrix, and pass genome.build ('Ch37' or 'Ch38') backed by the Bioconductor BSgenome.Hsapiens.UCSC.hg19/hg38 libraries for trinucleotide context reconstruction. The parsing step must precede denovoNMF (signature extraction) or InverseNMF (fixed-signature sample contribution estimation), because denovoNMF and InverseNMF consume the count matrix, while samFisherSigs/samPerMutSigs/genePerMutSigs re-read the same MAF keyed by Hugo_Symbol to link signatures to gene mutations. Restrict analyses to coding regions when sequencing backgrounds are inconsistent, as the article did for the pooled 1073-sample ESCC cohort (508 WGS + 565 exon).

## Related tools

- **RNMF** (Provides SigsInput to import MAF/VCF somatic variants and build the SBS/DBS/ID count matrices used by denovoNMF, InverseNMF, and the sam*/genePerMutSigs association functions) — https://github.com/zhenzhang-li/RNMF
- **R** (Language in which RNMF is implemented and invoked as a library)
- **BSgenome.Hsapiens.UCSC.hg19** (Bioconductor genome build (Ch37) used to reconstruct mutation contexts from parsed variant coordinates)
- **BSgenome.Hsapiens.UCSC.hg38** (Alternative Bioconductor genome build (Ch38) for context reconstruction)

## Examples

```
library(RNMF); SigsInput(file='ESCC.exon.ALL.input.maf', Filetype='MAF', AnalCOSMICSigType='SBS', genome.build="Ch37", sample.id='Tumor_Sample_Barcode', chr='Chromosome', pos='Start_position', ref='Reference_Allele', alt='Tumor_Seq_Allele2')
```

## Evaluation signals

- SigsInput completes without errors and produces the SBS (and/or ID) count matrix plus originalGenomes, sampleNames, and subtypes objects consumable by denovoNMF
- Every parsed record retains a valid Tumor_Sample_Barcode and Hugo_Symbol key so later samPerMutSigs/genePerMutSigs calls with Mutfreq = 0.04, threshold = 0.06, Qvalue = 0.1 can join against the MAF
- Genome build passed to SigsInput (Ch37 vs Ch38) matches the BSgenome library installed; mismatched build yields wrong trinucleotide contexts
- Sample count in the parsed matrix matches the cohort size (e.g. 1073 ESCC samples for EXON1073, 508 for WGS508)
- Downstream denovoNMF run on the parsed matrices recovers the reported signature counts (12 SBS, 9 ID) with matched-signature cosine similarity ≥ 0.9

## Limitations

- Due to inconsistent sequencing backgrounds, only coding regions were uniformly analyzed, which introduces limitations on context reconstruction
- The framework relies on Bioconductor genome libraries (BSgenome.Hsapiens.UCSC.hg19/hg38), so unsupported builds cannot be parsed
- No software version numbers or installation instructions are stated for the framework in the discussion section

## Evidence

- [methods] somatic variants can be imported from a Variant Call Format (VCF) file or a Mutation Annotation Format (MAF) file: "somatic variants can be imported from a Variant Call Format (VCF) file or a Mutation Annotation Format (MAF) file"
- [methods] to acquire an information matrix of mutation types: "to acquire an information matrix of mutation types (single‐base substitutions [SBS], double‐base substitutions [DBS], and insertion and deletion mutation [ID])"
- [methods] Then it relies on the Bioconductor library, such as BS.genome.Hsapiens.UCSC.hg19 or BS.genome.Hsapiens.UCSC.hg38: "Then it relies on the Bioconductor library, such as BS.genome.Hsapiens.UCSC.hg19 or BS.genome.Hsapiens.UCSC.hg38"
- [readme] SigsInput(file, Filetype = 'MAF', AnalCOSMICSigType = 'SBS', genome.build = "Ch37", sample.id = 'Tumor_Sample_Barcode', chr = 'Chromosome', pos = 'Start_position', ref = 'Reference_Allele', alt = 'Tumor_Seq_Allele2'): "SigsInput(file, Filetype = 'MAF', AnalCOSMICSigType = 'SBS', genome.build = "Ch37", sample.id = 'Tumor_Sample_Barcode', chr = 'Chromosome', pos = 'Start_position', ref = 'Reference_Allele', alt ="
- [discussion] Due to the inconsistent sequencing background of the data, we uniformly analyzed the data of coding regions: "Due to the inconsistent sequencing background of the data, we uniformly analyzed the data of coding regions"
