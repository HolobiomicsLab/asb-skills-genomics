---
name: paired-end-and-long-read-alignment-input-handling
description: Use when input is a sorted/indexed BAM/CRAM (bwa-mem PE or minimap2/ngmlr
  long reads) plus an indexed reference FASTA and SV calling with dysgu is about to
  begin; triggers include mixed read-type pipelines (PE + PacBio/ONT) needing divergent
  invocations.
when_to_use_negative:
- Input is raw FASTQ rather than an aligned, sorted/indexed BAM/CRAM — alignment (bwa
  mem for PE, minimap2/ngmlr for LR) must happen first
- The task is benchmarking caller output against truth sets rather than running the
  caller itself — svbench is the appropriate tool for that stage
- Sample data are tumor/somatic with copy-number changes where the default diploid
  model is inappropriate; switch the model rather than the input-handling path
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: dysgu
  role: SV caller whose run/fetch/call commands consume the sorted BAM/CRAM and reference
    FASTA; handles both PE and LR inputs
  repo: https://github.com/kcleal/dysgu
- name: Manta
  role: Baseline PE SV caller whose input requirements (sorted indexed BAM + reference)
    are analogous in benchmark comparisons
- name: nanovar
  role: Baseline LR SV caller consuming the same long-read BAM inputs in benchmark
    comparisons
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkac039
    title: 'Dysgu: efficient structural variant calling using short or long reads'
  source_content_hash: sha256:267942934380ecc67a6ffb7232aa17af6c757f290c40bdf2794cc7b046173923
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kcleal/dysgu
  tools:
  - Manta
  - delly
  - dysgu
  - lumpy
  - nanovar
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# paired-end-and-long-read-alignment-input-handling

## Summary

Prepares sorted/indexed BAM/CRAM alignment inputs (PE or LR) for dysgu SV calling, selecting the correct run/call pipeline, --mode preset, and insert-size handling per read type. Ensures dysgu receives the alignment-cigar, discordant/split-read and reference inputs it needs before calling.

## When to use

Use when input is a sorted/indexed BAM/CRAM (bwa-mem PE or minimap2/ngmlr long reads) plus an indexed reference FASTA and SV calling with dysgu is about to begin; triggers include mixed read-type pipelines (PE + PacBio/ONT) needing divergent invocations.

## When NOT to use

- Input is raw FASTQ rather than an aligned, sorted/indexed BAM/CRAM — alignment (bwa mem for PE, minimap2/ngmlr for LR) must happen first
- The task is benchmarking caller output against truth sets rather than running the caller itself — svbench is the appropriate tool for that stage
- Sample data are tumor/somatic with copy-number changes where the default diploid model is inappropriate; switch the model rather than the input-handling path

## Inputs

- Sorted and indexed paired-end BAM/CRAM (e.g. bwa mem-aligned Illumina PE reads)
- Sorted and indexed long-read BAM/CRAM (minimap2- or ngmlr-aligned PacBio HiFi/Revio or ONT R9/R10 reads)
- Indexed reference genome FASTA (e.g. GRCh37/GRCh38)
- Optional: phased BAM with HP/PS tags (from hiphase/whatshap/longshot) for haplotag-aware calling

## Outputs

- VCF of called SVs/indels written to stdout (svs.vcf)
- Temporary working directory containing input.dysgu_reads.bam and analysis files (>5 GB for a 30X PE genome unless --clean is used)

## How to apply

Confirm the input BAM/CRAM is sorted and indexed and the reference genome FASTA is indexed, since both the run and call commands require this. For paired-end data (bwa mem-aligned), use `dysgu run`, which wraps `fetch` (separating SV-associated reads into a temp BAM of split/discordant reads and reads with soft-clips >= 15 bp) and `call`; add `-p` for multiprocessing and `--clean` to remove temp files. For long reads (minimap2 or ngmlr-aligned), use `dysgu call` with the appropriate `--mode` preset (pacbio-sequel2, pacbio-revio, nanopore-r9, nanopore-r10) and set `--divergence auto` for higher-error reads, since the default divergence of 0.02 can over-filter lower-accuracy reads. For PE data pass `--ibam` to the call step so insert-size metrics are inferred from the full alignment file rather than the small extracted reads BAM (or supply insert size manually with -I). Choose the model deliberately: default diploid model uses read-depth around break sites for germline WGS; use `--diploid False` for somatic samples with low allelic fraction, and `--contigs False` only if those fail.

## Related tools

- **dysgu** (SV caller whose run/fetch/call commands consume the sorted BAM/CRAM and reference FASTA; handles both PE and LR inputs) — https://github.com/kcleal/dysgu
- **Manta** (Baseline PE SV caller whose input requirements (sorted indexed BAM + reference) are analogous in benchmark comparisons)
- **nanovar** (Baseline LR SV caller consuming the same long-read BAM inputs in benchmark comparisons)

## Examples

```
dysgu run -p4 --clean reference.fa temp_dir input.bam > svs.vcf
```

## Evaluation signals

- dysgu test passes after installation, confirming the tool can read BAM/CRAM and reference inputs
- fetch stage produces temp_dir/input.dysgu_reads.bam containing split/discordant/soft-clipped reads (soft-clip >= 15 bp default for PE)
- Call stage emits a VCF to stdout; empty or near-empty output suggests divergence/insert-size misconfiguration
- For PE, insert-size metrics were taken from the full alignment file (--ibam all_reads.bam) rather than the small dysgu_reads.bam
- Correct --mode preset used for the read chemistry (e.g. pacbio-revio, nanopore-r10) and --divergence auto applied when read accuracy is uncertain

## Limitations

- Default sequence divergence 0.02 can be too stringent for lower-accuracy reads, filtering more reads and lowering sensitivity; use --divergence auto
- When fetch/run is used, a large temp file of SV-associated reads (>5 GB for a 30X PE genome) is generated; --clean or adequate disk is required
- Haplotag-based phased output requires since v1.8 and a pre-phased BAM (hiphase/whatshap/longshot); unphased inputs lose that accuracy gain

## Evidence

- [readme] To call SVs, a sorted and indexed .bam/cram is needed plus an indexed reference genome (fasta format).: [evidence span withheld — anchor status: external_doc]
- [readme] This is achieved using the `fetch` command: [evidence span withheld — anchor status: external_doc]
- [readme] Since v1.8, higher calling accuracy can be achieved by adding "haplotags": [evidence span withheld — anchor status: external_doc]
- [intro] Conceptually, dysgu identifies SVs from alignment cigar information as well as discordant and split-read mappings.: "Conceptually, dysgu identifies SVs from alignment cigar information as well as discordant and split-read mappings."
- [readme] Dysgy is designed to work with long reads aligned using minimap2 or ngmlr.: [evidence span withheld — anchor status: external_doc]
