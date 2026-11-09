---
name: vcf-variant-call-output
description: Use when a local-assembly variant caller (Novel-X) has produced placed
  NRS insertion candidates and they must be serialized and threshold-filtered into
  a standard VCF for downstream benchmarking.
when_to_use_negative:
- The caller produced raw orphan contigs that have not yet been placed on the reference
  — placement/anchoring must precede VCF emission
- The insertion is 300 bp or shorter — the method's output filter is restricted to
  insertions > 300 bp, and small insertions often lack sufficient barcode coverage
  anyway
- The goal is general SNV/small-indel calling from short reads, not non-reference
  sequence insertion discovery from synthetic long reads
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0157
tools:
- name: Novel-X
  role: Detects, genotypes, and places NRS insertions from 10x linked-read barcodes
    and writes the final threshold-filtered VCF named after the input BAM
  repo: github.com/1dayac/Novel-X
- name: Minimap2
  role: Aligns reassembled contigs to the reference genome; its anchor alignments
    feed the insertion signatures that become VCF records
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkac653
    title: Efficient detection and assembly of non-reference DNA sequences with synthetic
      long reads
  source_content_hash: sha256:5fc6cec88c8b5b166601d4c0a78f51103d0480daa04170120bf6bef36bb7e416
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BLASTN
  - Minimap2
  - Novel-X
  - SPAdes
  - Samtools
  - Velvet
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# vcf-variant-call-output

## Summary

Emit Novel-X's detected non-reference sequence (NRS) insertions as a parseable VCF named after the input BAM, retaining only calls > 300 bp whose summed anchor lengths exceed 300 bp to prevent false positives.

## When to use

Use when a local-assembly variant caller (Novel-X) has produced placed NRS insertion candidates and they must be serialized and threshold-filtered into a standard VCF for downstream benchmarking.

## When NOT to use

- The caller produced raw orphan contigs that have not yet been placed on the reference — placement/anchoring must precede VCF emission
- The insertion is 300 bp or shorter — the method's output filter is restricted to insertions > 300 bp, and small insertions often lack sufficient barcode coverage anyway
- The goal is general SNV/small-indel calling from short reads, not non-reference sequence insertion discovery from synthetic long reads

## Inputs

- LongRanger-aligned 10x BAM (with barcode tags) processed by Novel-X
- Minimap2 contig-to-reference alignments with insertion signatures from the placement step

## Outputs

- VCF file named after the input BAM, located in the Novel-X output directory, with one record per NRS insertion > 300 bp (anchor length sum > 300 bp) including the insertion sequence

## How to apply

After Novel-X's four-step pipeline (orphan contig assembly with Velvet k=63, barcode list extraction with t=5 barcode minimum, reassembly per barcode set with SPAdes k=77 cov-cutoff=3, and Minimap2 anchor placement with QUAST-like filtering), write detected insertion signatures to a VCF file named after the input BAM inside the output directory. Only emit insertions longer than 300 bp whose sum of anchor lengths exceeds 300 bp, per the paper's false-call prevention rule. A VCF record should carry the insertion sequence itself (as in the demo chr1 record near position 29503). Validate the output by confirming the VCF exists in the outdir, parses as VCF, and that every record satisfies the >300 bp insertion and >300 bp anchor-sum thresholds.

## Related tools

- **Novel-X** (Detects, genotypes, and places NRS insertions from 10x linked-read barcodes and writes the final threshold-filtered VCF named after the input BAM) — github.com/1dayac/Novel-X
- **Minimap2** (Aligns reassembled contigs to the reference genome; its anchor alignments feed the insertion signatures that become VCF records)

## Examples

```
import gzip; recs=[l for l in gzip.open('sample.bam.vcf','rt') if not l.startswith('#')]; assert all(int(r.split(chr(9))[7].split('SVLEN=')[1].split(';')[0].rstrip('+-'))>300 for r in recs)
```

## Evaluation signals

- A VCF file exists in the outdir and is named after the input BAM
- The file parses as valid VCF
- Every record's insertion length exceeds 300 bp and the sum of its anchor lengths exceeds 300 bp
- In the demo run, a single chr1 record around position 29503 is present with the insertion sequence included
- Novel-X output should reproduce the benchmark VCFs published in 1dayac/novel_insertions_supplementary (e.g. NA12878, HG002, CHM13 10X Chromium calls)

## Limitations

- Small insertions typically recruit too few distinct barcodes, so coverage is insufficient for anchor assembly and they are missed or filtered out
- The output is restricted to insertions > 300 bp with anchor-sum > 300 bp; calls below these thresholds are suppressed by design
- Known unresolved issue on the filter_target_contig stage is flagged in the methods ('There is a problem on filter_target_contig stage at the moment')

## Evidence

- [methods] Output insertions longer than 300 bp whose summed anchor lengths exceed 300 bp into a VCF file named after the input BAM: [evidence span withheld — anchor status: not_found]
- [other] Demo validation: single chr1 record near 29503 with insertion sequence present: [evidence span withheld — anchor status: not_found]
- [readme] Novel-X detects and genotypes insertions using alignment signatures and barcodes: "Novel-X detects and genotypes novel sequence insertions in 10X sequencing dataset using non-trivial read alignment signatures and barcode information."
- [supplementary] Benchmark VCFs available in the supplementary repository: [evidence span withheld — anchor status: external_doc]
