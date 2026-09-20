---
name: repetitive-kmer-identification
description: Use when preparing HiFi read alignments against a repetitive draft assembly
  (e.g., a T2T or HiFiasm primary assembly) for NextPolish2 polishing, where you need
  winnowmap's -W repetitive k-mer list instead of plain minimap2 masking.
when_to_use_negative:
- The reference is low in segmental duplications or tandem repeats — minimap2 alone
  (-ax map-hifi/-ax map-pb) suffices and avoids the meryl counting step
- You are using meryl databases built with canu 1.8 or earlier — the rewritten meryl
  is not compatible with old .mcdat/.mcidx formats
- You need a k-mer set for quality-value (QV) estimation or Merqury/yak-based evaluation
  rather than for mapping — those workflows use yak k-mer datasets (k21/k31), not
  the winnowmap -W list
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: meryl
  role: counts all k=15 k-mers in the draft assembly and prints k-mers exceeding the
    distinct=0.9998 threshold to repetitive_k15.txt
  repo: https://github.com/marbl/meryl
- name: Winnowmap
  role: consumes repetitive_k15.txt via -W and uses weighted minimizer sampling to
    align PacBio HiFi reads (-ax map-pb) accurately within repeats
  repo: https://github.com/marbl/Winnowmap
- name: minimap2
  role: alternative long-read mapper for producing the HiFi mapping file when the
    reference is not repetitive
  repo: https://github.com/lh3/minimap2/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gpbjnl/qzad009
    title: 'NextPolish2: A Repeat-aware Polishing Tool for Genomes Assembled Using
      HiFi Long Reads'
  source_content_hash: sha256:4b96985e1ddd83715cea0c27efb793bce0d3496bfe5d7b6a42b8395c90ea4555
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/marbl/meryl
  tools:
  - NextPolish2
  - Winnowmap
  - meryl
  - minimap2
  - yak
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# repetitive-kmer-identification

## Summary

Identifies highly repetitive k-mers in a HiFi-derived draft assembly using meryl (k=15, distinct>0.9998) and supplies them to winnowmap via -W so that repetitive references — centromeric satellites, segmental duplications, large tandem repeats — are seeded correctly during HiFi read alignment for polishing.

## When to use

Use when preparing HiFi read alignments against a repetitive draft assembly (e.g., a T2T or HiFiasm primary assembly) for NextPolish2 polishing, where you need winnowmap's -W repetitive k-mer list instead of plain minimap2 masking.

## When NOT to use

- The reference is low in segmental duplications or tandem repeats — minimap2 alone (-ax map-hifi/-ax map-pb) suffices and avoids the meryl counting step
- You are using meryl databases built with canu 1.8 or earlier — the rewritten meryl is not compatible with old .mcdat/.mcidx formats
- You need a k-mer set for quality-value (QV) estimation or Merqury/yak-based evaluation rather than for mapping — those workflows use yak k-mer datasets (k21/k31), not the winnowmap -W list

## Inputs

- gzipped draft genome assembly FASTA (asm.fa.gz), e.g., a HiFiasm primary HiFi assembly
- PacBio HiFi long reads in FASTQ/FASTA (hifi.fasta.gz)

## Outputs

- repetitive_k15.txt — list of k=15 k-mers with distinct > 0.9998 in the assembly, for winnowmap -W
- meryl k-mer database directory (merylDB)
- hifi.map.sort.bam — sorted, indexed BAM of HiFi reads aligned to the draft assembly

## How to apply

Count all 15-mers in the gzipped draft assembly with meryl (meryl count k=15 output merylDB asm.fa.gz), then print only k-mers observed at near-saturation frequency with 'meryl print greater-than distinct=0.9998 merylDB > repetitive_k15.txt'. Pass this file to winnowmap via the -W flag and map PacBio HiFi reads with the map-pb preset (winnowmap -t 5 -W repetitive_k15.txt -ax map-pb asm.fa.gz hifi.fasta.gz), then sort and index the output with samtools to produce hifi.map.sort.bam. The rationale is that winnowmap's weighted minimizer sampling down-weights frequently occurring k-mers during seeding rather than masking them, preserving mapping accuracy in complex repeats; the repetitive k-mer list tells winnowmap which seeds are repeated so alignments within tandem repeats remain anchored. Minimap2 with map-pb or map-hifi presets is an acceptable fallback when the reference is not dominated by long repeats. The resulting sorted BAM is the alignment input consumed by the NextPolish2 correction engine.

## Related tools

- **meryl** (counts all k=15 k-mers in the draft assembly and prints k-mers exceeding the distinct=0.9998 threshold to repetitive_k15.txt) — https://github.com/marbl/meryl
- **Winnowmap** (consumes repetitive_k15.txt via -W and uses weighted minimizer sampling to align PacBio HiFi reads (-ax map-pb) accurately within repeats) — https://github.com/marbl/Winnowmap
- **minimap2** (alternative long-read mapper for producing the HiFi mapping file when the reference is not repetitive) — https://github.com/lh3/minimap2/

## Examples

```
meryl count k=15 output merylDB asm.fa.gz && meryl print greater-than distinct=0.9998 merylDB > repetitive_k15.txt
```

## Evaluation signals

- repetitive_k15.txt contains only 15-mers, one per line, all with distinct > 0.9998 in the meryl DB
- The winnowmap command includes -W repetitive_k15.txt and uses k=15 consistent with the meryl count step
- The resulting hifi.map.sort.bam is a valid sorted, indexed BAM consumable by NextPolish2
- Alignment coverage of repeat-rich regions (centromeric satellites, segmental duplications) is preserved relative to a plain minimap2 run, not lost to seed masking

## Limitations

- The distinct=0.9998 threshold and k=15 are specific to this NextPolish2 pipeline; other read types or assemblies may need different parameters
- Meryl databases from canu 1.8 or earlier are incompatible with the rewritten meryl required here
- Minimap2 retains frequent-k-mer masking behavior during seeding, which degrades mapping accuracy within complex repeats — hence winnowmap is preferred for repetitive references

## Evidence

- [methods] meryl count k=15 output merylDB asm.fa.gz
meryl print greater-than distinct=0.9998 merylDB > repetitive_k15.txt: "meryl count k=15 output merylDB asm.fa.gz
meryl print greater-than distinct=0.9998 merylDB > repetitive_k15.txt"
- [methods] winnowmap -t 5 -W repetitive_k15.txt -ax map-pb asm.fa.gz hifi.fasta.gz|samtools sort -o hifi.map.sort.bam -: "winnowmap -t 5 -W repetitive_k15.txt -ax map-pb asm.fa.gz hifi.fasta.gz|samtools sort -o hifi.map.sort.bam -"
- [intro] Error correction for a T2T genome assembly is challenging because (1) complex segmental duplications and large tandem repeats, such as centromeric satellite arrays: "Error correction for a T2T genome assembly is challenging because (1) complex segmental duplications and large tandem repeats, such as centromeric satellite arrays"
- [readme] Winnowmap implements a novel **weighted minimizer** sampling algorithm (>=v1.0). This optimization was motivated by the need to avoid masking of frequently occurring k-mers during the seeding stage: [evidence span withheld — anchor status: external_doc]
- [readme] Get the latest meryl code from this repo. This is not compatible with old meryl dbs built from canu 1.8 or earlier.: [evidence span withheld — anchor status: external_doc]
