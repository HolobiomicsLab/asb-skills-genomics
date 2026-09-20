---
name: cosmic-signature-matrix-annotation
description: Use when you need to assign COSMIC reference signatures (e.g. SBS1, SBS2,
  SBS13, SBS3, SBS17b, SBS22) to a fixed signature matrix P for InverseNMF refitting
  of a single sample or small cohort, rather than de novo extraction.
when_to_use_negative:
- When the goal is de novo signature extraction from a large cohort with unknown signature
  number — use RNMF's denovoNMF 5-step framework instead of fixing P to COSMIC signatures
- When no catalog-backed signature labels are needed and only an unsupervised decomposition
  of P and S is required
- When the genome build of the variant data does not match an available BSgenome library
  (hg19/hg38), since trinucleotide contexts cannot be correctly annotated
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: RNMF
  role: Provides InverseNMF to refit the contribution matrix S against a user-defined
    (COSMIC-annotated) signature matrix P, and SigsInput to build mutation count matrices
    from MAF/VCF
  repo: https://github.com/zhenzhang-li/RNMF
- name: BSgenome.Hsapiens.UCSC.hg19
  role: Bioconductor genome-build dependency used to construct the observed 96-context
    SBS count matrix V from MAF/VCF variants
- name: deconstructSigs
  role: Baseline refitting tool used to validate that COSMIC-annotated InverseNMF
    contribution scores are highly consistent
- name: MutationalPatterns
  role: Baseline refitting tool used to cross-check contribution scores and runtime
    against the COSMIC-annotated InverseNMF refit
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/cam4.4717
    title: A practical framework <scp>RNMF</scp> for exploring the association between
      mutational signatures and genes using gene cumulative contribution abundance
  source_content_hash: sha256:21f9a5c4ac8951eb757f15fc3b4badc95cccf7637411514c60dedeb767c8fb5a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhenzhang-li/RNMF
  tools:
  - BSgenome.Hsapiens.UCSC.hg19
  - MutationalPatterns
  - R
  - RNMF
  - deconstructSigs
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# cosmic-signature-matrix-annotation

## Summary

This skill annotates a user-defined signature matrix P with COSMIC reference mutational signatures (v3.2 - March 2021 or v2.0 - March 2015) so that RNMF's InverseNMF can refit the contribution matrix S for a small dataset or a single tumor sample against a fixed, catalog-backed signature set.

## When to use

Use when you need to assign COSMIC reference signatures (e.g. SBS1, SBS2, SBS13, SBS3, SBS17b, SBS22) to a fixed signature matrix P for InverseNMF refitting of a single sample or small cohort, rather than de novo extraction.

## When NOT to use

- When the goal is de novo signature extraction from a large cohort with unknown signature number — use RNMF's denovoNMF 5-step framework instead of fixing P to COSMIC signatures
- When no catalog-backed signature labels are needed and only an unsupervised decomposition of P and S is required
- When the genome build of the variant data does not match an available BSgenome library (hg19/hg38), since trinucleotide contexts cannot be correctly annotated

## Inputs

- COSMIC Mutational Signatures matrix (v3.2 - March 2021 and/or v2.0 - March 2015), e.g. SBS.Pmatrix.txt
- Somatic variant catalog in MAF or VCF format (e.g. ESCC.exon.ALL.input.maf)
- Bioconductor genome build library BSgenome.Hsapiens.UCSC.hg19 or BSgenome.Hsapiens.UCSC.hg38

## Outputs

- Annotated/fixed COSMIC signature matrix P (user-defined, e.g. SBS.Pmatrix.txt)
- Refitted sample contribution matrix S (e.g. SBS.Smatrix.txt) from InverseNMF
- Cosine similarity scores between reference and observed signatures

## How to apply

Retrieve the COSMIC reference signatures from https://cancer.sanger.ac.uk/cosmic/signatures, choosing version v3.2 (March 2021) or v2.0 (March 2015) as used by the paper, and restrict to the required signature type (e.g. SBS). Build the observed mutation count matrix V from a MAF or VCF file using the Bioconductor BSgenome.Hsapiens.UCSC.hg19 (or hg38) library with genome.build set to the matching build (e.g. 'Ch37'). Fix P as the user-defined COSMIC signature matrix and run InverseNMF's multiplicative update S' = S * P^T V / (P^T P S + gamma), iterating until S converges or up to the iteration cap. Use cosine similarity to determine the similarity between the extracted/reference mutational signatures and to compare refitted contributions against baselines such as deconstructSigs and MutationalPatterns.

## Related tools

- **RNMF** (Provides InverseNMF to refit the contribution matrix S against a user-defined (COSMIC-annotated) signature matrix P, and SigsInput to build mutation count matrices from MAF/VCF) — https://github.com/zhenzhang-li/RNMF
- **BSgenome.Hsapiens.UCSC.hg19** (Bioconductor genome-build dependency used to construct the observed 96-context SBS count matrix V from MAF/VCF variants)
- **deconstructSigs** (Baseline refitting tool used to validate that COSMIC-annotated InverseNMF contribution scores are highly consistent)
- **MutationalPatterns** (Baseline refitting tool used to cross-check contribution scores and runtime against the COSMIC-annotated InverseNMF refit)

## Examples

```
Sigstype = c('SBS3','SBS16','SBS18','New','SBS33','SBS13','SBS2','SBS5','SBS1','SBS17b','SBS22','SBS15')
```

## Evaluation signals

- Cosine similarity between the annotated COSMIC signatures and the observed/extracted signatures is computed and reported; similarity should be high for correctly assigned signatures
- Refitted S contribution scores are highly consistent with deconstructSigs and MutationalPatterns on the same input catalog
- The InverseNMF refit converges (or hits the stated iteration cap) with S' = S * P^T V / (P^T P S + gamma) updates
- Runtime on the same ESCC input is comparable to MutationalPatterns and ~12x faster than deconstructSigs (runtime_benchmark.csv ratios)
- The chosen genome.build string (e.g. 'Ch37') matches the BSgenome library used, so all variants receive correct 96-context trinucleotide labels

## Limitations

- RNMF's overall framework spends more time on de novo signature extraction than previously published software, due to the 'Re-updated the initial value' correction and 'build a solution space' processes (though InverseNMF refitting itself is fast)
- Due to inconsistent sequencing background, only coding regions were uniformly analyzed, which limits generalization of the annotated analyses
- Analyses depend on the COSMIC version chosen (v2.0 March 2015 vs v3.2 March 2021); the paper used both, so results may differ across versions

## Evidence

- [methods] COSMIC reference versioning: "We used COSMIC Mutational Signatures (v3.2 ‐ March 2021 and v2.0 ‐ March 2015)"
- [methods] InverseNMF refitting purpose: "we also developed a reversible method named InverseNMF, which can identify mutational signatures within a small data set or a single tumor sample"
- [intro] COSMIC catalog location: "the final referenced mutational signatures are archived in the catalog of cancer of somatic mutations in cancer (COSMIC, https://cancer.sanger.ac.uk/cosmic/signatures)"
- [methods] Genome-build dependency for annotation: "Then it relies on the Bioconductor library, such as BS.genome.Hsapiens.UCSC.hg19 or BS.genome.Hsapiens.UCSC.hg38"
- [methods] Cosine similarity criterion: "we use the cosine similarity to determine the similarity between two mutational signature A and B"
- [readme] README invocation with signature type and build: "SigsInput(file, Filetype = 'MAF', AnalCOSMICSigType = 'SBS', genome.build = "Ch37", sample.id = 'Tumor_Sample_Barcode'"
