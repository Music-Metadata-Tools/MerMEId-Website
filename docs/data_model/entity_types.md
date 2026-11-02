# Entity Types

MerMEId MeLODy supports 14 entity types, plus one data catalog per repository. Each type corresponds to an RDF class in the `melod:` ontology, a dedicated folder in the data repository, and a SHACL shape file that defines its form fields.

---

## All Entity Types at a Glance

| Entity Type | Folder | Class IRI | Description |
| :--- | :--- | :--- | :--- |
| Work | `works/` | `melod:Work` | Abstract intellectual or artistic creation |
| Expression | `expressions/` | `melod:Expression` | A specific realisation of a work or source |
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
| DataCollection | `dataCollections/` | `melod:DataCollection` | A named collection (work or source catalogue) belonging to the data catalog |

In addition, each repository contains exactly one **data catalog** (`dataCatalogs/catalog.ttl`, `melod:DataCatalog`), which is edited via the **Catalog metadata** button in the repository panel.

---

## FRBR Entities

### Work

**Class:** `melod:Work` | **Folder:** `works/`

The abstract intellectual or artistic creation — the composer's intention, independent of any particular realisation, notation, or performance. See [FRBR Hierarchy](frbr.md) for details.

**Key fields:** title (primary and alternative), composer (linked Person), catalogue numbers (opus, BWV, TWV, etc.), genre, related works, linked expression, data collections (file collections with citation and publication information).

---

### Expression

**Class:** `melod:Expression` | **Folder:** `expressions/`

A specific realisation of a Work defined by its musical content. See [FRBR Hierarchy](frbr.md) for details.

**Key fields:** title of this realisation, scoring/instrumentation (linked Instrumentation or inline medium of performance), key, meter, date of composition, scoring overview, movements and sections, musical incipits (PAE format), performance events (linked PerformanceEvents), related expressions, completion status.

---

### Manifestation

**Class:** `melod:Manifestation` | **Folder:** `manifestations/`

A published or otherwise distributed embodiment of an Expression. See [FRBR Hierarchy](frbr.md) for details.

**Key fields:** title as on the manifestation, publisher, place and date of publication, plate number, physical description and title pages (for manifestation singleton), related manifestations, linked items, linked expression, data collections (file collections with citation and publication information).

---

### Item

**Class:** `melod:Item` | **Folder:** `items/`

A single physical or digital copy of a Manifestation. See [FRBR Hierarchy](frbr.md) for details.

**Key fields:** holding institution (linked Institution), shelfmark and former shelfmark, RISM identifier, provenance notes, physical condition, link to digitisation, related items.

---

## Contextual Entities

### Person

**Class:** `melod:Person` | **Folder:** `persons/`

An individual relevant to the catalogue — composer, arranger, performer, editor, dedicatee, or any other biographical entity.

**Fields:**

| Field | RDF Property | Notes |
| :--- | :--- | :--- |
| Surname | `schema:familyName` | **Required** |
| First name | `schema:givenName` | |
| Alternate name | `schema:alternateName` | Alternative names or spellings |
| Link | `owl:sameAs` | GND, VIAF, Wikidata, etc. Must be a full HTTP(S) URL |
| Gender | `schema:gender` | Allowed values: `female`, `male`, `diverse` |
| Birth date | `schema:birthDate` | Single date (YYYY-MM-DD) or a date span |
| Birth place | `schema:birthPlace` | Linked entity dropdown (Place) |
| Death date | `schema:deathDate` | Same pattern as birth date |
| Death place | `schema:deathPlace` | Linked entity dropdown (Place) |
| Occupation | `schema:hasOccupation` | |
| Description | `schema:description` | |

---

### Institution

**Class:** `melod:Institution` | **Folder:** `institutions/`

An organisation relevant to the catalogue — publisher, library, archive, research institution, or performing organisation.

**Key fields:** name, abbreviated name, RISM siglum, place (linked Place), address, founding date, external authority link (`owl:sameAs`), description.

---

### Place

**Class:** `melod:Place` | **Folder:** `places/`

A geographic location — city, region, country — used as a reference point for persons, events, performances, institutions, expressions and venues.

**Key fields:** name, country, Link/SameAs (Geonames, GND, Wikidata, ...), description.

---

### Venue

**Class:** `melod:Venue` | **Folder:** `venues/`

A specific performance venue such as a concert hall, opera house, or church. More specific than Place — a Venue is located in a Place.

**Key fields:** name, place (linked Place), description, external link.

---

### PerformanceEvent

**Class:** `melod:PerformanceEvent` | **Folder:** `performanceEvents/`

A documented performance of a Expression. Links together performers, venues, dates, and the performed expression.

**Key fields:** date of performance (single date or date span), venue (linked Venue), contributors (linked Persons or Institutions with role and certainty), description, evidence of the performance (linked Bibliography).

---

### Instrumentation

**Class:** `melod:Instrumentation` | **Folder:** `instrumentations/`

A reusable scoring description that can be referenced by multiple Expressions. Defines the medium of performance (instruments, voices, ensemble type) independently of any specific work version.

**Key fields:** label (e.g. orchestra or SATB), medium of performance entries (instrument or voice type, count, solo or ad libitum, link to identifier), ensemble as part of a instrumentation (linked Instrumentation).

---

### Bibliography

**Class:** `melod:Bibliography` | **Folder:** `bibliography/`

A bibliographic reference to secondary literature — a book, journal article, chapter, or edition — that can be cited from Work, Expression, or other entity records.

**Key fields:** reference type (letter, article, book, website), author(s) (linked Person or Institution), editors, title in different levels, year, publisher / journal, volume/issue/pages, identifier (DOI, ISBN, etc.), link (to Zotero).

---

### Event

**Class:** `melod:Event` | **Folder:** `events/`

A general event entity for non-performance contexts — a conference, workshop, commissioning event, or premiere of a work without full performance documentation.

**Key fields:** name of the event, date (single date or date span), place (linked Place), description, contributors (linked Persons or Institutions with roles), evidence of the event (linked Bibliography).

---

### Letter

**Class:** `melod:Letter` | **Folder:** `letters/`

A letter or correspondence document. Used to document primary source material such as composers' letters that relate to specific works or expressions.

**Key fields:** title of the letter, sent and received (each with participant (linked Person), place (linked Place) and date), repository (linked Institution), shelfmark and former shelfmark, description. Letters are modelled with the [correspSearch](https://correspsearch.net/) vocabulary.

---

## Catalog Entities

### DataCollection

**Class:** `melod:DataCollection` | **Folder:** `dataCollections/`

A named collection of works or manifestations, e.g. a work catalogue or a source catalogue, belonging to the data catalog of the repository. The default collections `dataCollections/works.ttl` ("All works") and `dataCollections/manifestations.ttl` ("All manifestations") are created automatically; new works and manifestations are added to them.

**Key fields:** name, data catalog, type (work catalog or source catalog), restrictions of use, status (draft, candidate, approved, published), responsible.

---

### DataCatalog

**Class:** `melod:DataCatalog` | **File:** `dataCatalogs/catalog.ttl`

The data catalog describes the catalogue of the repository as a whole. There is exactly one per repository; it is created automatically and edited via the **Catalog metadata** button.

**Key fields:** name, contributors with roles (data manager, data contributor, editor, contributor, publisher), application ("MerMEId MeLODy"), description.

---

## Repository Folder Structure

All entity folders are present in a data repository created from the [MerMEId MeLODy Template](https://github.com/Music-Metadata-Tools/MerMEId-MeLODy-Template). In this Template you can find example .ttl files as well. The structure looks like this:

```
my-catalogue/
├── configuration/
│   └── config.json
├── dataCatalogs/
│   └── catalog.ttl
├── dataCollections/
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
├── letters/
├── indexes/                
└── modules/
    └── datasets-generator/
```

The GitHub template additionally contains the workflows in `.github/workflows/` and the data validation (`package.json`, `tests/`); the GitLab template contains a `.gitlab-ci.yml`. Folders for entity types without examples (e.g. `events/`, `letters/`) may be missing in the template — they are created when the first entity of that type is saved.

The folder names are defined in the configuration of the editor. Do not rename these folders.