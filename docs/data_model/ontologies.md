# Ontologies

MerMEId MeLODy does not define everything from scratch — it builds on well-established vocabularies from the cultural heritage and semantic web communities, and adds a small domain-specific ontology for concepts not covered elsewhere.

---

## External Ontologies

| Ontology | Prefix | Used for |
| :--- | :--- | :--- |
| [LRMoo](https://cidoc-crm.org/lrmoo/) / [FRBRoo](https://erlangen-crm.org/efrbroo/) | — | Core bibliographic hierarchy: Work, Expression, Manifestation, Item and their relationships |
| [CIDOC-CRM](https://cidoc-crm.org/) | `crm:` | Events, places, actors (persons and institutions as agents), temporal and spatial modelling |
| [Schema.org](https://schema.org/) | `schema:` | Persons (name, birth/death dates and places, gender, external identifiers), organisations, basic descriptive metadata |
| [DoReMus](https://data.doremus.org/ontology/) | — | Music-specific extensions: instrumentation, medium of performance, musical keys, casting |
| [SKOS](https://www.w3.org/TR/skos-reference/) | `skos:` | Preferred labels (`skos:prefLabel`) used for entity display names and search index entries |
| [OWL](https://www.w3.org/TR/owl2-overview/) | `owl:` | `owl:sameAs` for linking to external authority records (GND, VIAF, Wikidata, GeoNames, etc.) |
| [MEI](https://music-encoding.org/) | — | Music notation and encoding concepts; XML output tab converts to MEI format for interoperability |

---

## The MeLODy Ontology

MerMEId MeLODy defines its own ontology (`melod:`) for concepts not covered by the standard vocabularies above — in particular the entity classes used in the editor and properties specific to the musicological use case.

**Ontology IRI:** `https://lod.academy/melod/vocab/ontology#`
**Prefix:** `melod:`

All entity types in the editor (Person, Place, Work, Expression, etc.) are subclasses or instances defined within this ontology. When you see a class or property prefixed with `melod:` in a Turtle file, it comes from here.

### Example: a Person entity in Turtle

```turtle
@prefix melod:  <https://lod.academy/melod/vocab/ontology#> .
@prefix schema: <https://schema.org/> .
@prefix owl:    <http://www.w3.org/2002/07/owl#> .
@prefix skos:   <http://www.w3.org/2004/02/skos/core#> .

<urn:uuid:persons/2847362910>
    a melod:Person ;
    schema:familyName "Telemann" ;
    schema:givenName "Georg Philipp" ;
    schema:gender "male" ;
    skos:prefLabel "Georg Philipp Telemann" ;
    owl:sameAs <http://d-nb.info/gnd/11862119X> .
```

The `skos:prefLabel` is used by the search index to make this entity findable by name. The `owl:sameAs` links to the GND authority record, enabling integration with external datasets.
