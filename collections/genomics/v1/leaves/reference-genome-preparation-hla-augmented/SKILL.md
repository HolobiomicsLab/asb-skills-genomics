---
name: reference-genome-preparation-hla-augmented
description: Use when preparing to align long-read (Oxford Nanopore or PacBio) whole-genome
  shotgun data for HLA typing with HLAminer, and a combined human+HLA-allele reference
  must be built and indexed before streaming minimap2 alignments.
when_to_use_negative:
- The analysis targets short Illumina reads — HLAminer's HPTASR targeted-assembly
  route with a k-mer hash table (not the long-read augmented reference) is the intended
  path for 100–150 bp reads
- The goal is routine whole-genome variant calling where a standard reference including
  chr6 is correct; stripping chr6 would lose that locus
- The HLA database must remain frozen at an older IPD-IMGT/HLA release for reproducibility
  and no updated-allele augmentation is wanted
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3407
tools:
- name: HLAminer
  role: Supplies the ./database/ scripts to download and format updated IMGT/HLA sequences
    and consumes the combined reference as its HLA-I_II_GEN.fasta prediction database
  repo: https://github.com/bcgsc/HLAminer
- name: minimap2
  role: Indexes the combined GRCh38+HLA reference and aligns ONT long reads (-ax map-ont
    --MD) whose SAM output is streamed into HLAminer
  repo: https://github.com/lh3/minimap2
- name: TASR
  role: Alternative consumer of the known HLA class I/II reference sequences, building
    the 15-nt k-mer hash table for the HPTASR assembly route
  repo: https://github.com/warrenlr/TASR
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/cpz1.70124
    title: Streaming Long‐Read Sequence Alignments for HLA Predictions Using HLAminer
  source_content_hash: sha256:9d97ed98287a4f16caed0a5c46fb910c1b9c97d7c147297030c64840f7c850e9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bcgsc/HLAminer
  tools:
  - HLAminer
  - TASR
  - minimap2
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3407
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# reference-genome-preparation-hla-augmented

## Summary

Prepare a combined long-read alignment reference consisting of GRCh38 (GCA_000001405.15, chromosome-only with chr6 excluded) augmented with HLA class I and II allele sequences downloaded from the IMGT/HLA repository, then index it for minimap2 so that HLA reads map to the allele targets rather than being suppressed by the alt/chr6 reference locus.

## When to use

Use when preparing to align long-read (Oxford Nanopore or PacBio) whole-genome shotgun data for HLA typing with HLAminer, and a combined human+HLA-allele reference must be built and indexed before streaming minimap2 alignments.

## When NOT to use

- The analysis targets short Illumina reads — HLAminer's HPTASR targeted-assembly route with a k-mer hash table (not the long-read augmented reference) is the intended path for 100–150 bp reads
- The goal is routine whole-genome variant calling where a standard reference including chr6 is correct; stripping chr6 would lose that locus
- The HLA database must remain frozen at an older IPD-IMGT/HLA release for reproducibility and no updated-allele augmentation is wanted

## Inputs

- GRCh38 genome assembly FASTA, GCA_000001405.15, chromosome-only, chr6 excluded
- HLA class I and class II allele sequences from the IMGT/HLA public repository
- hla_nom_p.txt P-designation file from hla.alleles.org
- HLAminer ./database/ download/format shell scripts

## Outputs

- Combined reference FASTA (e.g. GCA_000001405.15_GRCh38_genomic.chr-only-noChr6-HLA-I_II_GEN.fa.gz / HLA-I_II_GEN.fasta)
- minimap2 index of the combined reference
- HLAminer HPRA database file supplied via HLA-I_II_GEN.fasta to HLAminer.pl

## How to apply

Download the latest HLA class I and II allele sequences from the EBI/IMGT-HLA repository — shell scripts for automatic download and database formatting ship in the HLAminer ./database/ directory — and retrieve the P-designation file hla_nom_p.txt from hla.alleles.org. Combine these allele sequences with the chromosome-only GRCh38 assembly (GCA_000001405.15) from which chr6 has been removed, yielding the file named GCA_000001405.15_GRCh38_genomic.chr-only-noChr6-HLA-I_II_GEN.fa.gz (e.g. HLA-I_II_GEN.fasta for HLAminer's database argument). Index the combined FASTA with 'minimap2 -x map-ont -d' (or let minimap2 build the index on the fly) so ONT reads align with '-t 60 -ax map-ont --MD'. The rationale: removing chr6 and adding the up-to-date allele targets lets reads from the highly polymorphic HLA region align directly to reference alleles, which the streamed SAM output feeds into HLAminer.pl (-a stream) for HPRA-style prediction without writing bulky .sam files to disk.

## Related tools

- **HLAminer** (Supplies the ./database/ scripts to download and format updated IMGT/HLA sequences and consumes the combined reference as its HLA-I_II_GEN.fasta prediction database) — https://github.com/bcgsc/HLAminer
- **minimap2** (Indexes the combined GRCh38+HLA reference and aligns ONT long reads (-ax map-ont --MD) whose SAM output is streamed into HLAminer) — https://github.com/lh3/minimap2
- **TASR** (Alternative consumer of the known HLA class I/II reference sequences, building the 15-nt k-mer hash table for the HPTASR assembly route) — https://github.com/warrenlr/TASR

## Examples

```
minimap2 -t 60 -ax map-ont --MD GCA_000001405.15_GRCh38_genomic.chr-only-noChr6-HLA-I_II_GEN.fa ERR2585115.fastq.gz | HLAminer.pl -a stream -d HLA-I_II_GEN.fasta -p hla_nom_p.txt -s 500 -q 1 -i 1
```

## Evaluation signals

- The reference file name matches the expected pattern GCA_000001405.15_GRCh38_genomic.chr-only-noChr6-HLA-I_II_GEN.fa.gz, i.e. chr6 is absent and HLA-I/II allele sequences are present
- minimap2 index construction completes and alignments against the combined reference resolve to the HLA allele targets (e.g. groups A26/A33, alleles A*26:33 and A*33:24 for NA19240 ERR2585115)
- The hla_nom_p.txt P-designation file loads so HLAminer can collapse alleles with identical antigen-binding domains to P-designated entries
- The streamed minimap2→HLAminer run on the 10x NA19240 dataset finishes in roughly ~40 minutes wall-clock (measured with /usr/bin/time -v), confirming the reference/index is usable at the expected scale
- Downstream CSV (HLAminer_HPRA_*.csv) predictions agree with the clinically confirmed HLA types, indicating the augmented reference and database are consistent

## Limitations

- The IMGT/HLA repository is continually expanding, so the augmented reference must be rebuilt from updated downloads to capture newly described alleles; predictions depend on database currency
- Older, less accurate nanopore chemistry (e.g. the 2018 PromethION NA19240 data) at ~10x coverage limits resolution, so four-digit allele calls may be ambiguous
- Larger k values in the TASR alternative route increase specificity at the possible expense of sensitivity, and blast 2.2.22 may be faster than blast+ 2.2.28 for the reciprocal BLASTN step

## Evidence

- [methods] GCA_000001405.15_GRCh38_genomic.chr-only-noChr6-HLA-I_II_GEN.fa.gz: "GCA_000001405.15_GRCh38_genomic.chr-only-noChr6-HLA-I_II_GEN.fa.gz"
- [methods] download updated HLA sequences: "Follow these instructions to download updated HLA sequences from ebi/imgt (shell scripts to automatically download and format the databases exist in ./database/)"
- [methods] IMGT/HLA reference sequences for TASR: "Known HLA class I/class II reference sequences available from the IMGT/HLA public repository are read by TASR using default options (Warren and Holt 2011)"
- [intro] protocol enables reference preparation and single-command prediction: "This protocol enables the preparation of reference genome sequences for long‐read alignments and, using a single command at the Unix terminal, prediction of likely HLA class I and II alleles"
- [abstract] robust predictions with low-coverage older nanopore data: "demonstrate the robustness of predictions even with older, less accurate WGS nanopore datasets and relatively low (10×) sequencing coverage"
