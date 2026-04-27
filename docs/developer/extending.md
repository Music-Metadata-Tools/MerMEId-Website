# Extending the Editor

MerMEId MeLODy is designed to be extended without modifying core components. Adding a new entity type requires only four steps: creating a SHACL shape, registering the type, creating the data folder, and optionally adding a search index and MEI/XML converter.

---

## At a Glance

```
Editor repository
└── configuration/
    ├── editor-default.ttl    ← which entity types exist and which SHACL shape each uses
    └── *.shacl               ← one shape file per entity type, defines form fields

Data repository
└── configuration/
    └── config.json           ← datasetBaseUrl and projectDomain
```

No build step is required for most configuration changes. Editing a SHACL shape or `editor-default.ttl` takes effect immediately on the next page load.

## Editor Configuration

The editor configuration file `configuration/editor-default.ttl` is a Turtle file loaded at startup. It tells the editor which entity types are available, what their display names are, where their files are stored, and which SHACL shape file defines their form.

---

## How It Works

When the editor starts, `js/index.js` fetches `editor-default.ttl` and loads its content into an in-browser [Oxigraph](https://github.com/oxigraph/oxigraph) triple store. A SPARQL SELECT query then extracts all entity type definitions:

```sparql
SELECT ?entity_type ?label ?entity_folder_name ?shacl_file_location WHERE {
  ?entity_type a melod:EntityType ;
        rdfs:label ?label ;
        melod_ui:entity_folder_name ?entity_folder_name ;
        melod_ui:shacl_file_location ?shacl_file_location .
}
```

The results are passed to the `adwlm-entity-editor` component as `entity_type_definitions`. This drives the "New entity" dialog and tells the editor which SHACL shape to load when a file of that type is opened.

---

## Registration Format

Each entity type is an RDF subject with four required properties:

```turtle
@prefix melod: <https://lod.academy/melod/vocab/ontology#> .
@prefix rdfs:  <http://www.w3.org/2000/01/rdf-schema#> .
@prefix melod_ui: <https://lod.academy/melod/vocab/ui/> .

melod:Person
    a melod:EntityType ;
    rdfs:label "Person" ;
    melod_ui:entity_folder_name "persons" ;
    melod_ui:shacl_file_location "configuration/person.shacl" .
```

### The Four Properties

| Property | Description |
| :--- | :--- |
| `a melod:EntityType` | Required type declaration — the SPARQL query matches on this |
| `rdfs:label` | Human-readable display name shown in the "New entity" dialog (e.g. `"Person"`) |
| `melod_ui:entity_folder_name` | Name of the folder in the data repository where files of this type are stored (e.g. `"persons"`) — **must** match the actual folder name exactly |
| `melod_ui:shacl_file_location` | Path or URL of the SHACL shape file for this entity type |

---

## SHACL File Locations

The `melod_ui:shacl_file_location` value can be either:

- A **relative path** within the editor repository: e.g. `"configuration/person.shacl"` — resolved against the editor's base URL at runtime
- An **absolute HTTPS URL**: e.g. `"https://example.org/shapes/person.shacl"` — fetched directly

Using an absolute URL allows you to host SHACL shapes on an external server or in the data repository and reference them without modifying the editor files. This is useful when managing multiple data repositories that share the same entity type definitions but may have different field configurations.

---

## Adding a New Entity Type

### Step 1 — Create a SHACL shape

Create a new `.shacl` file in `configuration/`. This is a Turtle file that defines the form fields for your entity type.

```turtle
@prefix sh:    <http://www.w3.org/ns/shacl#> .
@prefix melod: <https://lod.academy/melod/vocab/ontology#> .
@prefix xsd:   <http://www.w3.org/2001/XMLSchema#> .
@prefix skos:  <http://www.w3.org/2004/02/skos/core#> .

melod:MyNewTypeShape
    a sh:NodeShape ;
    sh:targetClass melod:MyNewType ;
    sh:property [
        sh:path skos:prefLabel ;
        sh:name "Name" ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:datatype xsd:string ;
    ] ;
    sh:property [
        sh:path <http://www.w3.org/2000/01/rdf-schema#comment> ;
        sh:name "Description" ;
        sh:datatype xsd:string ;
    ] .
```

Save the file as `configuration/mynewtype.shacl`.

See [SHACL Shapes](../configuration/shacl_shapes.md) for the full property reference.

---

### Step 2 — Register the entity type

Open `configuration/editor-default.ttl` and add a new block:

```turtle
melod:MyNewType
    a melod:EntityType ;
    rdfs:label "My New Type" ;
    melod_ui:entity_folder_name "mynewtypes" ;
    melod_ui:shacl_file_location "configuration/mynewtype.shacl" .
```

- Replace `melod:MyNewType` with the full class IRI from your ontology (e.g. `melod:MyNewType` if the class is defined in the MeLODy ontology, or any other IRI if you're using a different namespace)
- `rdfs:label` is the display name shown in the "New entity" dialog
- `melod_ui:entity_folder_name` must exactly match the folder name you will create in Step 3
- `melod_ui:shacl_file_location` can be a relative path or an absolute HTTPS URL

---

### Step 3 — Add a search index entry (optional)

If you want your new entity type to appear in the search panel, two things are needed:

**In the editor:** add an entry to `modules/entity-search/index.js` that includes the index file URL for your type (e.g. `mynewtype.ttl`). The search component fetches this file and loads it into Oxigraph.

**In the data repository CI/CD pipeline:** add a step that aggregates all `.ttl` files in `mynewtype/` into a single `datasets/mynewtype.ttl` index file. This is the file the search component fetches. The existing pipeline steps for `persons.ttl`, `works.ttl`, etc. serve as a template. You need to also add a SPARQL CONSTRUCT query, that created the index. Read more in the section about [Datasets Generation](datasets.md).

---

### Step 4 — Add a MEI/XML converter (optional)

If you want MEI/XML output in the XML tab for your new entity type, add:

1. A converter module at `modules/rdf-xml-converter/converters/mynewtype-converter.js`
2. An MEI/XML template string at `modules/rdf-xml-converter/templates/mynewtype-template.js`
3. A mapping entry in `modules/rdf-xml-converter/types/index.js`:

```js
import { myNewTypeConverter } from "./converters/mynewtype-converter.js";

export const converters = {
    // ... existing entries ...
    "https://lod.academy/melod/vocab/ontology#MyNewType": myNewTypeConverter,
};
```

The converter receives the JSON-LD object from `shacl-form.serialize("application/ld+json")` and must return a complete MEI/XML string.

---

## Modifying an Existing Form

To add, remove, or change fields in an existing entity type, edit the corresponding `.shacl` file in `configuration/`. No code changes are needed — the form is regenerated from the shape on the next page load.

**Common modifications:**

- **Add a field:** add a new `sh:property` block with `sh:path`, `sh:name`, `sh:datatype`, and `sh:order`
- **Make a field required:** set `sh:minCount 1`
- **Restrict to a single value:** set `sh:maxCount 1`
- **Add a dropdown:** set `sh:in ( "value1" "value2" "value3" )`
- **Add a linked entity field:** set `sh:class melod:SomeType` and `sh:nodeKind sh:IRI`
- **Reorder fields:** change the `sh:order` values

---

## Hosting SHACL Shapes Externally

The `melod:shacl_file_location` value in `editor-default.ttl` accepts any HTTPS URL. This allows you to host your SHACL shapes in a separate repository, on a CDN, or alongside your data repository, and reference them by URL:

```turtle
melod:MyNewType
    a melod:EntityType ;
    rdfs:label "My New Type" ;
    melod:entity_folder_name "mynewtype" ;
    melod:shacl_file_location "https://yourname.github.io/my-shapes/mynewtype.shacl" .
```

The editor fetches the shape file at runtime for each entity opened. Make sure the URL returns the correct `Content-Type` (`text/turtle` or `application/octet-stream`) and is accessible without authentication.
