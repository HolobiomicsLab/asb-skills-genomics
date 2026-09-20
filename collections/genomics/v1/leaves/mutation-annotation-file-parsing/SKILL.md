---
name: mutation-annotation-file-parsing
description: Use when the input is a somatic mutation dataset in MAF (or VCF) format
  — e.g. an ESCC exon-sequencing MAF like 'ESCC.exon.ALL.input.maf' — and it must
  be keyed on 'Tumor_Sample_Barcode'/'Hugo_Symbol' for RNMF signature or per-gene
  analyses.
when_to_use_negative:
- Input is already a signature exposure or 96-channel count matrix rather than a raw
  MAF/VCF mutation-level file — parsing is redundant
- Variants are germline or not annotated with the required MAF columns (Tumor_Sample_Barcode,
  Chromosome, Start_position, Reference_Allele, Tumor_Seq_Allele2, Hugo_Symbol)
- Analysis needs non-SBS contexts without specifying the right AnalCOSMICSigType —
  DBS/ID parsing requires choosing 'DBS' or 'ID' (ID also has ID.row83=TRUE handling)
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: RNMF
  role: R package whose functions (SigsInput, cumulativeCA, samPerMutSigs, genePerMutSigs)
    ingest MAF/VCF somatic variant files using explicit column mapping and produce
    SBS/DBS/ID mutation-type information matrices
  repo: https://github.com/zhenzhang-li/RNMF
- name: BSgenome.Hsapiens.UCSC.hg19
  role: Bioconductor genome library used by RNMF to interpret chromosome positions
    (Ch37 build) from parsed MAF records
- name: BSgenome.Hsapiens.UCSC.hg38
  role: Bioconductor genome library for Ch38 build when parsing MAF positions at genome.build='Ch38'
- name: R
  role: Language in which the MAF-parsing RNMF package is implemented and run
  repo: https://github.com/zhenzhang-li/RNMF
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/cam4.4717
    title: A practical framework <scp>RNMF</scp> for exploring the association between
      mutational signatures and genes using gene cumulative contribution abundance
  source_content_hash: sha256:ad28306c89e0bd7a9f77511bd57c43f0be98761e4a9e70abff0301e89959b3ca
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhenzhang-li/RNMF
  tools:
  - BSgenome.Hsapiens.UCSC.hg19
  - BSgenome.Hsapiens.UCSC.hg38
  - R
  - RNMF
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# mutation-annotation-file-parsing

## Summary

Import somatic variant data from a Mutation Annotation Format (MAF) file (or VCF) into the RNMF framework by explicitly mapping MAF columns — Tumor_Sample_Barcode, Chromosome, Start_position, Reference_Allele, Tumor_Seq_Allele2, Hugo_Symbol — so downstream mutational signature and gene-association analyses (SBS/DBS/ID counting, gene cumulative contribution abundance) can run on the parsed mutation records.

## When to use

Use when the input is a somatic mutation dataset in MAF (or VCF) format — e.g. an ESCC exon-sequencing MAF like 'ESCC.exon.ALL.input.maf' — and it must be keyed on 'Tumor_Sample_Barcode'/'Hugo_Symbol' for RNMF signature or per-gene analyses.

## When NOT to use

- Input is already a signature exposure or 96-channel count matrix rather than a raw MAF/VCF mutation-level file — parsing is redundant
- Variants are germline or not annotated with the required MAF columns (Tumor_Sample_Barcode, Chromosome, Start_position, Reference_Allele, Tumor_Seq_Allele2, Hugo_Symbol)
- Analysis needs non-SBS contexts without specifying the right AnalCOSMICSigType — DBS/ID parsing requires choosing 'DBS' or 'ID' (ID also has ID.row83=TRUE handling)

## Inputs

- MAF-format somatic mutation file (e.g. ESCC.exon.ALL.input.maf; non-silent filtered variant table such as Regular.ESCC.exon.WES.maf-input-nonsilent.txt)
- VCF file (alternative accepted input format)
- Column-name configuration: sample.id='Tumor_Sample_Barcode', chr='Chromosome', pos='Start_position', ref='Reference_Allele', alt='Tumor_Seq_Allele2', Hugo='Hugo_Symbol'
- Genome build specification (Ch37/Ch38 via BSgenome.Hsapiens.UCSC.hg19 or hg38)

## Outputs

- Parsed per-sample mutation records suitable for RNMF SBS/DBS/ID information-matrix generation
- originalGenomes / sampleNames / subtypes matrices used by denovoNMF
- Per-sample signature contribution (Sfile) and signature profile (Pfile) matrices keyed on the parsed MAF samples
- Per-gene cumulative contribution abundance tables (e.g. samplesResults/<sig>.SBS.geneCumulativeContributionAbundance.txt)

## How to apply

Call RNMF functions that accept Filetype='MAF' and pass the MAF column names explicitly so the parser can locate sample, position, and allele fields: sample.id='Tumor_Sample_Barcode', chr='Chromosome', pos='Start_position', ref='Reference_Allele', alt='Tumor_Seq_Allele2', Hugo='Hugo_Symbol'. For initial ingestion into signature analysis, use SigsInput with Filetype='MAF', AnalCOSMICSigType='SBS' (or 'DBS'/'ID'), and genome.build='Ch37' (or Ch38), relying on the Bioconductor genomes BSgenome.Hsapiens.UCSC.hg19/hg38 for context. For association stages, feed the non-silent mutation table (e.g. 'Regular.ESCC.exon.WES.maf-input-nonsilent.txt') as the maffile argument to cumulativeCA and to samPerMutSigs/genePerMutSigs, pairing it with the S-matrix (Sfile), P-matrix (Pfile), and per-gene abundance file (Gfile). The rationale is that RNMF works directly from the mutation dataset rather than requiring precomputed 96-channel (or 78-DBS/153-ID) count matrices, so correct column mapping and genome build are the critical correctness decisions.

## Related tools

- **RNMF** (R package whose functions (SigsInput, cumulativeCA, samPerMutSigs, genePerMutSigs) ingest MAF/VCF somatic variant files using explicit column mapping and produce SBS/DBS/ID mutation-type information matrices) — https://github.com/zhenzhang-li/RNMF
- **BSgenome.Hsapiens.UCSC.hg19** (Bioconductor genome library used by RNMF to interpret chromosome positions (Ch37 build) from parsed MAF records)
- **BSgenome.Hsapiens.UCSC.hg38** (Bioconductor genome library for Ch38 build when parsing MAF positions at genome.build='Ch38')
- **R** (Language in which the MAF-parsing RNMF package is implemented and run) — https://github.com/zhenzhang-li/RNMF

## Examples

```
SigsInput(file='ESCC.exon.ALL.input.maf', Filetype='MAF', AnalCOSMICSigType='SBS', genome.build="Ch37", sample.id='Tumor_Sample_Barcode', chr='Chromosome', pos='Start_position', ref='Reference_Allele', alt='Tumor_Seq_Allele2')
```

## Evaluation signals

- The parser returns non-empty originalGenomes/sampleNames/subtypes objects from the MAF and denovoNMF runs without key errors, confirming Tumor_Sample_Barcode/Chromosome/Start_position/Reference_Allele/Tumor_Seq_Allele2 columns were resolved
- Sample identifiers in the parsed output match the sample.id field values ('Tumor_Sample_Barcode') and can be merged with a Groupfile keyed on the same IDs (as required to join TP53 R248Q status with G2/G4 survival groups)
- Only non-silent variants from the specified coding-region table (e.g. Regular.ESCC.exon.WES.maf-input-nonsilent.txt) are reflected in per-gene outputs such as the geneCumulativeContributionAbundance.txt files
- Genome build used for parsing (Ch37 vs Ch38) is consistent with the chosen BSgenome.Hsapiens.UCSC.hg19/hg38 library, and SBS/DBS/ID counts are produced for all three mutation classes as RNMF supports

## Limitations

- Inconsistent sequencing background across samples forced uniform analysis of coding regions only, which limits what MAF-derived signals can capture
- The framework spends more time on de novo signature extraction than previously published software, partly due to initial-value correction and solution-space building after MAF ingestion
- Only coding regions were uniformly analyzed for the 1073-ESCC cohort (508 WGS, 565 exon sequencing), so non-coding variants in a MAF may not contribute

## Evidence

- [methods] somatic variants can be imported from a Variant Call Format (VCF) file or a Mutation Annotation Format (MAF) file: "somatic variants can be imported from a Variant Call Format (VCF) file or a Mutation Annotation Format (MAF) file"
- [readme] sample.id='Tumor_Sample_Barcode', chr='Chromosome', pos='Start_position', ref='Reference_Allele', alt='Tumor_Seq_Allele2': "sample.id = 'Tumor_Sample_Barcode', chr = 'Chromosome', pos = 'Start_position', ref = 'Reference_Allele', alt = 'Tumor_Seq_Allele2')"
- [readme] SigsInput(file, Filetype = 'MAF', AnalCOSMICSigType = 'SBS', genome.build = "Ch37", ...): "SigsInput(file, Filetype = 'MAF', AnalCOSMICSigType = 'SBS', genome.build = "Ch37""
- [methods] Then it relies on the Bioconductor library, such as BS.genome.Hsapiens.UCSC.hg19 or BS.genome.Hsapiens.UCSC.hg38: "Then it relies on the Bioconductor library, such as BS.genome.Hsapiens.UCSC.hg19 or BS.genome.Hsapiens.UCSC.hg38"
- [methods] our designed RNMF software provides the ability to obtain the count matrix of SBS, DBS, and ID directly from mutation data sets: "our designed RNMF software provides the ability to obtain the count matrix of SBS, DBS, and ID directly from mutation data sets"
