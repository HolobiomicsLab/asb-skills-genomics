---
name: gtf-preprocessing-and-annotation-merging
description: Use when gene-prediction outputs (braker.gtf, tsebra.gtf, AUGUSTUS GTF)
  need format normalization and merging with a reference GTF (e.g., galGal6_ncbiRefSeq.gtf)
  before gffcompare-based precision/recall/F1 benchmarking.
when_to_use_negative:
- Organism lacks an assembled genome or an NCBI-available GTF reference annotation
  — stringtie --merge -G and the reconciliation strategy depend on a reference GTF,
  and the pipeline is stated to apply only to organisms that have both
- You only need to run annotate-my-genomes/add-ncbi-annotation itself; this skill's
  AGAT conversion and merge steps exist specifically to prepare comparator annotations
  for benchmarking
- Comparator GTFs are already format-compliant and coordinate-compatible with the
  reference, making AGAT conversion and --merge reconciliation redundant
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0622
tools:
- name: AGAT
  role: agat_convert_sp_gff2gtf.pl pre-processes braker.gtf/tsebra.gtf into compliant
    GTF before merging
  repo: https://github.com/NBISweden/AGAT
- name: StringTie
  role: stringtie --merge -G merges the fixed prediction GTF with the reference annotation
    (e.g., galGal6_ncbiRefSeq.gtf -> braker_merged.gtf)
- name: gffcompare
  role: compares each merged/filtered pipeline GTF against the reference annotation
    to obtain the .tmap and precision/recall/F1 statistics
- name: BRAKER
  role: produces the comparator braker.gtf annotations that are pre-processed and
    merged
  repo: https://github.com/Gaius-Augustus/BRAKER
- name: TSEBRA
  role: produces tsebra.gtf comparator annotations that are pre-processed and merged
  repo: https://github.com/Gaius-Augustus/TSEBRA
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giac099
    title: 'annotate_my_genomes: an easy-to-use pipeline to improve genome annotation
      and uncover neglected genes by hybrid RNA sequencing'
  source_content_hash: sha256:618080df202957ca326fcba5f6fa73f19c180d67a353223bef510090f818c927
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NBISweden/AGAT
  tools:
  - AGAT
  - AUGUSTUS
  - BRAKER
  - BRAKER1
  - BRAKER2
  - IsoSeqSim
  - StringTie
  - TSEBRA
  - gffcompare
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# gtf-preprocessing-and-annotation-merging

## Summary

Standardizes gene-prediction GTF outputs (BRAKER1/2, TSEBRA, AUGUSTUS) with the AGAT toolkit and reconciles them with a reference annotation using StringTie --merge, enabling like-for-like gffcompare benchmarking of annotate_my_genomes against existing pipelines.

## When to use

Use when gene-prediction outputs (braker.gtf, tsebra.gtf, AUGUSTUS GTF) need format normalization and merging with a reference GTF (e.g., galGal6_ncbiRefSeq.gtf) before gffcompare-based precision/recall/F1 benchmarking.

## When NOT to use

- Organism lacks an assembled genome or an NCBI-available GTF reference annotation — stringtie --merge -G and the reconciliation strategy depend on a reference GTF, and the pipeline is stated to apply only to organisms that have both
- You only need to run annotate-my-genomes/add-ncbi-annotation itself; this skill's AGAT conversion and merge steps exist specifically to prepare comparator annotations for benchmarking
- Comparator GTFs are already format-compliant and coordinate-compatible with the reference, making AGAT conversion and --merge reconciliation redundant

## Inputs

- Gene-prediction GTF/GFF outputs: braker.gtf (BRAKER1/BRAKER2), tsebra.gtf (TSEBRA), AUGUSTUS GTF
- Reference coding GTF annotation matched to the genome assembly (e.g., galGal6_ncbiRefSeq.gtf)
- Genome assemblies retrieved with genome-download (galGal6, hg38, mm10, danRer11, ce11)

## Outputs

- AGAT-converted GTF files (e.g., braker_fixed.gtf)
- Merged annotation GTFs (e.g., braker_merged.gtf) from stringtie --merge -G
- grep-filtered subset of StringTie-tagged transcripts per pipeline
- gffcompare outputs: gffcompare.tmap transcript map and accuracy stats (precision, recall, F1 at base/exon/intron/intron-chain/transcript/locus levels)

## How to apply

Pre-process each comparator's annotation by running agat_convert_sp_gff2gtf.pl on braker.gtf/tsebra.gtf to produce a format-compliant GTF (braker_fixed.gtf), since AGAT must be installed for this step. Reconcile the fixed prediction GTF with the matched NCBI reference annotation via 'stringtie --merge -G <ref>.gtf braker_fixed.gtf -o <pred>_merged.gtf' (e.g., -G galGal6_ncbiRefSeq.gtf), which unions transcript loci so downstream comparison is not biased by ID/locus incompatibilities. Retain only the StringTie-tagged transcripts from the merged file with grep so that accuracy metrics reflect the predicted annotation, not the reference backbone. Then run gffcompare of each pipeline's GTF against the corresponding reference annotation (galGal6, hg38, mm10, danRer11, ce11) to obtain the .tmap and precision/recall/F1 at base, exon, intron, intron-chain, transcript, and locus levels; the rationale is to reproduce the Supplementary Table 2 comparison in which annotate_my_genomes performed equal or better than existing pipelines on StringTie de-novo assemblies.

## Related tools

- **AGAT** (agat_convert_sp_gff2gtf.pl pre-processes braker.gtf/tsebra.gtf into compliant GTF before merging) — https://github.com/NBISweden/AGAT
- **StringTie** (stringtie --merge -G merges the fixed prediction GTF with the reference annotation (e.g., galGal6_ncbiRefSeq.gtf -> braker_merged.gtf))
- **gffcompare** (compares each merged/filtered pipeline GTF against the reference annotation to obtain the .tmap and precision/recall/F1 statistics)
- **BRAKER** (produces the comparator braker.gtf annotations that are pre-processed and merged) — https://github.com/Gaius-Augustus/BRAKER
- **TSEBRA** (produces tsebra.gtf comparator annotations that are pre-processed and merged) — https://github.com/Gaius-Augustus/TSEBRA

## Examples

```
agat_convert_sp_gff2gtf.pl --gff braker.gtf -o braker_fixed.gtf; stringtie --merge -G galGal6_ncbiRefSeq.gtf braker_fixed.gtf -o braker_merged.gtf
```

## Evaluation signals

- agat_convert_sp_gff2gtf.pl completes without error and braker_fixed.gtf is accepted by stringtie --merge
- stringtie --merge -G <ref>.gtf yields braker_merged.gtf containing transcripts reconciled from both the reference and the prediction
- grep on the merged GTF retains only StringTie-tagged transcripts, excluding the reference-backbone transcripts
- gffcompare reports precision, recall, and harmonic-mean F1 at base, exon, intron, intron-chain, transcript, and locus levels for every genome/pipeline combination
- Compiled F1-scores reproduce Supplementary Table 2 and confirm annotate_my_genomes performed equal or better than BRAKER1/2, TSEBRA, and AUGUSTUS on StringTie de-novo assemblies

## Limitations

- Requires an assembled genome and an NCBI-available annotation in GTF format; no procedure is provided for organisms lacking these
- F1-scores and precision/recall values are deferred to Supplementary Table 2 and are not reported in the main text, so reproduction requires the supplementary files
- The benchmark covers five genomes (galGal6, hg38, mm10, danRer11, ce11) and simulated PacBio reads generated with IsoSeqSim, so conclusions may not transfer to other species or real long-read data

## Evidence

- [methods] The AGAT toolkit must be installed before converting comparator GTFs: "the AGAT toolkit (https://github.com/NBISweden/AGAT) must be installed"
- [methods] Merging command reconciles prediction GTF with the reference GTF: "stringtie --merge -G galGal6_ncbiRefSeq.gtf braker_fixed.gtf -o braker_merged.gtf"
- [discussion] Downstream accuracy evaluation is performed with gffcompare: "The precision, recall, and their harmonic mean (F1-score) were obtained by using gffcompare"
- [discussion] Benchmark expectation after merging and comparison: "According to the F1-score, using StringTie de-novo assemblies, our method performed equal or better than existing pipelines"
