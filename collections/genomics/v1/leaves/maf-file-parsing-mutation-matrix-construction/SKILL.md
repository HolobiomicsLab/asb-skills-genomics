---
name: maf-file-parsing-mutation-matrix-construction
description: Use when somatic mutations are provided as a MAF or VCF file and a gene-by-sample
  mutation type information matrix (SBS, DBS, ID) must be built for signature-gene
  association analysis with RNMF.
when_to_use_negative:
- Input is already a gene-by-sample mutation count matrix or a processed signature
  matrix (P/S) rather than raw variant calls
- Variant data is not in MAF or VCF format and cannot be trivially converted, since
  RNMF imports somatic variants from these two file types
- The goal is de novo signature extraction alone with no gene-level (Hugo_Symbol)
  annotation in the variant file
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3673
tools:
- name: RNMF
  role: R package that parses MAF/VCF somatic variants and builds the gene-by-sample
    mutation type information matrix used by cumulativeCA
  repo: https://github.com/zhenzhang-li/RNMF
- name: BSgenome.Hsapiens.UCSC.hg19
  role: Bioconductor genome library used to resolve genome build ('Ch37') during mutation
    type classification
- name: BSgenome.Hsapiens.UCSC.hg38
  role: Bioconductor genome library for the alternative genome build during parsing
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/cam4.4717
    title: A practical framework <scp>RNMF</scp> for exploring the association between
      mutational signatures and genes using gene cumulative contribution abundance
  source_content_hash: sha256:11f7e83dce1afa9f8a8e0d7604a51f9c1a92e9dfcf87a7625bd1990cdfe9f02e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhenzhang-li/RNMF
  tools:
  - BSgenome.Hsapiens.UCSC.hg19
  - BSgenome.Hsapiens.UCSC.hg38
  - R
  - RNMF
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# maf-file-parsing-mutation-matrix-construction

## Summary

Parses somatic variants from MAF (or VCF) files and constructs gene-by-sample mutation count matrices within the RNMF R package, providing the input matrices needed for downstream cumulative contribution abundance (CCA) analysis linking mutational signatures to genes.

## When to use

Use when somatic mutations are provided as a MAF or VCF file and a gene-by-sample mutation type information matrix (SBS, DBS, ID) must be built for signature-gene association analysis with RNMF.

## When NOT to use

- Input is already a gene-by-sample mutation count matrix or a processed signature matrix (P/S) rather than raw variant calls
- Variant data is not in MAF or VCF format and cannot be trivially converted, since RNMF imports somatic variants from these two file types
- The goal is de novo signature extraction alone with no gene-level (Hugo_Symbol) annotation in the variant file

## Inputs

- MAF file (e.g. ESCC.exon.ALL.input.maf) with Tumor_Sample_Barcode, Chromosome, Start_position, Reference_Allele, Tumor_Seq_Allele2, Hugo_Symbol columns
- VCF file (alternative to MAF)
- Genome build specification (BSgenome.Hsapiens.UCSC.hg19 or hg38, e.g. 'Ch37')

## Outputs

- Gene-by-sample mutation count matrix of mutation types (SBS, DBS, ID)
- originalGenomes, sampleNames, and subtypes objects for denovoNMF
- Input matrices for cumulativeCA-derived geneCumulativeContributionAbundance tables

## How to apply

Use RNMF's SigsInput or cumulativeCA entry points, supplying the MAF file together with column mappings sample.id='Tumor_Sample_Barcode', chr='Chromosome', pos='Start_position', ref='Reference_Allele', alt='Tumor_Seq_Allele2', and Hugo='Hugo_Symbol', plus Filetype='MAF', AnalCOSMICSigType (e.g. 'SBS') and genome.build ('Ch37'/'hg19' or 'hg38' via BSgenome libraries). RNMF acquires an information matrix of mutation types (SBS, DBS, ID) directly from the mutation data set; for ID calls set ID.mc.cores and ID.row83=TRUE. The resulting matrices feed the CCA model, where per-gene impact factors rho_mgn = N_mgn / sum_i N_min are combined with signature-specific probabilities rho_s,mn = P_ms*S_sn to compute cumulative (alpha_s,g) and relative (eta_s,g) abundance, the latter with gene-length normalization to correct for gene-length bias. Verify column names exist in the MAF header before running, since mappings are positional string matches.

## Related tools

- **RNMF** (R package that parses MAF/VCF somatic variants and builds the gene-by-sample mutation type information matrix used by cumulativeCA) — https://github.com/zhenzhang-li/RNMF
- **BSgenome.Hsapiens.UCSC.hg19** (Bioconductor genome library used to resolve genome build ('Ch37') during mutation type classification)
- **BSgenome.Hsapiens.UCSC.hg38** (Bioconductor genome library for the alternative genome build during parsing)

## Examples

```
SigsInput(file='ESCC.exon.ALL.input.maf', Filetype='MAF', AnalCOSMICSigType='SBS', genome.build="Ch37", sample.id='Tumor_Sample_Barcode', chr='Chromosome', pos='Start_position', ref='Reference_Allele', alt='Tumor_Seq_Allele2')
```

## Evaluation signals

- The MAF column mappings (sample.id, chr, pos, ref, alt, Hugo) match the file header; a mismatch produces empty or failed matrix construction
- The constructed matrix covers SBS, DBS, and ID categories as produced by RNMF's 'count matrix of SBS, DBS, and ID directly from mutation data sets'
- Row totals per tumor equal the number of nonsilent mutations for that sample (e.g. after filtering to nonsilent variants as in 'Regular.ESCC.exon.WES.maf-input-nonsilent.txt')
- Downstream cumulativeCA runs without error and produces per-signature geneCumulativeContributionAbundance tables containing alpha_s,g and eta_s,g values
- The number of unique samples in the matrix equals the number of unique Tumor_Sample_Barcode values in the MAF

## Limitations

- Due to the inconsistent sequencing background of the data, only coding regions were uniformly analyzed, which introduces limitations
- No software version numbers, installation instructions, or code availability details are stated for the framework
- Column mapping strings must exactly match MAF headers; nonstandard MAF column names require remapping before parsing

## Evidence

- [methods] somatic variants can be imported from a Variant Call Format (VCF) file or a Mutation Annotation Format (MAF) file: "somatic variants can be imported from a Variant Call Format (VCF) file or a Mutation Annotation Format (MAF) file"
- [methods] Mutation type information matrix generation (SBS, DBS, ID): "to acquire an information matrix of mutation types (single‐base substitutions [SBS], double‐base substitutions [DBS], and insertion and deletion mutation [ID])"
- [methods] our designed RNMF software provides the ability to obtain the count matrix of SBS, DBS, and ID directly from mutation data sets: "our designed RNMF software provides the ability to obtain the count matrix of SBS, DBS, and ID directly from mutation data sets"
- [readme] README cumulativeCA call with MAF column mappings: "sample.id = 'Tumor_Sample_Barcode', chr = 'Chromosome', pos = 'Start_position', ref = 'Reference_Allele', alt = 'Tumor_Seq_Allele2', Hugo = 'Hugo_Symbol'"
- [readme] README SigsInput example invocation: "SigsInput(file, Filetype = 'MAF', AnalCOSMICSigType = 'SBS', genome.build = "Ch37", sample.id = 'Tumor_Sample_Barcode'"
