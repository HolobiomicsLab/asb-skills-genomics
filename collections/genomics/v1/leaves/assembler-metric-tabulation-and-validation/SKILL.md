---
name: assembler-metric-tabulation-and-validation
description: Use when comparing de novo transcriptome assembler outputs via rnaQUAST
  and you need a per-assembler CSV of quality metrics verified against published anchor
  values before downstream ranking or reproduction claims.
when_to_use_negative:
- Input assemblies are from real (non-simulated) RNA-Seq datasets where the paper's
  anchor values (309/126 misassemblies, 1.74/1.93 duplication ratios) do not apply
  and re-derivation of expectations is required
- The goal is de novo assembly itself rather than post-hoc quality comparison of existing
  assemblies
- Evaluating metatranscriptome assemblies — the article notes its approach was designed
  for single-organism RNA-Seq and metatranscriptomic quality-evaluation pipelines
  remain future work
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: rnaQUAST
  role: Reference-based quality evaluation of each de novo assembly, producing the
    four benchmark metrics per assembler
- name: rnaSPAdes
  role: Transcriptome assembler being benchmarked; run with default k-values on the
    simulated Human reads
  repo: github.com/ablab/spades
- name: SPAdes
  role: Competitor assembler run unmodified in single-cell mode with k-mer sizes 21,
    33, 55
  repo: github.com/ablab/spades
- name: Trinity
  role: Competing assembler run under default settings; closest competitor with duplication
    ratio 1.74
- name: RNA-Bloom
  role: Competing assembler run under default settings; third-ranked with duplication
    ratio 1.93
- name: Trans-ABySS
  role: Competing assembler run under default settings; anchor value of 126 misassemblies
    on the simulated Human dataset
- name: RSEM
  role: Read simulator used to generate the Human simulated RNA-Seq dataset (Supplementary
    Section S3 command lines)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giz100
    title: 'rnaSPAdes: a <i>de novo</i> transcriptome assembler and its application
      to RNA-Seq data'
  source_content_hash: sha256:d7ccda0e592aea2f101ff2eceb9dab5dc878cc10339aea477ea74e9f97e3a86c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BinPacker
  - Bridger
  - IDBA-tran
  - RNA-Bloom
  - RSEM
  - SOAPdenovo-Trans
  - SPAdes
  - Trans-ABySS
  - Trinity
  - rnaQUAST
  - rnaSPAdes
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# assembler-metric-tabulation-and-validation

## Summary

Extracts benchmark metrics (95%-assembled genes/isoforms, database coverage, misassemblies, duplication ratio) from rnaQUAST reports for each transcriptome assembler on the Human simulated RNA-Seq dataset and tabulates them into a CSV, validating against paper-reported anchor values (e.g. rnaSPAdes 309 misassemblies, Trans-ABySS 126 misassemblies, duplication ratios 1.74/1.93/2.19).

## When to use

Use when comparing de novo transcriptome assembler outputs via rnaQUAST and you need a per-assembler CSV of quality metrics verified against published anchor values before downstream ranking or reproduction claims.

## When NOT to use

- Input assemblies are from real (non-simulated) RNA-Seq datasets where the paper's anchor values (309/126 misassemblies, 1.74/1.93 duplication ratios) do not apply and re-derivation of expectations is required
- The goal is de novo assembly itself rather than post-hoc quality comparison of existing assemblies
- Evaluating metatranscriptome assemblies — the article notes its approach was designed for single-organism RNA-Seq and metatranscriptomic quality-evaluation pipelines remain future work

## Inputs

- FASTQ reads from the Human simulated RNA-Seq dataset (RSEM-simulated)
- Transcriptome assembly FASTA output from each assembler (rnaSPAdes, SPAdes, Trinity, RNA-Bloom, Trans-ABySS, BinPacker, Bridger, IDBA-tran, SOAPdenovo-Trans)
- Human reference transcriptome for rnaQUAST evaluation
- rnaQUAST report files per assembler

## Outputs

- CSV table with one row per assembler and columns for 95%-assembled genes, 95%-assembled isoforms, database coverage, misassemblies, and duplication ratio
- Validation check of CSV values against paper-reported anchors (rnaSPAdes 309 misassemblies, Trans-ABySS 126 misassemblies, duplication ratios 1.74/1.93/2.19)

## How to apply

Assemble the Human simulated RNA-Seq reads (generated with the RSEM simulator per Supplementary Section S3) with each assembler — BinPacker, Bridger, IDBA-tran, RNA-Bloom, SOAPdenovo-Trans, Trans-ABySS, Trinity under default settings, plus rnaSPAdes with default k-values and unmodified SPAdes single-cell mode (k = 21, 33, 55). Run rnaQUAST on each assembly against the human reference transcriptome and extract four metrics per assembler: fraction of 95%-assembled genes and isoforms, database coverage, number of misassemblies, and duplication ratio. Write one CSV row per assembler, then validate the table against reported anchors: rnaSPAdes 309 misassemblies, Trans-ABySS 126 misassemblies, and duplication ratios 1.74 (Trinity), 1.93 (RNA-Bloom), and 2.19 for the leading assemblers on the simulated Human dataset. Rationale: rnaQUAST is the paper's designated quality-evaluation tool, and the four metrics capture completeness (95%-assembled genes/isoforms, where rnaSPAdes leads with 0.50 and 0.32), accuracy (misassemblies), and redundancy (duplication ratio, where Trinity and RNA-Bloom score 1.74 and 1.93).

## Related tools

- **rnaQUAST** (Reference-based quality evaluation of each de novo assembly, producing the four benchmark metrics per assembler)
- **rnaSPAdes** (Transcriptome assembler being benchmarked; run with default k-values on the simulated Human reads) — github.com/ablab/spades
- **SPAdes** (Competitor assembler run unmodified in single-cell mode with k-mer sizes 21, 33, 55) — github.com/ablab/spades
- **Trinity** (Competing assembler run under default settings; closest competitor with duplication ratio 1.74)
- **RNA-Bloom** (Competing assembler run under default settings; third-ranked with duplication ratio 1.93)
- **Trans-ABySS** (Competing assembler run under default settings; anchor value of 126 misassemblies on the simulated Human dataset)
- **RSEM** (Read simulator used to generate the Human simulated RNA-Seq dataset (Supplementary Section S3 command lines))

## Examples

```
rnaQUAST.py --transcripts rnaSPAdes_transcripts.fasta Trinity.fasta RNA-Bloom.fasta TransABySS.fasta --reference human_transcriptome.fa --output_dir rnaquast_human_sim
```

## Evaluation signals

- rnaSPAdes misassemblies on the Human simulated dataset equal 309 and Trans-ABySS misassemblies equal 126 in the CSV
- Duplication ratios in the CSV include 1.74 (Trinity) and 1.93 (RNA-Bloom) among the leading assemblers
- rnaSPAdes has the highest fraction of 95%-assembled genes (0.50) and isoforms (0.32), with Trinity second (0.44 and 0.30) and RNA-Bloom third (0.41 and 0.28)
- CSV contains exactly one row per assembler (nine total including rnaSPAdes and unmodified SPAdes) with all four metrics populated
- Average database coverage ranks rnaSPAdes best (74%), followed by Trinity (72%), Trans-ABySS (71%), and RNA-Bloom (68%)

## Limitations

- No single transcriptome assembler is the absolute leader across all quality metrics and all datasets tested; assembler choice may require additional per-project benchmarks
- High database coverage or nucleotide recall can be achieved by very fragmented assemblies (or even raw reads), so these metrics do not reflect assembly contiguity
- Unmodified SPAdes has a smaller mean duplication ratio (2% vs 32% for rnaSPAdes), so lower duplication should not be read as overall superiority

## Evidence

- [discussion] In comparison with other assemblers, rnaSPAdes shows the highest fraction of 95%-assembled genes and isoforms (0.50 and 0.32, respectively).: "rnaSPAdes shows the highest fraction of 95%-assembled genes and isoforms (0.50 and 0.32, respectively)"
- [intro] To perform the benchmarking we have used the rnaQUAST tool [16], which was designed for quality evaluation of de novo assemblies: "To perform the benchmarking we have used the rnaQUAST tool [16], which was designed for quality evaluation of de novo assemblies"
- [full_text] Trinity and RNA-Bloom had high duplication ratios of 1.74 and 1.93, respectively.: "Trinity (0.44 and 0.30) and RNA-Bloom (0.41 and 0.28) are ranked second and the third according to these metrics"
- [discussion] for which rnaSPAdes also has the best average value across all datasets (74%), followed by Trinity (72%), Trans-ABySS (71%), and RNA-Bloom (68%).: "rnaSPAdes also has the best average value across all datasets (74%), followed by Trinity (72%), Trans-ABySS (71%), and RNA-Bloom (68%)"
- [discussion] high database coverage or nucleotide recall can be achieved by a very fragmented assembly (or even raw reads), which, indeed, does not suit well for further reference-free analysis.: "high database coverage or nucleotide recall can be achieved by a very fragmented assembly (or even raw reads)"
