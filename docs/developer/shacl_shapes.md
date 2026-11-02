# SHACL Shapes

Forms in MerMEId MeLODy are generated automatically from [SHACL shapes](https://www.w3.org/TR/shacl/) by the [shacl-form](https://github.com/ulb-darmstadt/shacl-form) web component developed by ULB Darmstadt (in the MerMEId fork [mermeid-shacl-form](https://github.com/Music-Metadata-Tools/mermeid-shacl-form)). You do not write HTML or JavaScript to add or modify fields — defining them in the SHACL shape file is sufficient.

---

## What is a SHACL Shape?

SHACL (Shapes Constraint Language) is a W3C standard for describing and validating RDF data. A "node shape" describes the expected structure of an RDF resource of a given type — its properties, their datatypes, cardinality, and allowed values.

In MerMEId MeLODy, SHACL shapes serve a dual purpose:

1. **Form generation** — `shacl-form` reads the shape and renders an interactive HTML form with the appropriate input widgets
2. **Data validation** — the shape constraints (required fields, allowed values, pattern matching) are checked when data is serialised

Each entity type has its own `.shacl` file, located in `configuration/` (in the editor repository or the data repository, depending on how `melod:shacl_file_location` is configured).

---

## Key SHACL Properties for Form Generation

The following properties control how fields appear and behave in the generated form:

| Property | Effect |
| :--- | :--- |
| `sh:name` | Label shown for the field in the form (one per UI language, e.g. `"Surname"@en, "Nachname"@de`) |
| `sh:description` | Help text displayed below the field (one per UI language) |
| `sh:path` | The RDF property this field represents (e.g. `schema:familyName`) |
| `sh:minCount` | Minimum number of values; `sh:minCount 1` makes the field required |
| `sh:maxCount` | Maximum number of values; `sh:maxCount 1` restricts to a single value |
| `sh:datatype` | Datatype of the value (e.g. `xsd:string`, `xsd:date`, `xsd:integer`) |
| `sh:nodeKind sh:IRI` | Value must be an IRI (used for `owl:sameAs` and linked entity fields) |
| `sh:in` | Fixed list of allowed values for an enum / dropdown field |
| `sh:class` | Class of the linked entity (for cross-reference fields) |
| `sh:or` | Allows a field to accept alternative shapes (e.g. a single date or a date span) |
| `sh:pattern` | Regex the value must match (e.g. `^http(s)?://.+?/..+?` for URL fields) |
| `sh:order` | Controls the order in which fields appear in the form |
| `sh:group` | Groups related fields under a shared heading |

---

## Example: Person Shape

The following is a simplified excerpt from `person.shacl`:

```turtle
@prefix sh:     <http://www.w3.org/ns/shacl#> .
@prefix melod:  <https://lod.academy/melod/vocab/ontology#> .
@prefix schema: <https://schema.org/> .
@prefix owl:    <http://www.w3.org/2002/07/owl#> .
@prefix xsd:    <http://www.w3.org/2001/XMLSchema#> .
@prefix skos:   <http://www.w3.org/2004/02/skos/core#> .
@prefix dataset: <https://adwmainz.pages.gitlab.rlp.net/nfdi4culture/cdmd/project_templates/mermeid-template/datasets/> .

melod:PersonShape
    a sh:NodeShape ;
    sh:targetClass melod:Person ;
    sh:property [
        sh:path schema:familyName ;
        sh:name "Surname"@en, "Nachname"@de ;
        sh:description "Surname of the person"@en, "Nachname der Person"@de ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:datatype xsd:string ;
        sh:group melod:General ;
    ] ;
    sh:property [
        sh:path schema:givenName ;
        sh:name "First name"@en, "Vorname"@de ;
        sh:maxCount 1 ;
        sh:group melod:General ;
    ] ;
    sh:property [
        sh:path owl:sameAs ;
        sh:name "Link"@en, "Link"@de ;
        sh:description "Identifier e.g. GND, VIAF"@en, "Identifier z.B. GND, VIAF"@de ;
        sh:nodeKind sh:IRI ;
        sh:pattern "^http(s)?://.+?/..+?" ;
        sh:group melod:General ;
    ] ;
    sh:property [
        sh:path schema:gender ;
        sh:name "Gender"@en, "Geschlecht"@de ;
        sh:in ( "female" "male" "diverse" ) ;
        sh:maxCount 1 ;
        sh:group melod:General ;
    ] ;
    sh:property [
        sh:path schema:birthPlace ;
        sh:name "Place"@en, "Ort"@de ;
        sh:class melod:PlaceEntity ;
        sh:maxCount 1 ;
        sh:group melod:Birth ;
    ] ;
    sh:targetClass melod:Person .

melod:General a sh:PropertyGroup ;
    rdfs:label "General Information"@en, "Allgemeine Informationen"@de .
melod:Birth a sh:PropertyGroup ;
    rdfs:label "Birth"@en, "Geburt"@de .
```

---

## Multilingual Labels

The editor supports the UI languages English (`en`) and German (`de`). The form receives the selected language via the `data-language` attribute and shows the labels that match it. Therefore all labels — `sh:name`, `sh:description`, the `rdfs:label` of property groups and of `sh:or` alternatives — should be given once per language with language tags (`@en`, `@de`).

---

## Linked Entity Fields

Fields that reference other entities (e.g. "Birth place" → Place) use `sh:class` to specify the type of the linked entity. The class is the **index class** of the target type, `melod:<Type>Entity` (e.g. `melod:PlaceEntity`, `melod:PersonEntity`, `melod:ExpressionEntity`), which is produced by the index queries (see [Datasets Generation](datasets.md)). The editor populates the dropdown for such fields from the repository's index files.

Fields that accept several entity types use `sh:or`:

```turtle
sh:or (
    [ sh:class melod:PersonEntity ; rdfs:label "Person"@en, "Person"@de ]
    [ sh:class melod:InstitutionEntity ; rdfs:label "Institution"@en, "Institution"@de ]
) ;
```

---

## How Dropdowns Are Populated

When a form is opened, the editor fetches the SHACL shape and appends the content of all files in the repository's `indexes/` folder to it. The combined Turtle is passed to `shacl-form` as a `blob:` URL. `shacl-form` then offers every entity typed with the requested class (e.g. `melod:PlaceEntity`) in the dropdown.

This allows the same SHACL shape files to work with any repository.

---

## Adding a New Field

To add a field to an existing entity type, open the corresponding `.shacl` file and add a new `sh:property` block to the node shape. For example, to add an "Honorific prefix" field to the Person shape:

```turtle
sh:property [
    sh:path schema:honorificPrefix ;
    sh:name "Honorific prefix"@en, "Namenszusatz"@de ;
    sh:description "Title or honorific, e.g. Dr."@en, "Titel, z.B. Dr."@de ;
    sh:datatype xsd:string ;
    sh:group melod:General ;
] ;
```

Save the file and reload the editor — the new field will appear in all Person forms automatically.
