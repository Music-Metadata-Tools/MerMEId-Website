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

The data model is centred on the [FRBR](https://en.wikipedia.org/wiki/Functional_Requirements_for_Bibliographic_Records) hierarchy as formalised in [LRMoo](https://cidoc-crm.org/lrmoo/), extended with contextual entities for persons, places, events, and bibliographic references. It maintains compatibility with the MEI (Music Encoding Initiative) XML Schema Version 5.1, ensuring interoperability with existing MEI-based systems and ensures MEI/XML as an output of the editor.

```
Work                     ← abstract creative work
 └── Expression          ← a specific realisation (score, arrangement, version)
      └── Manifestation  ← a published or distributed edition
           └── Item      ← a single physical or digital copy

Person  Place  Venue  Institution  PerformanceEvent  Event Instrumentation  Bibliography  Letter
```

All entity classes are defined in the [MeLODy ontology](https://lod.academy/melod/vocab/ontology#) (`melod:` → `https://lod.academy/melod/vocab/ontology#`).
