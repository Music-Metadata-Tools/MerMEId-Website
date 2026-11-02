# Extending the Editor

MerMEId MeLODy is designed to be extended with little changes to the core components. Adding a new entity type requires a SHACL shape, an entry in `editor-default.ttl`, and — to make it searchable and linkable — a SPARQL index query and an `INDEXES` entry. Optionally you can add a colour for the graph view and an MEI/XML converter.

---

## At a Glance

```
Editor repository
├── configuration/
│   ├── editor-default.ttl            ← which entity types exist and which SHACL shape each uses
│   └── *.shacl                       ← one shape file per entity type, defines form fields
├── modules/datasets-generator/
│   └── <folder>.sparql               ← index query per entity folder
├── modules/services/index-store-service.js  ← INDEXES: index files loaded for search and graph
└── modules/rdf-xml-converter/        ← optional MEI/XML converters

Data repository
├── configuration/
│   └── config.json                   ← datasetBaseUrl, projectDomain, IDprefix
└── indexes/                          ← generated index files
```

No build step is required for most configuration changes. Editing a SHACL shape or `editor-default.ttl` takes effect immediately on the next page load.

## Editor Configuration

The editor configuration file `configuration/editor-default.ttl` is a Turtle file loaded at startup. It tells the editor which entity types are available, what their display names are, where their files are stored, and which SHACL shape file defines their form.

---

## How It Works

When the editor starts, `js/index.js` fetches `editor-default.ttl` and loads its content into an in-browser [Oxigraph](https://github.com/oxigraph/oxigraph) triple store. A SPARQL SELECT query then extracts all entity type definitions with the label in the current UI language:

```sparql
SELECT ?entity_type ?entity_name ?entity_folder_name ?shacl_file_location WHERE {
  ?entity_type a melod:Entity ;
        rdfs:label ?entity_name ;
        melod_ui:entity_folder_name ?entity_folder_name ;
        melod_ui:shacl_file_location ?shacl_file_location .
  FILTER (lang(?entity_name) = '<current UI language>')
}
```

The results are passed to the `adwlm-entity-editor`, `adwlm-filesystem-manager` and `adwlm-graph-view` components as `entity_type_definitions`. This drives the "New entity" dialog and tells the editor which SHACL shape to load when a file of that type is opened.

---

## Registration Format

Each entity type is an RDF subject with four required properties:

```turtle
@prefix melod: <https://lod.academy/melod/vocab/ontology#> .
@prefix rdfs:  <http://www.w3.org/2000/01/rdf-schema#> .
@prefix melod_ui: <https://lod.academy/melod/vocab/ui/> .

melod:Person a melod:Entity ;
    rdfs:label "Person"@en , "Person"@de ;
    melod_ui:entity_folder_name "persons" ;
    melod_ui:shacl_file_location "configuration/person.shacl" .
```

### The Four Properties

| Property | Description |
| :--- | :--- |
| `a melod:Entity` | Required type declaration — the SPARQL query matches on this |
| `rdfs:label` | Human-readable display name shown in the "New entity" dialog — one label per UI language (`@en`, `@de`) |
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

Create a new `.shacl` file in `configuration/`. This is a Turtle file that defines the form fields for your entity type. Give labels and descriptions in every UI language.

```turtle
@prefix sh:     <http://www.w3.org/ns/shacl#> .
@prefix melod:  <https://lod.academy/melod/vocab/ontology#> .
@prefix rdfs:   <http://www.w3.org/2000/01/rdf-schema#> .
@prefix xsd:    <http://www.w3.org/2001/XMLSchema#> .
@prefix schema: <https://schema.org/> .

melod:MyNewType
    a sh:NodeShape, rdfs:Class ;
    sh:property [
        sh:path schema:name ;
        sh:name "Name"@en, "Name"@de ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:datatype xsd:string ;
    ] ;
    sh:property [
        sh:path schema:description ;
        sh:name "Description"@en, "Beschreibung"@de ;
        sh:datatype xsd:string ;
    ] ;
    sh:targetClass melod:MyNewType .
```

Save the file as `configuration/mynewtype.shacl`.

See [SHACL Shapes](shacl_shapes.md) for the full property reference.

---

### Step 2 — Register the entity type

Open `configuration/editor-default.ttl` and add a new block:

```turtle
melod:MyNewType a melod:Entity ;
    rdfs:label "My New Type"@en , "Mein neuer Typ"@de ;
    melod_ui:entity_folder_name "mynewtypes" ;
    melod_ui:shacl_file_location "configuration/mynewtype.shacl" .
```

- Replace `melod:MyNewType` with the full class IRI from your ontology (e.g. `melod:MyNewType` if the class is defined in the MeLODy ontology, or any other IRI if you're using a different namespace)
- `rdfs:label` is the display name shown in the "New entity" dialog; without a label in the current UI language the type is not shown
- `melod_ui:entity_folder_name` must exactly match the folder name you will create in Step 3
- `melod_ui:shacl_file_location` can be a relative path or an absolute HTTPS URL

---

### Step 3 — Add a dataset and search index (optional)

If you want your new entity type to appear in the search panel, in the graph view or want to cross reference an entity, you need an index query and an entry in the index list.

#### Create a SPARQL query

Add a new `.sparql` file in `modules/datasets-generator/` of the **editor** repository. The file name must equal the folder name (`mynewtypes.sparql`): the editor uses it to update `indexes/mynewtypes.ttl` in the data repository whenever an entity is saved, and the CI/CD pipeline of the data repository fetches it from the editor repository.

```sparql
prefix melod: <https://lod.academy/melod/vocab/ontology#>
prefix skos: <http://www.w3.org/2004/02/skos/core#>
prefix schema: <https://schema.org/>

construct {
  ?iri a melod:MyNewTypeEntity ;
    skos:prefLabel ?label .
} where {
  select ?iri ?label where {
    ?iri a melod:MyNewType .
    ?iri schema:name ?label .
  }
  order by ?label
}
```

Key requirements:
- Use `CONSTRUCT` (not `SELECT`) to output RDF
- Always include `skos:prefLabel` — the search component and the dropdowns use this property
- Type the index entries as `melod:<Type>Entity` (e.g. `melod:MyNewTypeEntity`) — linked entity fields in the SHACL shapes refer to this class
- Optionally add `skos:altLabel` and `skos:broader` for the search, and `skos:related` for the graph view
- Use `OPTIONAL` for any properties that may not be present on every entity

#### Register in the editor

Add an entry to `INDEXES` in `modules/services/index-store-service.js`:

```js
{ name: "MyNewType", url: "mynewtypes.ttl", labels: { en: "My New Type", de: "Mein neuer Typ" } },
```

Without this entry, the index is not loaded for the search, the type filter and the graph view. (Linked entity dropdowns work independently, because the editor appends all files from `indexes/` to the shapes.)

Optionally add a colour for the new type to `TYPE_COLORS` in `modules/graph-view/index.js`.

#### Update the CI/CD pipeline of the data repository

Add the query name to the list in `modules/datasets-generator/fetch-sparql-queries.sh` and add an entry to `modules/datasets-generator/datasets-generator.sh` of the data repository:

```bash
echo "construct the mynewtypes dataset"
time /static-publishing-backend rdf-data-aggregator \
  $CI_PROJECT_DIR/mynewtypes/ "*.ttl" \
  $datasets_generator_dir_path/mynewtypes.sparql \
  $indexes_dir_path/mynewtypes.ttl
```

#### Update SHACL configuration

If the entity type is used in form dropdowns (i.e. as a linked entity field in another shape), reference the index class in the relevant SHACL shapes: `sh:class melod:MyNewTypeEntity`.

#### Test locally

Dataset generation can be tested locally without pushing (from the root of the data repository, after fetching the queries into `sparql-queries/`):

```bash
sh modules/datasets-generator/fetch-sparql-queries.sh
docker run --rm -v $(pwd):/repo -e GITHUB_WORKSPACE=/repo \
  registry.gitlab.rlp.net/adwmainz/nfdi4culture/cdmd/static-publishing-backend/static-publishing-backend:latest \
  /repo/modules/datasets-generator/datasets-generator.sh
```

Generated files will appear in `indexes/`. See [Datasets Generation](datasets.md) for background on the pipeline and SPARQL query reference.

---

### Step 4 — Add a MEI/XML converter (optional)

If you want MEI/XML output in the XML tab for your new entity type, add:

1. A converter module at `modules/rdf-xml-converter/converters/mynewtype.js`
2. An MEI/XML template string at `modules/rdf-xml-converter/templates/mynewtype-template.js`
3. A mapping entry in `modules/rdf-xml-converter/types/index.js`:

```js
import { myNewTypeConverter } from "./converters/mynewtype.js";

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

- **Add a field:** add a new `sh:property` block with `sh:path`, `sh:name` (one per language, e.g. `"Name"@en, "Name"@de`), `sh:datatype`, and `sh:order` or `sh:group`
- **Make a field required:** set `sh:minCount 1`
- **Restrict to a single value:** set `sh:maxCount 1`
- **Add a dropdown:** set `sh:in ( "value1" "value2" "value3" )`
- **Add a linked entity field:** set `sh:class melod:SomeTypeEntity` (the class of the index entries) and `sh:nodeKind sh:IRI`
- **Reorder fields:** change the `sh:order` values

---

## Hosting SHACL Shapes Externally

The `melod:shacl_file_location` value in `editor-default.ttl` accepts any HTTPS URL. This allows you to host your SHACL shapes in a separate repository, on a CDN, or alongside your data repository, and reference them by URL:

```turtle
melod:MyNewType
    a melod:Entity ;
    rdfs:label "My New Type" ;
    melod:entity_folder_name "mynewtype" ;
    melod:shacl_file_location "https://yourname.github.io/my-shapes/mynewtype.shacl" .
```

The editor fetches the shape file at runtime for each entity opened. Make sure the URL returns the correct `Content-Type` (`text/turtle` or `application/octet-stream`) and is accessible without authentication.
