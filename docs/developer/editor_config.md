# Configuration

MerMEId MeLODy is configured at two distinct levels: the **editor** itself (which entity types are available and how their forms look) and the **data repository** (IRI scheme and ID prefixes). SHACL shape files provide a third layer of configuration, driving the form fields for each entity type.

---


## At a Glance

```
Editor repository
├── configuration/
│   ├── editor-default.ttl    ← which entity types exist and which SHACL shape each uses
│   └── *.shacl               ← one shape file per entity type, defines form fields
└── modules/datasets-generator/
    └── <folder>.sparql       ← one SPARQL query per entity folder, builds the index files

Data repository
├── configuration/
│   └── config.json           ← datasetBaseUrl, projectDomain and optional IDprefix
└── indexes/
    └── <folder>.ttl          ← generated index files (search and dropdowns)
```

No build step is required for most configuration changes. Editing a SHACL shape or `editor-default.ttl` takes effect immediately on the next page load.

## Editor Configuration

The editor configuration file `configuration/editor-default.ttl` is a Turtle file loaded at startup. It tells the editor which entity types are available, what their display names are in each UI language, where their files are stored, and which SHACL shape file defines their form.

---

## How It Works

When the editor starts, `js/index.js` fetches `editor-default.ttl` and loads its content into an in-browser [Oxigraph](https://github.com/oxigraph/oxigraph) triple store. A SPARQL SELECT query then extracts all entity type definitions for the current UI language:

```sparql
SELECT ?type ?label ?folder ?shacl WHERE {
  ?type a melod:EntityType ;
        rdfs:label ?label ;
        melod:entity_folder_name ?folder ;
        melod:shacl_file_location ?shacl .
}
```

The results are passed to the `adwlm-entity-editor`, `adwlm-filesystem-manager` and `adwlm-graph-view` components as `entity_type_definitions`. This drives the "New entity" dialog, tells the editor which SHACL shape to load when a file of that type is opened, and determines which folders are shown in the repository tree. The query is run again whenever the UI language is changed.

---

## Registration Format

Each entity type is an RDF subject with four required properties:

```turtle
@prefix melod: <https://lod.academy/melod/vocab/ontology#> .
@prefix rdfs:  <http://www.w3.org/2000/01/rdf-schema#> .

melod:Person a melod:Entity ;
    rdfs:label "Person"@en , "Person"@de ;
    melod:entity_folder_name "persons" ;
    melod:shacl_file_location "configuration/person.shacl" .
```

### The Four Properties

| Property | Description |
| :--- | :--- |
| `a melod:Entity` | Required type declaration — the SPARQL query matches on this |
| `rdfs:label` | Human-readable display name shown in the "New entity" dialog (e.g. `"Person"@en`). One label per UI language (`@en`, `@de`) is required — a type without a label in the current language does not appear in the editor |
| `melod:entity_folder_name` | Name of the folder in the data repository where files of this type are stored (e.g. `"persons"`) — **must** match the actual folder name exactly |
| `melod:shacl_file_location` | Path or URL of the SHACL shape file for this entity type |

The data catalog (`melod:DataCatalog`, `dataCatalogs/catalog.ttl`) is not registered in `editor-default.ttl`; it is edited via the catalog metadata dialog with `configuration/dataCatalog.shacl`.

---

## SHACL File Locations

The `melod:shacl_file_location` value can be either:

- A **relative path** within the editor repository: e.g. `"configuration/person.shacl"` — resolved against the editor's base URL at runtime
- An **absolute HTTPS URL**: e.g. `"https://example.org/shapes/person.shacl"` — fetched directly

Using an absolute URL allows you to host SHACL shapes on an external server or in the data repository and reference them without modifying the editor files. This is useful when managing multiple data repositories that share the same entity type definitions but may have different field configurations.

---

## Adding a New Entity Type

To add a custom entity type to the editor:

### Step 1 — Create a SHACL shape

Create a `.shacl` file (which is a valid Turtle file) in the `configuration/` folder. Define the node shape for your entity type. See [SHACL Shapes](shacl_shapes.md) for the properties to use.

### Step 2 — Register the entity type

Add a new block to `configuration/editor-default.ttl`:

```turtle
melod:MyNewType a melod:Entity ;
    rdfs:label "My New Type"@en , "Mein neuer Typ"@de ;
    melod:entity_folder_name "mynewtype" ;
    melod:shacl_file_location "configuration/mynewtype.shacl" .
```

Replace `melod:MyNewType` with the full class IRI from your ontology. Update the labels, folder name, and shape file path accordingly.

### Step 3 — Create the data folder

Create the folder (e.g. `mynewtype/`) in the data repository. If it does not exist, it is created when the first entity of this type is saved.

### Step 4 — Add a search index entry (optional)

If you want the new entity type to be searchable, add a SPARQL query `modules/datasets-generator/mynewtype.sparql` to the editor and an entry for its index file to `INDEXES` in `modules/services/index-store-service.js`. See [Extending the Editor](extending.md) for details.

No changes to the core editor components are needed for the form — it is generated automatically from the SHACL shape.

---

## Important Notes

- The `entity_folder_name` must **exactly** match the folder name in the data repository. A mismatch means the editor cannot find existing files of that type in the tree and will write new files to the wrong path.
- The SHACL shape is **not cached** between sessions — it is fetched fresh each time a file of that type is opened.
- Entity type definitions from `editor-default.ttl` are loaded once at startup. After editing this file, reload the page.
