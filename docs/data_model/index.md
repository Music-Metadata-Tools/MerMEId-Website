# Data Model

MerMEId MeLODy uses a Linked Open Data model based on established musicological and bibliographic ontologies. All metadata is stored as [RDF/Turtle](https://www.w3.org/TR/turtle/) files, one per entity, in a Git repository.

This section documents the conceptual and technical foundations of the data model.

---

## Sections

- [Ontologies](ontologies.md) — the external vocabularies the data model is built on, and the MeLODy ontology
- [FRBR Hierarchy](frbr.md) — the core bibliographic hierarchy: Work, Expression, Manifestation, Item
- [Entity Types](entity_types.md) — all 13 supported entity types with fields and repository structure

---

## At a Glance

The data model is centred on the [FRBR](https://en.wikipedia.org/wiki/Functional_Requirements_for_Bibliographic_Records) hierarchy as formalised in [LRMoo](https://cidoc-crm.org/lrmoo/) and directly aligned with the [MEI (Music Encoding Initiative)](https://music-encoding.org/) XML Schema (v5.1). The RDF structure is specifically designed to map seamlessly to MEI elements, ensuring that complex thematic work and source catalogue metadata can be reliably transformed into well-formed MEI/XML. Extended with contextual entities for persons, places, events, and bibliographic references, the model ensures both semantic precision and MEI standards compliance.

```
Work                     ← abstract creative work
 └── Expression          ← a specific realisation (score, arrangement, version)
      └── Manifestation  ← a published or distributed edition
           └── Item      ← a single physical or digital copy

Person  Place  Venue  Institution  PerformanceEvent  Event Instrumentation  Bibliography  Letter
```

All entity classes are defined in the [MeLODy ontology](https://lod.academy/melod/vocab/ontology#) (`melod:` → `https://lod.academy/melod/vocab/ontology#`).
