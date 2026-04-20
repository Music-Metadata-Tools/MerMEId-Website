# Entity Types

MerMEId MeLODy supports 13 entity types. Each type corresponds to an RDF class in the `melod:` ontology, a dedicated folder in the data repository, and a SHACL shape file that defines its form fields.

---

## All Entity Types at a Glance

| Entity Type | Folder | Class IRI | Description |
| :--- | :--- | :--- | :--- |
| Work | `works/` | `melod:Work` | Abstract intellectual or artistic creation |
| Expression | `expressions/` | `melod:Expression` | A specific realisation of a work |
| Manifestation | `manifestations/` | `melod:Manifestation` | A published or distributed version of an expression |
| Item | `items/` | `melod:Item` | A single physical or digital copy |
| Person | `persons/` | `melod:Person` | An individual relevant to the catalogue |
| Institution | `institutions/` | `melod:Institution` | An organisation (publisher, archive, library) |
| Place | `places/` | `melod:Place` | A geographic location |
| Venue | `venues/` | `melod:Venue` | A specific performance venue |
| PerformanceEvent | `performanceEvents/` | `melod:PerformanceEvent` | A documented performance |
| Instrumentation | `instrumentations/` | `melod:Instrumentation` | A reusable scoring description |
| Bibliography | `bibliography/` | `melod:Bibliography` | A bibliographic reference |
| Event | `events/` | `melod:Event` | A general event (non-performance) |
| Letter | `letters/` | `melod:Letter` | A letter or correspondence document |

---

## FRBR Entities

### Work

**Class:** `melod:Work` | **Folder:** `works/`

The abstract intellectual or artistic creation — the composer's intention, independent of any particular realisation, notation, or performance. See [FRBR Hierarchy](frbr.md) for details.

**Key fields:** title (primary and alternative), composer (linked Person), catalogue numbers (opus, BWV, TWV, etc.), genre, key (tonic and mode), date of composition, scoring overview, related works.

---

### Expression

**Class:** `melod:Expression` | **Folder:** `expressions/`

A specific realisation of a Work defined by its musical content. See [FRBR Hierarchy](frbr.md) for details.

**Key fields:** title of this realisation, scoring/instrumentation (linked Instrumentation or inline medium of performance), movements and sections, musical incipits (PAE format), performance events (linked PerformanceEvents), related expressions, completion status.

---

### Manifestation

**Class:** `melod:Manifestation` | **Folder:** `manifestations/`

A published or otherwise distributed embodiment of an Expression. See [FRBR Hierarchy](frbr.md) for details.

**Key fields:** title as on the manifestation, publisher, place and date of publication, series and plate number, physical format and description, related manifestations (reprints, facsimiles), held items.

---

### Item

**Class:** `melod:Item` | **Folder:** `items/`

A single physical or digital copy of a Manifestation. See [FRBR Hierarchy](frbr.md) for details.

**Key fields:** holding institution (linked Institution), shelfmark / call number, RISM identifier, provenance notes, physical condition, link to digitisation.

---

## Contextual Entities

### Person

**Class:** `melod:Person` | **Folder:** `persons/`

An individual relevant to the catalogue — composer, arranger, performer, editor, dedicatee, or any other biographical entity.

**Fields:**

| Field | RDF Property | Notes |
| :--- | :--- | :--- |
| Surname | `schema:familyName` | **Required** |
| First name | `schema:givenName` | **Required** |
| Preferred label | `skos:prefLabel` | Used for search index and display |
| Link / sameAs | `owl:sameAs` | GND, VIAF, Wikidata, etc. Must be a full HTTP(S) URL |
| Gender | `schema:gender` | Allowed values: `female`, `male`, `diverse` |
| Birth date | `schema:birthDate` | Single date (YYYY or YYYY-MM-DD) or a date span |
| Birth place | `melod:birthPlace` | Linked entity dropdown (Place) |
| Death date | `schema:deathDate` | Same pattern as birth date |
| Death place | `melod:deathPlace` | Linked entity dropdown (Place) |

---

### Institution

**Class:** `melod:Institution` | **Folder:** `institutions/`

An organisation relevant to the catalogue — publisher, library, archive, research institution, or performing organisation.

**Key fields:** name, abbreviated name, place (linked Place), external authority link (`owl:sameAs`), description.

---

### Place

**Class:** `melod:Place` | **Folder:** `places/`

A geographic location — city, region, country, or venue area — used as a reference point for persons, events, and works.

**Key fields:** name (preferred and alternative), GeoNames link (`owl:sameAs`), geographic coordinates, broader place (linked Place, for hierarchical nesting).

---

### Venue

**Class:** `melod:Venue` | **Folder:** `venues/`

A specific performance venue such as a concert hall, opera house, or church. More specific than Place — a Venue is located in a Place.

**Key fields:** name, place (linked Place), capacity, description, external link.

---

### PerformanceEvent

**Class:** `melod:PerformanceEvent` | **Folder:** `performanceEvents/`

A documented performance of a Work or Expression. Links together performers, venues, dates, and the performed work.

**Key fields:** date of performance, venue (linked Venue), performers (linked Persons or Institutions), performed expression (linked Expression), programme notes, source reference.

---

### Instrumentation

**Class:** `melod:Instrumentation` | **Folder:** `instrumentations/`

A reusable scoring description that can be referenced by multiple Expressions. Defines the medium of performance (instruments, voices, ensemble type) independently of any specific work version.

**Key fields:** label, medium of performance entries (instrument or voice type + count + notes), total count, description.

---

### Bibliography

**Class:** `melod:Bibliography` | **Folder:** `bibliography/`

A bibliographic reference to secondary literature — a book, journal article, chapter, or edition — that can be cited from Work, Expression, or other entity records.

**Key fields:** author(s), title, year, publisher / journal, volume/issue/pages, edition, identifier (DOI, ISBN, etc.), link.

---

### Event

**Class:** `melod:Event` | **Folder:** `events/`

A general event entity for non-performance contexts — a conference, workshop, commissioning event, or premiere of a work without full performance documentation.

**Key fields:** title, date, place (linked Place), description, participants (linked Persons or Institutions).

---

### Letter

**Class:** `melod:Letter` | **Folder:** `letters/`

A letter or correspondence document. Used to document primary source material such as composers' letters that relate to specific works or expressions.

**Key fields:** sender (linked Person), recipient (linked Person), date, place of writing (linked Place), summary, archive location (linked Institution), shelfmark, digitisation link.

---

## Repository Folder Structure

All entity folders are present in a data repository created from the [MerMEId MeLODy Template](https://github.com/Music-Metadata-Tools/MerMEId-MeLODy-Template). The structure looks like this:

```
my-catalogue/
├── configuration/
│   ├── config.json
│   ├── editor-default.ttl
│   └── *.shacl
├── works/
├── expressions/
├── manifestations/
├── items/
├── persons/
├── institutions/
├── places/
├── venues/
├── performanceEvents/
├── instrumentations/
├── bibliography/
├── events/
└── letters/
```

The folder names are defined in `editor-default.ttl` and **must** match the `melod:entity_folder_name` values registered there. Do not rename these folders.

---

## Example Data

Rendered examples of entity records from the dataset:

- [Person example](https://examples-ttl-to-html-f7aa2e.pages.gitlab.rlp.net/person-example.html)
- [Place example](https://examples-ttl-to-html-f7aa2e.pages.gitlab.rlp.net/place-example.html)
- [Work example](https://examples-ttl-to-html-f7aa2e.pages.gitlab.rlp.net/work-example.html)
- [Expression example](https://examples-ttl-to-html-f7aa2e.pages.gitlab.rlp.net/expression-example.html)
