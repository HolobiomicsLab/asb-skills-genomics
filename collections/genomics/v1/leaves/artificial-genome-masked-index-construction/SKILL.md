---
name: artificial-genome-masked-index-construction
description: Use when detecting tRNA-derived non-coding RNAs (tRF-5, tRF-3, tRF-1,
  tRNA halves) from small RNA-seq and you need an index that forces tncRNA assignments
  to come only from tRNA loci plus 50 nt leader/trailer flanks.
when_to_use_negative:
- The analysis goal is general small RNA annotation (miRNA, piRNA, rRNA-derived reads)
  rather than tRNA-derived reads — a SPORTS1.0-style annotation pipeline is more appropriate
- The genome lacks reliable tRNA annotations or tRNAscan-SE cannot be run for the
  organism, so masking coordinates and mature/leader/trailer FASTAs cannot be generated
- Input data is paired-end or long-read RNA-seq rather than single-end small RNA reads
  of 14–50 nt, which is the size class this artificial genome design was built to
  classify
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0099
tools:
- name: tRNAscan-SE
  role: Predicts nuclear (eukaryotes model) and organellar ('-O' option) tRNA genes
    whose filtered regions define the mask and flanks; pseudogenes and score <50 tRNAs
    are eliminated
  repo: https://github.com/skbinfo/tncRNA-Toolkit
- name: bedtools
  role: maskfasta script masks filtered tRNA regions with 50 bp 3' and 5' flanks to
    create the artificial genome
- name: Bowtie1
  role: bowtie-build creates the per-species index of the artificial genome; provided
    in the toolkit tar package (v1.3.0)
  repo: https://github.com/skbinfo/tncRNA-Toolkit
- name: tncRNA Toolkit
  role: Workflow wrapping index construction via 'python3 tncRNAs.py -g <genome fasta>
    -s <species name>'
  repo: https://github.com/skbinfo/tncRNA-Toolkit
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2021.09.021
    title: 'Transfer RNA-derived non-coding RNAs (tncRNAs): Hidden regulation of plants''
      transcriptional regulatory circuits'
  source_content_hash: sha256:e4812294f1c59f3b273783b35fc019847a93ae22d9bbd04284c717f18a154e88
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/skbinfo/tncRNA-Toolkit
  repo_url: https://github.com/skbinfo/tncRNA-Toolkit
  tools:
  - Bowtie1
  - HAMR
  - bedtools
  - biopython
  - pandas
  - tRNAscan-SE
  - tRNAscanSE
  - tncRNA Toolkit
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# artificial-genome-masked-index-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Builds a Bowtie1 'artificial genome' in which genuine tRNA gene regions are masked with 50 nt 5'/3' flanks so that sRNA-seq reads are classified as tncRNAs only if they map exclusively to the tRNA-derived artificial chromosomes, avoiding ambiguous assignments from non-tRNA regions.

## When to use

Use when detecting tRNA-derived non-coding RNAs (tRF-5, tRF-3, tRF-1, tRNA halves) from small RNA-seq and you need an index that forces tncRNA assignments to come only from tRNA loci plus 50 nt leader/trailer flanks.

## When NOT to use

- The analysis goal is general small RNA annotation (miRNA, piRNA, rRNA-derived reads) rather than tRNA-derived reads — a SPORTS1.0-style annotation pipeline is more appropriate
- The genome lacks reliable tRNA annotations or tRNAscan-SE cannot be run for the organism, so masking coordinates and mature/leader/trailer FASTAs cannot be generated
- Input data is paired-end or long-read RNA-seq rather than single-end small RNA reads of 14–50 nt, which is the size class this artificial genome design was built to classify

## Inputs

- Genome FASTA for the species (e.g. TAIR10.1 for Arabidopsis thaliana, SL4.0 nuclear / SL3.0 organellar for tomato) with '>chr[Num]', '>chrMt', '>chrPt' headers
- tRNA gene predictions from tRNAscan-SE (nuclear eukaryote model and '-O' organellar model)
- bed (or equivalent) coordinates of filtered genuine tRNA regions for masking

## Outputs

- Masked genome FASTA with tRNA regions masked plus 50 nt upstream/downstream flanks
- Mature, leader, and trailer tRNA FASTA files appended as artificial chromosomes
- Bowtie1 index stored in lib/indexes/<species name>
- Bowtie index files consumed by downstream tncRNA classification and RPM quantification

## How to apply

First predict tRNAs with tRNAscan-SE — nuclear tRNAs with the eukaryotes model and organellar tRNAs with the '-O' option — then eliminate tRNA pseudogenes and any tRNA with a score below 50. Using bedtools maskfasta, mask the filtered tRNA gene regions in the genome while retaining 50 bp flanks at both the 3' and 5' ends, and append the mature, leader, and trailer tRNA FASTA files as additional artificial chromosomes. Ensure FASTA headers follow the '>chr[Num]' convention (organellar as '>chrMt' and '>chrPt') so the toolkit can separate nuclear and organellar regions. Run bowtie-build on this composite artificial genome to create the species index under lib/indexes/<species name>, which downstream tncRNA identification then uses with bowtie '-v2 --best' for error-tolerant, uniquely mapping alignments. The rationale is that masking genuine tRNA regions in the genome plus flanking definitions lets the pipeline classify as tncRNAs only reads mapping exclusively to artificial chromosomes, and it enables subtype assignment (tRF-5, tRF-3, tRF-1, 5'tRH, 3'tRH) by mapped position relative to the mature tRNA and its flanks.

## Related tools

- **tRNAscan-SE** (Predicts nuclear (eukaryotes model) and organellar ('-O' option) tRNA genes whose filtered regions define the mask and flanks; pseudogenes and score <50 tRNAs are eliminated) — https://github.com/skbinfo/tncRNA-Toolkit
- **bedtools** (maskfasta script masks filtered tRNA regions with 50 bp 3' and 5' flanks to create the artificial genome)
- **Bowtie1** (bowtie-build creates the per-species index of the artificial genome; provided in the toolkit tar package (v1.3.0)) — https://github.com/skbinfo/tncRNA-Toolkit
- **tncRNA Toolkit** (Workflow wrapping index construction via 'python3 tncRNAs.py -g <genome fasta> -s <species name>') — https://github.com/skbinfo/tncRNA-Toolkit

## Examples

```
python3 tncRNAs.py -g genome.fa -s arabidopsis
```

## Evaluation signals

- All genomic FASTA headers start with '>chr[Num]' and organellar sequences use '>chrMt' and '>chrPt' so the toolkit automation can separate nuclear and organellar regions
- Masked genome retains exactly 50 nt upstream and 50 nt downstream flanks around each genuine tRNA region, and no unmasked genuine tRNA sequences remain in the main chromosomes
- Bowtie index directory lib/indexes/<species name> is created automatically after the '-g/-s' step, with mature, leader, and trailer tRNAs present as artificial chromosomes
- Downstream alignment with bowtie '-v2 --best' yields reads mapping exclusively to artificial chromosomes, and only those reads are classified as tncRNAs
- Resulting tncRNAs fall within the 14–50 nt size range reported across the 2,448 sRNA-seq samples of six angiosperms

## Limitations

- Genome FASTA header conventions ('>chr[Num]', '>chrMt', '>chrPt') are mandatory for the toolkit's automation; nonconforming headers break the workflow
- The design classifies reads as tncRNAs only if they map exclusively to artificial chromosomes, so ambiguous tncRNA-like reads from non-tRNA regions are deliberately excluded
- Conventional small RNA-seq suffers from tRNA modification biases, which can affect read recovery; the article notes switching to tRNA-seq is advantageous

## Evidence

- [results] To avoid ambiguous reads from non-tRNA regions, we created an artificial genome by masking the genuine tRNA gene region with 50 nt upstream and downstream: "To avoid ambiguous reads from non-tRNA regions, we created an artificial genome by masking the genuine tRNA gene region with 50 nt upstream and downstream"
- [methods] Pseudogene and score filtering of tRNA pool: "The tRNA pseudogenes and tRNAs with a score less than 50 were eliminated"
- [readme] Build artificial genome and index via tncRNAs.py: "python3 tncRNAs.py -g < genome fasta > -s < species name >"
- [readme] FASTA header requirements: "Genome fasta header should be start with '>chr[Num]', for example >chr1, >chr2 so on. Mitochondrial and plastid fasta headers also should be as >chrMt & >chrPt respectively."
- [results] Rationale: exclusive artificial chromosome mapping for tncRNA classification: "We classified only those reads as tncRNAs that mapped exclusively to the artificial chromosomes"
