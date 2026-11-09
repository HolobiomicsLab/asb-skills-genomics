---
name: gffcompare-transcript-precision-recall-benchmarking
description: Use when comparing two or more predicted/merged annotation GTFs (e.g.
  StringTie-based, BRAKER1/2, TSEBRA, AUGUSTUS) against a reference annotation and
  you need level-resolved accuracy metrics to rank pipelines.
when_to_use_negative:
- The pipeline under evaluation has no reference GTF available to compare against,
  since precision/recall cannot be computed without a ground-truth annotation
- You only need transcript abundance quantification rather than structural annotation
  accuracy — gffcompare evaluates structure, not expression
- Annotations are already reconciled/merged and only need visualization (use pyGenomeTracks
  instead)
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
tools:
- name: gffcompare
  role: Compares each pipeline's GTF against the reference annotation and computes
    precision, recall, and F1-score plus the transcript map (gffcompare.tmap)
- name: AGAT
  role: Pre-processes braker.gtf/tsebra.gtf to GTF-compatible format via agat_convert_sp_gff2gtf.pl
    before comparison
  repo: https://github.com/NBISweden/AGAT
- name: StringTie
  role: Merges candidate annotations with the reference GTF via 'stringtie --merge
    -G' and assembles the de-novo/transcriptome GTFs being benchmarked
- name: IsoSeqSim
  role: Simulates PacBio long reads so StringTie-assembled transcriptome inputs can
    be generated for each benchmarked genome
  repo: https://github.com/yunhaowang/IsoSeqSim
- name: annotate_my_genomes
  role: Candidate pipeline whose annotation output is benchmarked with gffcompare
    against BRAKER1/2, TSEBRA, and AUGUSTUS
  repo: https://github.com/cfarkas/annotate_my_genomes
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giac099
    title: 'annotate_my_genomes: an easy-to-use pipeline to improve genome annotation
      and uncover neglected genes by hybrid RNA sequencing'
  source_content_hash: sha256:446d8fb7144d06c551b91faf4fafaccf0b262dace5846b5c38eb59a80fbf0328
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
  - annotate_my_genomes
  - gffcompare
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# gffcompare-transcript-precision-recall-benchmarking

## Summary

Benchmark gene/transcript annotations by running gffcompare against a reference GTF and extracting precision, recall, and their harmonic mean (F1-score) at base, exon, intron, intron-chain, transcript, and locus levels, as used to show annotate_my_genomes surpasses BRAKER1/2 and TSEBRA on StringTie assemblies.

## When to use

Use when comparing two or more predicted/merged annotation GTFs (e.g. StringTie-based, BRAKER1/2, TSEBRA, AUGUSTUS) against a reference annotation and you need level-resolved accuracy metrics to rank pipelines.

## When NOT to use

- The pipeline under evaluation has no reference GTF available to compare against, since precision/recall cannot be computed without a ground-truth annotation
- You only need transcript abundance quantification rather than structural annotation accuracy — gffcompare evaluates structure, not expression
- Annotations are already reconciled/merged and only need visualization (use pyGenomeTracks instead)

## Inputs

- Reference genome annotation in GTF format (e.g. galGal6_ncbiRefSeq.gtf)
- Candidate pipeline annotation GTFs (StringTie assemblies, annotate_my_genomes output, braker.gtf, tsebra.gtf, AUGUSTUS output)
- AGAT-converted GTF versions of braker.gtf/tsebra.gtf
- StringTie-merged GTFs (stringtie --merge -G reference braker_fixed.gtf)
- Simulated transcriptome inputs from IsoSeqSim reads assembled with StringTie

## Outputs

- gffcompare.tmap transcript map per pipeline/genome
- gffcompare accuracy statistics with precision, recall, and F1-score at base, exon, intron, intron-chain, transcript, and locus levels
- Summary table of F1-scores across genomes, pipelines, and transcriptome conditions (analogous to Supplementary Table 2)

## How to apply

For each genome (galGal6, hg38, mm10, danRer11, ce11), run gffcompare with the reference GTF (e.g. galGal6_ncbiRefSeq.gtf) as the -r reference against each candidate annotation GTF, producing a transcript map (gffcompare.tmap) and accuracy statistics. Candidate GTFs from BRAKER/TSEBRA must first be converted to GTF with agat_convert_sp_gff2gtf.pl and optionally merged with the reference via 'stringtie --merge -G', retaining only StringTie-tagged transcripts with grep. Parse the gffcompare stats to extract precision, recall, and their harmonic mean (F1-score) at six levels: base, exon, intron, intron-chain, transcript, and locus. Compile metrics into a summary table across genomes, pipelines, and transcriptome conditions (de-novo vs simulated long-read inputs from IsoSeqSim + StringTie) to reproduce the ranking (annotate_my_genomes equal or better than BRAKER1/2 and TSEBRA).

## Related tools

- **gffcompare** (Compares each pipeline's GTF against the reference annotation and computes precision, recall, and F1-score plus the transcript map (gffcompare.tmap))
- **AGAT** (Pre-processes braker.gtf/tsebra.gtf to GTF-compatible format via agat_convert_sp_gff2gtf.pl before comparison) — https://github.com/NBISweden/AGAT
- **StringTie** (Merges candidate annotations with the reference GTF via 'stringtie --merge -G' and assembles the de-novo/transcriptome GTFs being benchmarked)
- **IsoSeqSim** (Simulates PacBio long reads so StringTie-assembled transcriptome inputs can be generated for each benchmarked genome) — https://github.com/yunhaowang/IsoSeqSim
- **annotate_my_genomes** (Candidate pipeline whose annotation output is benchmarked with gffcompare against BRAKER1/2, TSEBRA, and AUGUSTUS) — https://github.com/cfarkas/annotate_my_genomes

## Examples

```
gffcompare -r galGal6_ncbiRefSeq.gtf braker_merged.gtf -o braker_vs_galGal6
```

## Evaluation signals

- Compiled F1-scores reproduce the values in Supplementary Table 2 for the corresponding genome/pipeline/transcriptome combination
- annotate_my_genomes F1-score is equal or better than BRAKER1, BRAKER2, and TSEBRA on StringTie de-novo assemblies
- Metrics are present at all six levels (base, exon, intron, intron-chain, transcript, locus) for every pipeline-genome pair
- A gffcompare.tmap file exists for each candidate GTF, confirming the comparison ran against the correct reference
- Comparator GTFs were pre-processed (AGAT conversion, stringtie --merge, grep for StringTie-tagged transcripts) before comparison, so no format-driven false mismatches occur

## Limitations

- Precision and recall values are deferred to Supplementary Table 2 and are not shown in the main text, so exact numeric verification requires the SI
- The benchmark depends on StringTie GTF assemblies as input; results may not transfer to other assembly engines or to organisms lacking an assembled genome and an NCBI GTF annotation
- Results are reported only for the benchmarked vertebrate genomes (galGal6, hg38, mm10, danRer11, ce11) and their matched reference annotations

## Evidence

- [discussion] The precision, recall, and their harmonic mean (F1-score) were obtained by using gffcompare and are available in Supplementary Table 2: "The precision, recall, and their harmonic mean (F1-score) were obtained by using gffcompare and are available in Supplementary Table 2"
- [discussion] Precision and recall values were obtained by comparing base, exon, intron, intron-chain, transcript, and locus level: "Precision and recall values were obtained by comparing base, exon, intron, intron-chain, transcript, and locus level"
- [discussion] According to the F1-score, using StringTie de-novo assemblies, our method performed equal or better than existing pipelines: "According to the F1-score, using StringTie de-novo assemblies, our method performed equal or better than existing pipelines"
- [methods] stringtie --merge -G galGal6_ncbiRefSeq.gtf braker_fixed.gtf -o braker_merged.gtf: "stringtie --merge -G galGal6_ncbiRefSeq.gtf braker_fixed.gtf -o braker_merged.gtf"
