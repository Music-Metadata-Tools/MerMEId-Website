# Component Reference

MerMEId MeLODy consists of three web components, an orchestration script, and a converter module. The web components communicate exclusively through custom DOM events — there are no direct references between components.

---

## Orchestration Layer (`js/index.js`)

This is the top-level script that bootstraps the application. It is not a web component — it runs as a plain ES module loaded from `index.html`. The startup sequence is described in detail in the [Architecture Overview](architecture.md).

Beyond startup, `js/index.js` is also responsible for wiring the **Output Panel** tabs:

- **XML tab** — listens for serialised entity data and invokes the appropriate converter from `modules/rdf-xml-converter/` to produce MEI/XML output
- **RDF tab** — displays the raw Turtle from `shacl-form.serialize()`
- **Preview tab** — renders the entity read-only in a second `shacl-form` in `data-view` mode
- Handles the copy buttons for all three tabs

---


## `adwlm-filesystem-manager`

**File:** `modules/filesystem-manager/index.js`

The left panel component. Manages the repository tree, the staged files queue, and all Git operations exposed to the user.

### Responsibilities

- Display and navigate the repository tree (lazily loaded via `sl-tree`)
- Clone, remove, rename, and synchronise repositories
- Stage, unstage, and push (commit + push) modified files
- Highlight unsaved and unshared files in the tree using colour indicators
- Load the selected `.ttl` file and dispatch it to the entity editor
- Respond to save requests from the entity editor

### Key Properties

| Property | Type | Description |
| :--- | :--- | :--- |
| `_selected_repository_path` | `String` | Absolute path of the active repository (e.g. `/my-catalogue`) |
| `_file_path` | `String` | Relative path of the currently open file |
| `_staged_files` | `Array` | Relative paths of files staged for the next commit |
| `entity_to_save` | `Object` | Set externally to trigger a file save; observed via `updated()` |

### Events Dispatched

| Event name | When | Payload |
| :--- | :--- | :--- |
| `adwlm-filesystem-manager:entity-to-edit` | File selected in the tree | `{ contents, path }` |
| `adwlm-filesystem-manager:repository-selected` | Repository clicked or auto-selected | `{ repositoryPath }` |
| `adwlm-filesystem-manager:clear-entity-editor` | Entity file deleted | — |
| `adwlm-filesystem-manager:reload-indexes` | After a push or pull | — |

### Events Consumed

| Event name | Source | Effect |
| :--- | :--- | :--- |
| `adwlm-entity-editor:unsaved-changes` | Entity editor | Blocks synchronise if there are unsaved changes |
| `adwlm-entity-editor:entity-to-delete` | Entity editor | Deletes the selected entity file |
| `entity-selected` | `window` | Navigates the tree to the given file path |

### Repository Tree

The tree is lazily loaded. When a `sl-tree-item[lazy]` node is expanded, the `sl-lazy-load` event fires and `_generate_folder_tree()` reads the contents of that folder from the virtual filesystem. Each tree item carries its path in `data-*` attributes:

- `data-entry-type`: `repo-folder`, `folder`, or `file`
- `data-entry-absolute-path`: e.g. `/my-catalogue/persons/12345.ttl`
- `data-entry-relative-path`: e.g. `persons/12345.ttl`

### Conflict Detection

Before a push or pull, `VirtualFilesystem.canPullSafely()` is called. It fetches the remote HEAD, finds the common ancestor commit, and checks whether any locally staged files have also been modified remotely. If conflicts are detected, the operation is blocked and the user is warned. See [Services](services.md) for details on `canPullSafely()`.

---

## `adwlm-entity-editor`

**File:** `modules/entity-editor/index.js`

The main editing area. Wraps the `shacl-form` component and manages the lifecycle of the currently open entity.

### Responsibilities

- Render the `shacl-form` for the currently selected entity
- Load the correct SHACL shape for the entity type (patched with the repository namespace)
- Track unsaved changes and guard against accidental navigation
- Dispatch save requests back to the filesystem manager
- Handle "quick add" — creating a new linked entity from within an open form
- Show linked entities in a draggable, floating side panel (read-only preview)

### Key Properties

| Property | Type | Description |
| :--- | :--- | :--- |
| `entity_to_edit` | `Object` | Currently open entity: `{ contents, path, entity_iri, entity_type, shapesUrl }` |
| `entity_type_definitions` | `Array` | Entity type definitions loaded from `editor-default.ttl` |
| `_hasUnsavedChanges` | `Boolean` | Set to `true` on any `change` event from `shacl-form` |
| `_selected_repository_path` | `String` | Kept in sync with the filesystem manager via events |
| `_cachedConfig` | `Object` | Cached result of `configuration/config.json` — cleared on repository change |

### Events Dispatched

| Event name | When | Payload |
| :--- | :--- | :--- |
| `adwlm-entity-editor:entity-to-save` | Save button clicked | `{ entity_iri, rdf_contents, json_ld_contents, path, shapesUrl }` |
| `adwlm-entity-editor:unsaved-changes` | `_hasUnsavedChanges` changes | `{ hasUnsavedChanges }` |
| `adwlm-quick-add:entity-to-save` | Quick-add form saved | same payload as `entity-to-save` |

### Events Consumed

| Event name | Source | Effect |
| :--- | :--- | :--- |
| `adwlm-filesystem-manager:entity-to-edit` | Filesystem manager | Loads the entity into the form |
| `adwlm-filesystem-manager:clear-entity-editor` | Filesystem manager | Clears the form |
| `adwlm-filesystem-manager:repository-selected` | Filesystem manager | Updates `_selected_repository_path`, clears config cache |
| `adwlm-entity-types-dialog:entity-to-add` | Entity types dialog | Creates a new empty entity of the selected type |
| `shacl-form:quick-add` | `shacl-form` shadow DOM | Opens the quick-add dialog for a linked entity |

### Entity ID Generation

New entity IRIs are generated at creation time using the Web Crypto API:

```js
_generate_entity_id() {
    let array = new Uint32Array(1);
    self.crypto.getRandomValues(array);
    return array[0]; // random 32-bit unsigned integer
}
```

This produces paths like `persons/2847362910.ttl` and IRIs like `urn:uuid:persons/2847362910`.

### Side Panel (Linked Entity Preview)

When a user clicks a link inside a `shacl-form` field (links are rendered as `<a class="uuid">`), the editor intercepts the click using `composedPath()` to pierce the shadow DOM, reads the linked `.ttl` file from the virtual filesystem, and renders it in a floating, draggable panel using a second `shacl-form` instance in read-only `data-view` mode.

### Repository Configuration Cache

On first load of a repository, the editor reads `configuration/config.json`. The result is cached in `_cachedConfig` and cleared when a different repository is selected. This avoids repeated filesystem reads for every entity opened in the same session.

---

## `adwlm-entity-search`

**File:** `modules/entity-search/index.js`

The search panel. Provides full-text search across all entity labels in the active repository.

### Responsibilities

- Fetches pre-built dataset index files (one `.ttl` per entity type) from `datasetBaseUrl`
- Parses all index files into an in-memory Oxigraph RDF store
- Runs a SPARQL query to extract all `skos:prefLabel` values and their types
- Filters results by label text and/or entity type
- On selection, navigates the filesystem manager tree to the matching file

### How Indexes Work

The search component does **not** index files from the local virtual filesystem. Instead, it fetches pre-built aggregated index files from the `datasetBaseUrl` — a published static URL, typically GitHub/GitLab Pages. These files are generated by a CI/CD pipeline when changes are pushed to the repository.

**Implication:** the search index reflects the **last pushed state**, not local unsaved changes. After a push, the filesystem manager dispatches `adwlm-filesystem-manager:reload-indexes` to trigger a fresh reload of all indexes.

### SPARQL Query

```sparql
SELECT ?subject ?label ?type WHERE {
  ?subject <http://www.w3.org/2004/02/skos/core#prefLabel> ?label .
  ?subject a ?type .
}
ORDER BY ?label
```

### Events Consumed

| Event name | Source | Effect |
| :--- | :--- | :--- |
| `adwlm-entity-editor:cached-config` | Entity editor | Sets the dataset URL and triggers index loading |
| `adwlm-filesystem-manager:reload-indexes` | Filesystem manager | Clears the Oxigraph store and reloads all indexes |


## RDF-XML Converter

**Directory:** `modules/rdf-xml-converter/`

Converts entity data from JSON-LD (produced by `shacl-form.serialize("application/ld+json")`) to MEI/XML or TEI/XML for the XML output tab.

**Structure:**

- `converters/` — one JavaScript module per entity type (e.g. `person-converter.js`). Each converter receives the JSON-LD object and returns an MEI/XML string.
- `templates/` — MEI/XML or TEI/XML template strings (one per entity type) used as output scaffolding.
- `types/index.js` — maps entity type IRIs to their converter module.

**How it works:**

1. The form content changes → `adwlm-entity-editor` calls `shacl-form.serialize("application/ld+json")`
2. `js/index.js` receives the JSON-LD and looks up the correct converter by entity type IRI
3. The converter fills in the MEI/XML template and returns the result
4. The XML tab displays the result

Each converter is independent. Adding a new entity type requires only a new converter and template — no changes to shared orchestration code.

---