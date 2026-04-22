# FRBR Hierarchy

The core of the MerMEId MeLODy data model follows the **FRBR** (Functional Requirements for Bibliographic Records) hierarchy, as formalised in [LRMoo](https://cidoc-crm.org/lrmoo/). This hierarchy separates a musical work into four conceptually distinct levels.

```
Work
 └── Expression         (a specific realisation of the work)
      └── Manifestation  (a published or distributed version of the expression)
           └── Item      (a single physical or digital copy)
```

Understanding this hierarchy is fundamental to working with MerMEId MeLODy, because it determines how entities relate to one another and which fields are appropriate at each level.

---

## Work

**Class:** `melod:Work`
**Folder:** `works/`

A **Work** represents the abstract intellectual or artistic creation — the composer's intention, independent of any particular realisation, notation, or performance. A single work may give rise to many different expressions.

> *Example:* Beethoven's Symphony No. 9 in D minor, Op. 125 is a Work. It exists as an abstract concept, identified by the composer's intention and the thematic catalogue entry — not tied to any specific score or recording.

**Fields:**

- Title (primary and alternative titles)
- Composer (linked Person as Contributor with role as Composer)
- Catalogue numbers (opus number, thematic catalogue reference such as BWV, TWV, etc.)
- Historic context of the work creation
- Genre and sub-genre as classifications
- Evidence as linked bibliography
- Related works (part of a larger collection, preceded by, derived from, etc.)
- Is realised in (linked Expression)

---

## Expression

**Class:** `melod:Expression`
**Folder:** `expressions/`

An **Expression** is a specific realisation of a Work or a Source — a distinct version defined by its musical content: instrumentation, arrangement, language, number of movements, and so on. The same abstract Work can have multiple Expressions.

> *Example:* An arrangement of Beethoven's 9th for piano solo is a different Expression from the full orchestral score. A revision by the composer that adds a new movement creates another Expression.

**Fields:**

- Label of this realisation
- Scoring/instrumentation (linked Instrumentation entity)
- Movements and sections (can reference other Expressions as components)
- Key, Meter, Tempo, mensuration and extent
- Duration of the performed expression
- Musical incipits (in PAE — Plaine and Easie Code — format or linked MEI)
- Performance events (linked PerformanceEvents)
- Related expressions (arrangements, translations, revisions)
- Completion status

---

## Manifestation

**Class:** `melod:Manifestation`
**Folder:** `manifestations/`

A **Manifestation** is a published or otherwise distributed embodiment of an Expression — an edition, print run, manuscript type, or digital publication. All copies produced from the same printing or distribution constitute one Manifestation.

> *Example:* The first edition of the score published by Breitkopf & Härtel in 1826 is a Manifestation. A later critical edition by another publisher is a separate Manifestation of the same Expression.

**Fields:**

- Title as it appears on the manifestation and link to identifier (RISM)
- Publisher and place/date of publication
- Physical description (dimensions, watermark, plate number, inscriptions) and formatted title pages (for manifestation singletons)
- Classifications
- Related manifestations (reprints, facsimile editions)
- Embodies (Linked Expression)

---

## Item

**Class:** `melod:Item`
**Folder:** `items/`

An **Item** is a single physical or digital copy of a Manifestation — a specific exemplar held at a particular institution or location. Two copies of the same printed edition are two different Items.

> *Example:* The copy of the 1826 Breitkopf & Härtel score held at the Staatsbibliothek Berlin, with shelfmark Mus.ms. 1234, is one Item.

**Fields:**

- Label and link to identifier (RISM)
- Holding institution/repository (linked Institution)
- Shelfmark and former shelfmark
- Provenance notes as events and date of acquisition
- Physical description (dimensions, watermark, plate number, inscriptions, condition) and formatted title pages
- Scribes (linked Persons and Medium description)
- Link to digitisation (if available)
- Classifications
- Related items (components)
- Materialization Of (Linked Manifestation)

---

## How the Levels Relate

In practice, not every catalogue entry requires all four levels. A thematic catalogue might focus primarily on Works and Expressions, using Items only when documenting specific sources. The editor does not enforce a strict hierarchy — you can create an Item without a parent Manifestation, for example — but the relational fields in each form are designed to connect the levels.

Links between levels are stored as RDF property references between entity IRIs, for example:

```turtle
<urn:uuid:expressions/1234567890>
    a melod:Expression ;
    melod:isRealisationOf <urn:uuid:works/9876543210> .
```
