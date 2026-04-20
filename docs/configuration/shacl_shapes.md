# SHACL Shapes

Forms in MerMEId MeLODy are generated automatically from [SHACL shapes](https://www.w3.org/TR/shacl/) by the [shacl-form](https://github.com/ulb-darmstadt/shacl-form) web component developed by UB Darmstadt. You do not write HTML or JavaScript to add or modify fields — defining them in the SHACL shape file is sufficient.

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
| `sh:name` | Label shown for the field in the form |
| `sh:description` | Help text displayed below the field |
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

melod:PersonShape
    a sh:NodeShape ;
    sh:targetClass melod:Person ;
    sh:property [
        sh:path schema:familyName ;
        sh:name "Surname" ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:datatype xsd:string ;
        sh:order 1 ;
    ] ;
    sh:property [
        sh:path schema:givenName ;
        sh:name "First name" ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:datatype xsd:string ;
        sh:order 2 ;
    ] ;
    sh:property [
        sh:path owl:sameAs ;
        sh:name "Link / sameAs" ;
        sh:description "Authority record URL (GND, VIAF, Wikidata, etc.)" ;
        sh:nodeKind sh:IRI ;
        sh:pattern "^http(s)?://.+?/..+?" ;
        sh:order 3 ;
    ] ;
    sh:property [
        sh:path schema:gender ;
        sh:name "Gender" ;
        sh:in ( "female" "male" "diverse" ) ;
        sh:maxCount 1 ;
        sh:order 4 ;
    ] ;
    sh:property [
        sh:path melod:birthPlace ;
        sh:name "Birth place" ;
        sh:class melod:Place ;
        sh:nodeKind sh:IRI ;
        sh:order 6 ;
    ] .
```

---

## Linked Entity Fields

Fields that reference other entities (e.g. "Birth place" → Place) use `sh:class` to specify the type of the linked entity. The editor populates the dropdown for such fields from the published dataset index.

The index URL is injected at runtime via the `dataset:` namespace patch (see below).

---

## The `dataset:` Namespace Patch

SHACL shapes reference published dataset indexes using a special `dataset:` namespace:

```turtle
sh:property [
    sh:path melod:birthPlace ;
    sh:class melod:Place ;
    melod:index <dataset:places.ttl> ;
] .
```

When the editor loads a repository, it reads `datasetBaseUrl` from `configuration/config.json` and replaces every occurrence of `dataset:` in the loaded SHACL shape with that URL. For example, if `datasetBaseUrl` is `https://yourname.github.io/my-catalogue/datasets/`, the property above becomes:

```turtle
melod:index <https://yourname.github.io/my-catalogue/datasets/places.ttl> ;
```

This allows the same SHACL shape files to work with any repository — only `config.json` needs to point to the correct published dataset.

---

## Adding a New Field

To add a field to an existing entity type, open the corresponding `.shacl` file and add a new `sh:property` block to the node shape. For example, to add a "Nickname" field to the Person shape:

```turtle
sh:property [
    sh:path schema:alternateName ;
    sh:name "Nickname" ;
    sh:description "Informal or abbreviated name" ;
    sh:datatype xsd:string ;
    sh:order 10 ;
] ;
```

Save the file and reload the editor — the new field will appear in all Person forms automatically.

---

## Further Reading

- [W3C SHACL Specification](https://www.w3.org/TR/shacl/)
- [shacl-form documentation and demo](https://ulb-darmstadt.github.io/shacl-form/#intro)
- [DoReMus ontology](https://data.doremus.org/ontology/) — for instrumentation and medium of performance properties
