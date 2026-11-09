---
name: ab-initio-gene-prediction-benchmarking
description: Use when you need to quantitatively compare genome annotation methods
  (e.g., StringTie assembly-based annotation vs ab-initio/evidence-driven predictors
  like BRAKER1/2, TSEBRA, AUGUSTUS) on genomes with known reference GTFs.
when_to_use_negative:
- The organism lacks an assembled genome and an NCBI-available GTF annotation, since
  the pipeline requires both (e.g., no reference GTF means no ground truth for gffcompare).
- You need validated protein-level evidence for novel isoforms rather than structural
  annotation benchmarking — the F1 scores do not confirm protein products.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3170
tools:
- name: gffcompare
  role: Computes precision, recall, and F1-score against the reference GTF and produces
    the gffcompare.tmap transcript map
- name: StringTie
  role: Assembles transcripts from BAM alignments (settings -p 1 -j 2 -c 2 -v -a 4)
    and merges candidate GTFs with the reference via 'stringtie --merge -G'
- name: BRAKER1
  role: RNA-seq-based comparator gene prediction pipeline (AUGUSTUS + GeneMark-ET)
- name: BRAKER2
  role: Protein/RNA-homology comparator gene prediction pipeline
  repo: https://github.com/Gaius-Augustus/BRAKER
- name: TSEBRA
  role: Transcript selector combining BRAKER predictions as comparator
  repo: https://github.com/Gaius-Augustus/TSEBRA
- name: AUGUSTUS
  role: Ab-initio gene predictor used as comparator
  repo: https://github.com/Gaius-Augustus/Augustus
- name: AGAT
  role: Pre-processing of braker.gtf/tsebra.gtf via agat_convert_sp_gff2gtf.pl before
    merging
  repo: https://github.com/NBISweden/AGAT
- name: IsoSeqSim
  role: Simulates PacBio long reads to generate transcriptome inputs for each genome
  repo: https://github.com/yunhaowang/IsoSeqSim
- name: annotate_my_genomes
  role: Method under test; annotate-my-genomes.nf and add-ncbi-annotation.nf produce
    candidate annotations from StringTie assemblies
  repo: https://github.com/cfarkas/annotate_my_genomes
- name: genome-download
  role: Nextflow program that retrieves reference genomes and matched coding GTF annotations
  repo: https://github.com/cfarkas/annotate_my_genomes
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giac099
    title: 'annotate_my_genomes: an easy-to-use pipeline to improve genome annotation
      and uncover neglected genes by hybrid RNA sequencing'
  source_content_hash: sha256:93731db22e5694bb3a12723642de9377d98881506c3e69f37f2fe1495f459844
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Gaius-Augustus/BRAKER
  tools:
  - AGAT
  - AUGUSTUS
  - BRAKER1
  - BRAKER2
  - IsoSeqSim
  - StringTie
  - TSEBRA
  - annotate_my_genomes
  - genome-download
  - gffcompare
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# ab-initio-gene-prediction-benchmarking

## Summary

Benchmark gene prediction pipelines (annotate_my_genomes vs BRAKER1, BRAKER2, TSEBRA, AUGUSTUS) by comparing each pipeline's GTF against a reference annotation with gffcompute and compiling precision, recall, and harmonic-mean F1-score at six annotation levels.

## When to use

Use when you need to quantitatively compare genome annotation methods (e.g., StringTie assembly-based annotation vs ab-initio/evidence-driven predictors like BRAKER1/2, TSEBRA, AUGUSTUS) on genomes with known reference GTFs.

## When NOT to use

- The organism lacks an assembled genome and an NCBI-available GTF annotation, since the pipeline requires both (e.g., no reference GTF means no ground truth for gffcompare).
- You need validated protein-level evidence for novel isoforms rather than structural annotation benchmarking — the F1 scores do not confirm protein products.

## Inputs

- Reference genome assemblies (e.g., galGal6 GCA_000002315.5, hg38, mm10, danRer11, ce11)
- Matched reference coding GTF annotations (e.g., galGal6_ncbiRefSeq.gtf)
- StringTie de-novo GTF assemblies (parameters -p 1 -j 2 -c 2 -v -a 4)
- StringTie-mix assemblies from merged PacBio+Illumina BAM alignments
- braker.gtf / tsebra.gtf comparator outputs
- Simulated PacBio reads from IsoSeqSim for transcriptome inputs

## Outputs

- gffcompare.tmap transcript map per pipeline/genome pair
- gffcompare accuracy statistics (precision, recall, F1 at base, exon, intron, intron-chain, transcript, locus levels)
- Compiled summary F1-score table across all genomes, pipelines, and transcriptome conditions (reproducing Supplementary Table 2)

## How to apply

Download reference genome assemblies and matched coding GTF annotations (galGal6, hg38, mm10, danRer11, ce11) using the genome-download program. Generate candidate annotations: run annotate-my-genomes / add-ncbi-annotation on StringTie de-novo assemblies (settings -p 1 -j 2 -c 2 -v -a 4) and on StringTie-mix assemblies from merged PacBio+Illumina BAM alignments; run BRAKER1, BRAKER2, TSEBRA, and AUGUSTUS on the same genomes. Pre-process braker.gtf/tsebra.gtf with agat_convert_sp_gff2gtf.pl, merge with the reference GTF via 'stringtie --merge -G', and retain only StringTie-tagged transcripts with grep. Simulate PacBio long reads with IsoSeqSim and assemble with StringTie to obtain simulated transcriptome inputs. Run gffcompare on each pipeline's GTF against the reference to obtain the transcript map (gffcompare.tmap) and accuracy statistics, then parse precision, recall, and F1 at base, exon, intron, intron-chain, transcript, and locus levels.

## Related tools

- **gffcompare** (Computes precision, recall, and F1-score against the reference GTF and produces the gffcompare.tmap transcript map)
- **StringTie** (Assembles transcripts from BAM alignments (settings -p 1 -j 2 -c 2 -v -a 4) and merges candidate GTFs with the reference via 'stringtie --merge -G')
- **BRAKER1** (RNA-seq-based comparator gene prediction pipeline (AUGUSTUS + GeneMark-ET))
- **BRAKER2** (Protein/RNA-homology comparator gene prediction pipeline) — https://github.com/Gaius-Augustus/BRAKER
- **TSEBRA** (Transcript selector combining BRAKER predictions as comparator) — https://github.com/Gaius-Augustus/TSEBRA
- **AUGUSTUS** (Ab-initio gene predictor used as comparator) — https://github.com/Gaius-Augustus/Augustus
- **AGAT** (Pre-processing of braker.gtf/tsebra.gtf via agat_convert_sp_gff2gtf.pl before merging) — https://github.com/NBISweden/AGAT
- **IsoSeqSim** (Simulates PacBio long reads to generate transcriptome inputs for each genome) — https://github.com/yunhaowang/IsoSeqSim
- **annotate_my_genomes** (Method under test; annotate-my-genomes.nf and add-ncbi-annotation.nf produce candidate annotations from StringTie assemblies) — https://github.com/cfarkas/annotate_my_genomes
- **genome-download** (Nextflow program that retrieves reference genomes and matched coding GTF annotations) — https://github.com/cfarkas/annotate_my_genomes

## Examples

```
stringtie --merge -G galGal6_ncbiRefSeq.gtf braker_fixed.gtf -o braker_merged.gtf && gffcompare -r galGal6_ncbiRefSeq.gtf -o gffcompare_out braker_merged.gtf
```

## Evaluation signals

- Compiled F1-score summary table must reproduce Supplementary Table 2 of the giac099 paper across all genomes, pipelines, and transcriptome conditions.
- Validation criterion: annotate_my_genomes performed equal or better than existing pipelines by F1-score when using StringTie de-novo assemblies.
- gffcompare stats must yield paired precision/recall values at all six levels (base, exon, intron, intron-chain, transcript, locus) for every pipeline/genome combination.
- Comparator GTFs must be pre-processed with agat_convert_sp_gff2gtf.pl and, after 'stringtie --merge -G' with the reference GTF, only StringTie-tagged transcripts should be retained by grep.
- gffcompare.tmap must be produced for each pipeline's GTF against its corresponding reference annotation.

## Limitations

- Precision, recall, and F1 values are deferred to Supplementary Table 2 rather than shown in the main text, so independent verification requires the supplementary material.
- The F1 benchmarking is structural only; protein products of predicted novel isoforms (e.g., SSPO isoforms) remain unvalidated.
- BRAKER README notes the tool family is deprecated in favor of BRAKER4, so version drift may affect reproduction of comparator results.

## Evidence

- [results] Using StringTie GTF assembly as input, the annotate_my_genomes method tends to improve current genome annotation, surpassing the performances of BRAKER1, BRAKER2, and TSEBRA.: [evidence span withheld — anchor status: not_found]
- [discussion] The precision, recall, and their harmonic mean (F1-score) were obtained by using gffcompare and are available in Supplementary Table 2: "The precision, recall, and their harmonic mean (F1-score) were obtained by using gffcompare and are available in Supplementary Table 2"
- [methods] stringtie --merge -G galGal6_ncbiRefSeq.gtf braker_fixed.gtf -o braker_merged.gtf: "stringtie --merge -G galGal6_ncbiRefSeq.gtf braker_fixed.gtf -o braker_merged.gtf"
- [discussion] using StringTie [57, 58] program with settings: -p 1 -j 2 -c 2 -v -a 4: "using StringTie [57, 58] program with settings: -p 1 -j 2 -c 2 -v -a 4"
- [readme] 2.1) Run genome-download.nf (i.e : output galGal6 genome): "2.1) Run genome-download.nf (i.e : output galGal6 genome)"
