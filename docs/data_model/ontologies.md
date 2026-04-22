# Ontologies

MerMEId MeLODy does not define everything from scratch — it builds on well-established vocabularies from the cultural heritage and semantic web communities, and adds a small domain-specific ontology for concepts not covered elsewhere.

---

## External Ontologies

| Ontology | Prefix | Used for |
| :--- | :--- | :--- |
| [LRMoo](https://cidoc-crm.org/lrmoo/) | `lrmoo:` | Core bibliographic hierarchy: Work, Expression, Manifestation, Item and their relationships |
| [CIDOC-CRM](https://cidoc-crm.org/) | `cidoc:` | Events, places, actors (persons and institutions as agents), temporal and spatial modelling |
| [Schema.org](https://schema.org/) | `schema:` | Persons (name, birth/death dates and places, gender, external identifiers), organisations, basic descriptive metadata |
| [DoReMus](https://data.doremus.org/ontology/) | `mus:` | Music-specific extensions: instrumentation, medium of performance, musical keys, casting |
| [SKOS](https://www.w3.org/TR/skos-reference/) | `skos:` | Preferred labels (`skos:prefLabel`) used for entity display names and search index entries |
| [OWL](https://www.w3.org/TR/owl2-overview/) | `owl:` | `owl:sameAs` for linking to external authority records (GND, VIAF, Wikidata, GeoNames, etc.) |
| [MEI](https://music-encoding.org/) | — | Music notation and encoding concepts; XML output tab converts to MEI format for interoperability |

---

## The MeLODy Ontology

MerMEId MeLODy defines its own [Music Metadata Ontology](https://lod.academy/melod/vocab/ontology#) (`melod:`) for concepts not covered by the standard vocabularies above — in particular the entity classes used in the editor and properties specific to the musicological use case. The concepts are based on the [MEI (Music Encoding Initiative)](https://music-encoding.org/guidelines/v5/content/index.html) guidelines and are mostly linked to the corresponding MEI elements. It implements the FRBR (Functional Requirements for Bibliographic Records) conceptual model, organizing musical information into Work, Expression, Manifestation, and Item levels. MeLOD integrates established ontologies including Schema.org, LRMoo, CIDOC-CRM and DoReMus to provide a rich semantic framework for music metadata.

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
    owl:sameAs <http://d-nb.info/gnd/11862119X> .
```
