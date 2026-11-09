---
name: gene-disease-association-data-ingestion
description: Use when building a local, offline-queryable gene/phenotype-disease association
  knowledge base from RDF sources (DisGeNET-RDF, HOOM, SIO) for tools like VIBE that
  map HPO terms to ranked candidate genes.
when_to_use_negative:
- The goal is to run a live external gene-prioritization web service (e.g. AMELIE)
  — privacy and availability are not guaranteed, and this skill builds a local store
  instead
- Input is raw variant-level data (VCF/structural variants) needing annotation filtering
  rather than gene-disease association ingestion
- Phenotype-gene matching is needed for non-SNV data types (CGH, Sanger, cytogenetics,
  RNA-seq, metabolomics), which the DisGeNET-based association set does not cover
edam_operation: http://edamontology.org/operation_3561
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3063
tools:
- name: Apache Jena
  role: Builds and loads the combined RDF sources into the TDB triple store and executes
    the SPARQL CONSTRUCT extraction
- name: VIBE
  role: Consumes the resulting TDB/HDT store to map HPO terms to CUIs and rank genes
    by highest GDA score
  repo: https://github.com/molgenis/vibe
- name: DisGeNET-RDF
  role: Core data source providing gene-disease associations mostly from literature
    and variant-disease associations from curated sources
- name: Orphadata HOOM
  role: Supplies additional phenotype-disease associations via HPO-ORDO matching
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/ggn2.10023
    title: A pipeline‐friendly software tool for genome diagnostics to prioritize
      genes by matching patient symptoms to literature
  source_content_hash: sha256:70860f79a65b0f59c6b512db89ce5379fef651954eba0ff9ff16a07cd63b735a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/molgenis/vibe
  tools:
  - Apache Jena
  - DisGeNET-RDF
  - Java
  - Orphadata HOOM
  - VIBE
  edam_operation: http://edamontology.org/operation_3561
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3063
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# gene-disease-association-data-ingestion

## Summary

Ingests gene-disease and phenotype-disease association sources (DisGeNET-RDF r6.0 core plus DisGeNET r5.0 pda.ttl/phenotype.ttl/void.ttl, SIO v1.43, Orphadata HOOM r1.3) into a local TDB triple store built with Apache Jena v3.12.0, yielding the queryable knowledge base behind VIBE's gene prioritization.

## When to use

Use when building a local, offline-queryable gene/phenotype-disease association knowledge base from RDF sources (DisGeNET-RDF, HOOM, SIO) for tools like VIBE that map HPO terms to ranked candidate genes.

## When NOT to use

- The goal is to run a live external gene-prioritization web service (e.g. AMELIE) — privacy and availability are not guaranteed, and this skill builds a local store instead
- Input is raw variant-level data (VCF/structural variants) needing annotation filtering rather than gene-disease association ingestion
- Phenotype-gene matching is needed for non-SNV data types (CGH, Sanger, cytogenetics, RNA-seq, metabolomics), which the DisGeNET-based association set does not cover

## Inputs

- DisGeNET r6.0 RDF representation (core gene-disease/variant-disease associations)
- DisGeNET r5.0 supplement files: pda.ttl, phenotype.ttl, void.ttl
- SIO (Semanticscience Integrated Ontology) v1.43
- Orphadata HOOM (HPO-ORDO Ontological Module) r1.3
- SPARQL CONSTRUCT query defining the minimized dataset

## Outputs

- TDB triple store built with Apache Jena v3.12.0
- Minimized dataset in TTL format extracted via SPARQL CONSTRUCT
- Final downloadable TDB store distributed as an HDT file with a sha256 checksum

## How to apply

Download the RDF representation of DisGeNET r6.0 as the core source, supplement it with pda.ttl, phenotype.ttl, and void.ttl from DisGeNET r5.0, then add SIO (Semanticscience Integrated Ontology, v1.43) and Orphadata HOOM (HPO-ORDO Ontological Module, r1.3) to cover phenotype-disease associations. Combine all sources into a TDB triple store built with Apache Jena (v3.12.0). Execute a SPARQL CONSTRUCT query on this triple store to extract a minimized dataset in TTL format, which removes unneeded triples while retaining the gene-disease associations and ontology mappings. Rebuild the final TDB triple store from the minimized TTL set with Apache Jena; per the repository distribution convention this is packaged as an HDT file accompanied by a sha256 checksum. At query time, VIBE maps HPO search terms to UMLS CUIs via SPARQL, retrieves matching GDAs, groups them by NCBI gene identifiers, and ranks genes by their highest GDA score.

## Related tools

- **Apache Jena** (Builds and loads the combined RDF sources into the TDB triple store and executes the SPARQL CONSTRUCT extraction)
- **VIBE** (Consumes the resulting TDB/HDT store to map HPO terms to CUIs and rank genes by highest GDA score) — https://github.com/molgenis/vibe
- **DisGeNET-RDF** (Core data source providing gene-disease associations mostly from literature and variant-disease associations from curated sources)
- **Orphadata HOOM** (Supplies additional phenotype-disease associations via HPO-ORDO matching)

## Examples

```
java -jar vibe-with-dependencies-<version>.jar -d -t vibe-<db-version>.hdt -w hp.owl -o results.tsv -p HP:0002996 -p HP:0001377
```

## Evaluation signals

- The final TDB store builds with Apache Jena without errors
- The store is fully queryable offline via SPARQL
- Per distribution convention, the packaged HDT file matches its sha256 checksum (as in vibe-<db-version>.hdt.tar.gz)
- SPARQL HPO-to-CUI mapping queries return valid UMLS concept unique identifiers for supplied HPO terms
- Retrieved GDAs group cleanly by NCBI gene identifiers with a highest GDA score per gene

## Limitations

- Version mixing is required: DisGeNET r6.0 core plus r5.0 supplement files (pda.ttl, phenotype.ttl, void.ttl), so the store reflects a specific combined release state
- VIBE shows a bias with over-represented genes (MECP2, KMT2D) at high positions, acknowledged to be resolved in upcoming versions
- The resulting association data applies to phenotype-gene matching for SNVs/indels and does not extend to structural variation, CGH, Sanger sequencing, cytogenetics, RNA-sequencing or metabolomics

## Evidence

- [methods] All these sources are combined into a TDB triple store, which is built using Apache Jena (v3.12.0): "All these sources are combined into a TDB triple store, which is built using Apache Jena (v3.12.0)"
- [methods] The core data source of VIBE v2.0 is the RDF (Resource Description Framework) representation of DisGeNET r6.0: [evidence span withheld — anchor status: not_found]
- [methods] we incorporated Orphadata HOOM (The HPO‐ORDO Ontological Module) r1.3, which adds additional phenotype‐disease associations: [evidence span withheld — anchor status: not_found]
- [methods] VIBE first maps the HPO search terms to CUIs (concept unique identifiers) from UMLS (Unified Medical Language System): [evidence span withheld — anchor status: not_found]
- [readme] Download and extract HDT (sha256 checksum) and run vibe with -t vibe-<db-version>.hdt: "Download and extract [HDT][hdt_download] ([sha256 checksum](./database/checksums/database.sha256))"
