---
name: f1-score-computation-across-annotation-levels
description: Use when comparing predicted genome annotations (GTF) from multiple gene-prediction
  pipelines against a reference GTF and you need multi-level precision/recall/F1 statistics
  to decide which pipeline performed .
when_to_use_negative:
- Input annotations are raw GFF3 that have not been converted with agat_convert_sp_gff2gtf.pl
  — format issues will corrupt the comparison
- No reference annotation GTF exists for the organism, so reference-based precision/recall/F1
  statistics cannot be computed
- The goal is ab initio prediction quality with no transcriptome assembly input, rather
  than benchmarking assembly-based annotation improvements
edam_operation: http://edamontology.org/operation_0282
tools:
- name: gffcompare
  role: Compares each pipeline's GTF against the reference annotation to produce the
    .tmap transcript map and precision/recall/F1 statistics
- name: StringTie
  role: Merges comparator GTFs with the reference via 'stringtie --merge -G' so only
    StringTie-tagged transcripts are retained for fair comparison
- name: AGAT
  role: Pre-processes braker.gtf/tsebra.gtf with agat_convert_sp_gff2gtf.pl into comparable
    GTF format
  repo: https://github.com/NBISweden/AGAT
- name: IsoSeqSim
  role: Simulates PacBio long reads to generate additional StringTie transcriptome
    inputs for benchmarking under simulated conditions
  repo: https://github.com/yunhaowang/IsoSeqSim
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giac099
    title: 'annotate_my_genomes: an easy-to-use pipeline to improve genome annotation
      and uncover neglected genes by hybrid RNA sequencing'
  source_content_hash: sha256:73517cf5119fb4c6e80fc8739ddb2d6caba3b24d37bf40ba80e3c67bf3124cd2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NBISweden/AGAT
  tools:
  - AGAT
  - AUGUSTUS
  - BRAKER1
  - BRAKER2
  - IsoSeqSim
  - StringTie
  - TSEBRA
  - gffcompare
  edam_operation: http://edamontology.org/operation_0282
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# f1-score-computation-across-annotation-levels

## Summary

Benchmark gene-prediction accuracy by running gffcompare on each pipeline's GTF (annotate_my_genomes, BRAKER1/2, TSEBRA, AUGUSTUS) against the corresponding reference annotation, extracting precision, recall, and harmonic-mean F1-score at base, exon, intron, intron-chain, transcript, and locus levels.

## When to use

Use when comparing predicted genome annotations (GTF) from multiple gene-prediction pipelines against a reference GTF and you need multi-level precision/recall/F1 statistics to decide which pipeline performed best.

## When NOT to use

- Input annotations are raw GFF3 that have not been converted with agat_convert_sp_gff2gtf.pl — format issues will corrupt the comparison
- No reference annotation GTF exists for the organism, so reference-based precision/recall/F1 statistics cannot be computed
- The goal is ab initio prediction quality with no transcriptome assembly input, rather than benchmarking assembly-based annotation improvements

## Inputs

- Predicted annotation GTF files from annotate_my_genomes, BRAKER1, BRAKER2, TSEBRA, and AUGUSTUS
- Reference coding GTF annotations (e.g., galGal6_ncbiRefSeq.gtf)
- braker_fixed.gtf / tsebra_fixed.gtf after AGAT GFF-to-GTF conversion
- StringTie-assembled GTFs from de-novo, StringTie-mix (merged PacBio+Illumina BAM), and IsoSeqSim-simulated inputs

## Outputs

- gffcompare.tmap transcript map per pipeline/genome pair
- gffcompare accuracy statistics with precision, recall, and F1 at base, exon, intron, intron-chain, transcript, and locus levels
- Summary table across genomes, pipelines, and transcriptome conditions (reproducing Supplementary Table 2)

## How to apply

For each benchmarked genome (galGal6, hg38, mm10, danRer11, ce11), run gffcompare on each pipeline's GTF against the matched reference coding GTF to obtain the transcript map (gffcompare.tmap) and accuracy statistics. Before comparison, pre-process braker.gtf/tsebra.gtf with agat_convert_sp_gff2gtf.pl, merge with the reference GTF using 'stringtie --merge -G', and retain only StringTie-tagged transcripts with grep to avoid inflating sensitivity. Parse the gffcompare stats to compute precision, recall, and the harmonic-mean F1-score at six levels: base, exon, intron, intron-chain, transcript, and locus. Compile results into a summary table across all genomes, pipelines, and transcriptome conditions (StringTie de-novo, StringTie-mix from merged PacBio+Illumina BAMs, and simulated PacBio inputs from IsoSeqSim).

## Related tools

- **gffcompare** (Compares each pipeline's GTF against the reference annotation to produce the .tmap transcript map and precision/recall/F1 statistics)
- **StringTie** (Merges comparator GTFs with the reference via 'stringtie --merge -G' so only StringTie-tagged transcripts are retained for fair comparison)
- **AGAT** (Pre-processes braker.gtf/tsebra.gtf with agat_convert_sp_gff2gtf.pl into comparable GTF format) — https://github.com/NBISweden/AGAT
- **IsoSeqSim** (Simulates PacBio long reads to generate additional StringTie transcriptome inputs for benchmarking under simulated conditions) — https://github.com/yunhaowang/IsoSeqSim

## Examples

```
stringtie --merge -G galGal6_ncbiRefSeq.gtf braker_fixed.gtf -o braker_merged.gtf
```

## Evaluation signals

- A gffcompare.tmap file and gffcompare stats file are produced for every pipeline-genome pair
- F1-score equals the harmonic mean of the reported precision and recall at each of the six levels (base, exon, intron, intron-chain, transcript, locus)
- The compiled F1 table reproduces Supplementary Table 2 of the giac099 paper
- Validation criterion: annotate_my_genomes F1-scores are equal or better than BRAKER1/2, TSEBRA, and AUGUSTUS on StringTie de-novo assemblies
- Only StringTie-tagged transcripts remain after the 'stringtie --merge -G' + grep step, confirming the comparator output was correctly filtered before comparison

## Limitations

- Precision, recall, and F1 values are deferred to Supplementary Table 2 and are not reproduced in the main text, requiring supplementary access for validation
- The comparison is reference-dependent: results reflect agreement with NCBI RefSeq annotations for galGal6, hg38, mm10, danRer11, and ce11 and may not transfer to organisms lacking curated annotations
- Benchmark findings are stated for StringTie assemblies; the method requires an assembled genome and an NCBI GTF annotation, so conclusions do not extend to organisms without these resources

## Evidence

- [supplementary] Precision and recall values were obtained by comparing base, exon, intron, intron-chain, transcript, and locus level: "Precision and recall values were obtained by comparing base, exon, intron, intron-chain, transcript, and locus level"
- [discussion] F1 scores obtained via gffcompare, deferred to Supplementary Table 2: "The precision, recall, and their harmonic mean (F1-score) were obtained by using gffcompare and are available in Supplementary Table 2"
- [discussion] Validation finding: annotate_my_genomes equal or better by F1: "According to the F1-score, using StringTie de-novo assemblies, our method performed equal or better than existing pipelines"
- [methods] Concrete merge command used before gffcompare benchmarking: "stringtie --merge -G galGal6_ncbiRefSeq.gtf braker_fixed.gtf -o braker_merged.gtf"
- [other] Benchmarking workflow step describing gffcompare tmap output: [evidence span withheld — anchor status: not_found]
