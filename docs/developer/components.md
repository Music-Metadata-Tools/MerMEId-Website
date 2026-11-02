# Component Reference

MerMEId MeLODy consists of four web components, several dialog components, an orchestration script, a service layer and a converter module. The web components communicate mainly through custom DOM events, which keeps each component independently testable and replaceable.

Most events follow the naming convention `<component-name>:<action>` and are dispatched with `bubbles: true` and `composed: true` so they cross shadow DOM boundaries — either on the component itself or on `document` — and are listened to on `document`. Exceptions are `entity-selected` and `mermeid-ui-language-changed`, which use `window`.


### Event Map

```
                    ┌──────────────────────────────────────────┐
                    │        adwlm-filesystem-manager          │
                    │                                          │
  User selects ───► │  dispatches: entity-to-edit              │──► js/index.js ──► 
  file in tree      │  dispatches: repository-selected         │────────────────────────────►┐
                    │  dispatches: clear-entity-editor         │                             │
                    │  dispatches: quick-add-indexed           │                             │
                    │  dispatches: entity-search:reload-indexes ─────────────────────────►┐  │
                    └──────────────────────────────────────────┘                          │  │
                              ▲                                                           │  │
                              │ entity_to_save                                            │  │
                              │ entity-to-delete                                          │  │
                              │ unsaved-changes                                           │  │
                    ┌───────────────────────────────────────┐                             │  │
                    │          adwlm-entity-editor          │◄────────────────────────────┼──┘
                    │                                       │  consumes: repository-selected
                    │  Save button ────────────────────────►│  dispatches: entity-to-save ──► js/index.js
                    │  Quick add ──────────────────────────►│  dispatches: quick-add:entity-to-save
                    │                                       │  dispatches: cached-config ─►┐
                    └───────────────────────────────────────┘                              │
                              ▲ preview-entity                                             │
                    ┌───────────────────────────────────────┐                              │
                    │           adwlm-graph-view            │◄── index-store:loaded        │
                    └───────────────────────────────────────┘                              │
                                                                                           │
                    ┌───────────────────────────────────────┐                              │
                    │          adwlm-entity-search          │◄─────────────────────────────┘
                    │                                       │  consumes: reload-indexes, clear-indexes
                    │  indexStoreService (indexes/*.ttl)    │  consumes: index-store:loaded
                    │  SPARQL query via Oxigraph            │
                    │  Click ──────────────────────────────►│  calls: selectEntityInTree()
                    └───────────────────────────────────────┘  (fallback: entity-selected)
```

### Complete Event List

| Event | Dispatched by | Payload | Consumed by |
| :--- | :--- | :--- | :--- |
| `adwlm-filesystem-manager:entity-to-edit` | Filesystem manager (file opened) | `{ contents, path }` | `js/index.js` → sets `entity_to_edit` on the entity editor and the graph view |
| `adwlm-filesystem-manager:repository-selected` | Filesystem manager (repository clicked, added or auto-selected) | `{ repositoryPath }` | Entity editor, entity search |
| `adwlm-filesystem-manager:item-selected` | Filesystem manager (repository folder expanded) | `{ repositoryPath }` | Entity editor |
| `adwlm-filesystem-manager:clear-entity-editor` | Filesystem manager (repository removed, open file deleted) | — | Entity editor |
| `adwlm-filesystem-manager:quick-add-indexed` | Filesystem manager (quick-add entity saved and indexed) | `{ entity_iri, indexContent }` | Entity editor → selects the new entity in the originating field |
| `adwlm-filesystem-manager:file-changed-remotely` | Filesystem manager (conflict resolved with remote version) | `{ repositoryPath, path, content }` | Entity editor → reloads the open entity |
| `adwlm-filesystem-manager:build-indexes` | currently not dispatched | `{ onlyIfMissing? }` | Filesystem manager → rebuilds all index files |
| `adwlm-entity-search:reload-indexes` | Filesystem manager (after save, delete, unstage, pull, push), entity editor (Reload indexes button) | — | Entity search → `indexStoreService.reloadIndexes()` |
| `adwlm-entity-search:clear-indexes` | Filesystem manager (repository removed) | — | Entity search |
| `adwlm-index-store:loaded` | Index store service | — | Entity search, graph view |
| `adwlm-entity-editor:entity-to-save` | Entity editor (Save button) | `{ entity_iri, rdf_contents, json_ld_contents, path, shapesUrl }` | `js/index.js` → updates the output panel and sets `entity_to_save` on the filesystem manager |
| `adwlm-quick-add:entity-to-save` | Entity editor (quick-add form saved) | same as `entity-to-save` plus `isQuickAdd: true` | `js/index.js` → sets `entity_to_save` on the filesystem manager |
| `adwlm-entity-editor:unsaved-changes` | Entity editor | `{ hasUnsavedChanges }` | Filesystem manager |
| `adwlm-entity-editor:entity-to-delete` | Entity editor (Delete button) | — | Filesystem manager |
| `adwlm-entity-editor:cached-config` | Entity editor (`config.json` read) | `{ datasetBaseUrl, projectDomain }` | Entity search |
| `adwlm-entity-types-dialog:entity-to-add` | Entity types dialog | type IRI (string) | Entity editor, filesystem manager |
| `adwlm-graph-view:preview-entity` | Graph view (eye icon on a node) | `{ relativePath }` | Entity editor → opens the side panel |
| `entity-selected` (on `window`) | Entity search (fallback), entity editor (side panel "Edit") | `{ filename }` | Filesystem manager → opens the file |
| `shacl-form:quick-add` | `shacl-form` (shadow DOM) | `{ classIri, … }` | Entity editor → opens the quick-add dialog |
| `shacl-form:quick-add-closed` | Entity editor | — | `shacl-form` |
| `mermeid-ui-language-changed` (on `window`) | `js/index.js` | `{ language }` | `index.html` (language button label) |

Internal events between the filesystem manager and its dialogs (`adwlm-filesystem-manager:repository-branches`, `…:repository-to-add`, `…:add-local-repository`, `adwlm-rename-filesystem-entry-dialog:new-entry-name`, `adwlm-push-conflict-dialog:resolve` / `:cancel`) are handled inside the filesystem manager.

---

## Orchestration Layer (`js/index.js`)

This is the top-level script that bootstraps the application. It is not a web component — it runs as a plain ES module loaded from `index.html`.
Beyond startup (see [Architecture](architecture.md#bootstrap-sequence)), `js/index.js` is responsible for:

- **Entity type detection** — when a file is opened, it looks for exactly one subject with a `melod:` type; otherwise an alert is shown
- **Language switch** — `window.__MERMEID_setUiLanguage(lang)`
- **Passing data to the components** — `entity_to_edit` (entity editor, graph view), `entity_to_save` (filesystem manager), `entity_type_definitions`, `ui_language`
- **Output Panel** tabs:
    - **XML tab** — listens for serialised entity data and invokes the appropriate converter from `modules/rdf-xml-converter/` to produce MEI/XML output
    - **RDF tab** — displays the raw Turtle from `shacl-form.serialize()`
    - **Preview tab** — renders the entity read-only in a second `shacl-form` in `data-view` mode
    - **Graph tab** — `adwlm-graph-view`

---
## Filesystem Manager

`adwlm-filesystem-manager`

**File:** `modules/filesystem-manager/index.js`

The left panel component. Manages the repository tree, the staged files queue, and all Git operations exposed to the user.

### Responsibilities

- Display and navigate the repository tree (lazily loaded via `sl-tree`); only folders of registered entity types are shown
- Clone, remove, rename, and synchronise repositories
- Stage, unstage, and push (commit + push) modified files, with optional commit message and "Select all"
- Check for conflicts before sharing and show the push conflict dialog
- Highlight unsaved and unshared files in the tree using colour indicators
- Load the selected `.ttl` file and dispatch it to the entity editor
- Respond to save requests from the entity editor and update the index files in `indexes/`
- Create `dataCatalogs/catalog.ttl`, `dataCollections/works.ttl` and `dataCollections/manifestations.ttl` if they are missing when a repository is selected
- Open the repository settings and catalog metadata dialogs

### Key Properties

| Property | Type | Description |
| :--- | :--- | :--- |
| `_selected_repository_path` | `String` | Absolute path of the active repository (e.g. `/my-catalogue`) |
| `_file_path` | `String` | Relative path of the currently open file |
| `_staged_files` | `Array` | Relative paths of files staged for the next commit |
| `entity_to_save` | `Object` | Set externally to trigger a file save; observed via `updated()` |
| `entity_type_definitions` | `Array` | Entity type definitions loaded from `editor-default.ttl` |
| `ui_language` | `String` | Current UI language (reflected as `data-ui-language`) |

### Events Dispatched

| Event | When | Payload |
| :--- | :--- | :--- |
| `adwlm-filesystem-manager:entity-to-edit` | File selected in the repository tree | `{ contents: String, path: String }` |
| `adwlm-filesystem-manager:repository-selected` | Repository clicked, added or auto-selected on load | `{ repositoryPath: String }` |
| `adwlm-filesystem-manager:item-selected` | Repository folder expanded | `{ repositoryPath: String }` |
| `adwlm-filesystem-manager:clear-entity-editor` | Repository removed or currently open entity file deleted | — |
| `adwlm-filesystem-manager:quick-add-indexed` | Quick-add entity saved and indexed | `{ entity_iri, indexContent }` |
| `adwlm-filesystem-manager:file-changed-remotely` | Conflict resolved with the remote version | `{ repositoryPath, path, content }` |
| `adwlm-entity-search:reload-indexes` | After save, delete, unstage, pull or push | — |
| `adwlm-entity-search:clear-indexes` | Repository removed | — |

### Events Consumed

| Event | Source | Effect |
| :--- | :--- | :--- |
| `adwlm-entity-editor:unsaved-changes` | Entity editor | Warns before synchronising if unsaved changes are present |
| `adwlm-entity-editor:entity-to-delete` | Entity editor | Deletes the entity file selected in the tree |
| `adwlm-entity-types-dialog:entity-to-add` | Entity types dialog | Clears the tree selection for the new entity |
| `adwlm-filesystem-manager:build-indexes` | — | Rebuilds the index files of all folders |
| `entity-selected` | Search / side panel (via `window`) | Opens the given file and highlights it in the tree |

### Repository Tree

The tree is lazily loaded. When a `sl-tree-item[lazy]` node is expanded, the `sl-lazy-load` event fires and `_generate_folder_tree()` reads the contents of that folder from the virtual filesystem. Each tree item carries its path in `data-*` attributes:

- `data-entry-type`: `repo-folder`, `folder`, or `file`
- `data-entry-absolute-path`: e.g. `/my-catalogue/persons/12345.ttl`
- `data-entry-relative-path`: e.g. `persons/12345.ttl`

### Conflict Detection

Before a push, `VirtualFilesystem.checkPushConflicts()` loads the current remote version of each file to be pushed and compares it with the version last synchronised locally (`.snapshot.json`). Files changed on both sides are shown in `adwlm-push-conflict-dialog`, which renders both versions with `shacl-form`. For each file the user chooses to keep the local version, take the remote version, or skip the file for this push.

### Dialogs

| Component | Directory | Purpose |
| :--- | :--- | :--- |
| `adwlm-add-repository-dialog` | `add-repository-dialog/` | Remote URL (HTTPS, SSH, GitLab subgroups) or local folder; username/token; branch selection via the API; "Use lazy loading" switch |
| `adwlm-rename-filesystem-entry-dialog` | `rename-filesystem-entry-dialog/` | Renames the selected repository |
| `adwlm-repository-settings-dialog` | `repository-settings-dialog/` | Edits username and token (`update_credentials`) |
| `adwlm-catalog-metadata-dialog` | `catalog-metadata-dialog/` | Edits `dataCatalogs/catalog.ttl` with `configuration/dataCatalog.shacl` |
| `adwlm-push-conflict-dialog` | `push-conflict-dialog/` | Resolves conflicts before sharing |

---

## VirtualFilesystem

**File:** `modules/filesystem-manager/virtual-filesystem/index.js`

A sub-module of `adwlm-filesystem-manager`. Provides a Git-backed virtual filesystem that persists in the browser's Origin Private File System (`opfs-directory-filesystem.js`) or in a local folder (`fsa-directory-filesystem.js`). Network operations use the GitHub/GitLab API provider (`api-provider.js`).

### Key Methods

| Method | Description |
| :--- | :--- |
| `add_repository(metadata)` | Loads a remote repository via the API (`{ folder, url, username, token, branch, use_lazy_loading }`): with lazy loading only the file tree is fetched (`.remote-tree.json`), otherwise all files are downloaded. Then runs `git init`, adds the remote and stores credentials |
| `add_local_repository(...)` | Registers a local folder (File System Access API) as repository |
| `remove_repository(path)` | Recursively deletes all files from the virtual FS (for local folders: forgets the folder handle) |
| `list_repository_names()` | Lists the repositories in the virtual FS root plus the registered local folders |
| `list_branches(metadata)` | Fetches the remote's branch list via the API |
| `list_entries_from_workdir(repo, folder)` | Lists files and subdirectories for a given folder (working directory merged with the lazy-loading manifest) |
| `read_file(repo, path)` | Reads a file as a string (downloads it first if lazy loading is used) |
| `read_directory_files(repo, folder)` | Reads all files of a folder (used to append `indexes/*.ttl` to the shapes) |
| `save_and_stage_file(repo, contents, path)` | Writes a file, stages it and tracks it in `.dirty.json` / `.snapshot.json` |
| `list_staged_files(repo)` | Reads `.dirty.json`; deleted files get a `-deleted` suffix |
| `commit_and_push_file(repo, staged, selected, message)` | Pushes the selected files as one commit via the API (`pushViaApi`) |
| `pull(repo)` | Downloads changed files via the API; returns `{ changedPaths, deletedPaths }` for incremental index updates |
| `checkPushConflicts(...)` | Compares the files to be pushed with their current remote versions |
| `resolveConflictWithRemote(...)` | Replaces a local file with its remote version |
| `unstageFile(repo, path)` | Restores the last synchronised version from `.snapshot.json`, or deletes a new, never-pushed file |
| `get_credentials(...)` / `update_credentials(...)` | Reads / changes username and token |
| `has_remote(repo)` / `has_token(repo)` | Checks whether sharing / synchronising is possible |
| `generate_indexes_for_saved_file(repo, path, isDeleted)` | Updates `indexes/<folder>.ttl` for one file |
| `generate_indexes_for_all_files(repo, folderNames)` | Rebuilds the index files for all files of the given folders |

---

## Entity Editor

`adwlm-entity-editor`

**File:** `modules/entity-editor/index.js`

The main editing area. Wraps the `shacl-form` component and manages the lifecycle of the currently open entity.

### Responsibilities

- Render the `shacl-form` for the currently selected entity
- Load the correct SHACL shape for the entity type and append the content of all `indexes/*.ttl` files of the repository to it, so linked entity dropdowns are populated
- Track unsaved changes and guard against accidental navigation
- Dispatch save requests; block saving if the form is invalid
- Handle "quick add" — creating a new linked entity from within an open form
- Show linked entities in a draggable, resizable floating side panel (read-only preview)
- Automatically link new works and manifestations to the default data collections
- Reload the indexes (Reload indexes button), copy the entity IRI, delete the entity

### Key Properties

| Property | Type | Description |
| :--- | :--- | :--- |
| `entity_to_edit` | `Object` | Currently open entity: `{ contents, path, entity_iri, entity_type, shapesUrl }` |
| `entity_type_definitions` | `Array` | Entity type definitions loaded from `editor-default.ttl` |
| `ui_language` | `String` | Current UI language |
| `_hasUnsavedChanges` | `Boolean` | Set to `true` on any `change` event from `shacl-form` |
| `_selected_repository_path` | `String` | Kept in sync with the filesystem manager via events |
| `_cachedConfig` | `Object` | Cached result of `configuration/config.json` — cleared on repository change |

### Events Dispatched

| Event | When | Payload |
| :--- | :--- | :--- |
| `adwlm-entity-editor:entity-to-save` | Save button clicked | `{ entity_iri, rdf_contents, json_ld_contents, path, shapesUrl }` |
| `adwlm-entity-editor:unsaved-changes` | Unsaved-changes state changes | `{ hasUnsavedChanges: Boolean }` |
| `adwlm-entity-editor:entity-to-delete` | Delete button clicked | — |
| `adwlm-entity-editor:cached-config` | Repository `config.json` read | `{ datasetBaseUrl, projectDomain }` |
| `adwlm-quick-add:entity-to-save` | Quick-add form saved | same payload as `entity-to-save` plus `isQuickAdd: true` |
| `adwlm-entity-search:reload-indexes` | Reload indexes button clicked | — |
| `entity-selected` | "Edit" in the side panel | `{ filename }` |

### Events Consumed

| Event | Source | Effect |
| :--- | :--- | :--- |
| `adwlm-filesystem-manager:clear-entity-editor` | Filesystem manager | Clears the current form |
| `adwlm-filesystem-manager:repository-selected` | Filesystem manager | Updates repository path; clears config cache |
| `adwlm-filesystem-manager:item-selected` | Filesystem manager | Updates repository path; clears config cache |
| `adwlm-filesystem-manager:quick-add-indexed` | Filesystem manager | Appends the new index entry to the form and selects the new entity |
| `adwlm-filesystem-manager:file-changed-remotely` | Filesystem manager | Reloads the open entity |
| `adwlm-entity-types-dialog:entity-to-add` | Entity types dialog | Creates a new empty entity of the selected type |
| `adwlm-graph-view:preview-entity` | Graph view | Opens the side panel for the entity |
| `shacl-form:quick-add` | `shacl-form` (shadow DOM) | Opens the quick-add dialog for a linked entity field |

### Entity ID Generation

New entity IRIs are generated at creation time using the Web Crypto API:

```js
_generate_entity_id() {
    let array = new Uint32Array(1);
    self.crypto.getRandomValues(array);
    return array[0]; // random 32-bit unsigned integer
}
```

The IRI is built as `<projectDomain><folder>/<IDprefix><id>` (`projectDomain` defaults to `urn:uuid:`, `IDprefix` per folder is optional, both from `config.json`). This produces paths like `persons/2847362910.ttl` and IRIs like `<urn:uuid:persons/2847362910>`.

### Quick Add

When `shacl-form` dispatches `shacl-form:quick-add`, the editor resolves the entity type from the `classIri` (removing a trailing `Entity`, e.g. `melod:PlaceEntity` → `melod:Place`) and opens an `sl-dialog` with its own `shacl-form`. Its Save button is enabled once the form is valid. After saving, the filesystem manager updates the index and dispatches `quick-add-indexed`; the editor then appends the new index entry to the originating form and selects the new entity in the dropdown.

### Side Panel (Linked Entity Preview)

When a user clicks a link inside a `shacl-form` field (links are rendered as `<a class="uuid">`), the editor intercepts the click using `composedPath()` to pierce the shadow DOM, reads the linked `.ttl` file from the virtual filesystem, and renders it in a floating, draggable and resizable panel using a second `shacl-form` instance in read-only `data-view` mode. The panel can also be opened from the graph view.

### Repository Configuration Cache

On first load of a repository, the editor reads `configuration/config.json`. The result is cached in `_cachedConfig` and cleared when a different repository is selected. This avoids repeated filesystem reads for every entity opened in the same session.

---

## Entity Search

`adwlm-entity-search`

**File:** `modules/entity-search/index.js`

The search panel. Provides full-text search across all entity labels, alternative labels and classifications in the active repository.

### Responsibilities

- Loads the index files of the repository (one `.ttl` per entity type, listed in `INDEXES` in `modules/services/index-store-service.js`) into the shared Oxigraph store of the index store service
- Runs multiple SPARQL queries to extract entity metadata: types, labels, composers, classifications, and alternative labels
- Filters results by label text (including alternative labels and classifications) and/or entity type
- On selection, calls `selectEntityInTree()` of the filesystem manager (fallback: dispatches `entity-selected`) to open the matching file

### Events Dispatched

| Event | When | Payload |
| :--- | :--- | :--- |
| `entity-selected` | Search result clicked | `{ path: String }` — navigates the tree |

### Events Consumed

| Event | Source | Effect |
| :--- | :--- | :--- |
| `adwlm-filesystem-manager:repository-selected` | Filesystem manager | Updates the repository path |
| `adwlm-entity-editor:cached-config` | Entity editor | Sets the config and loads indexes |
| `adwlm-entity-search:reload-indexes` | Filesystem manager, entity editor | Clears the Oxigraph store and reloads all index files |
| `adwlm-entity-search:clear-indexes` | Filesystem manager | Clears the search |
| `adwlm-index-store:loaded` | Index store service | Rebuilds the result entries |

### How Indexes Work

The search component reads the index files from the `indexes/` folder of the **local copy** of the repository. These files are generated by the CI/CD pipeline of the data repository after each push, and the editor keeps them up to date itself whenever an entity is saved, deleted, unstaged or pulled (see [Datasets Generation](datasets.md)).

**Implication:** the search index reflects the **locally saved state**, including changes that have not been pushed yet. After each of these operations, `adwlm-entity-search:reload-indexes` triggers a fresh reload of all indexes in search and graph view.

### SPARQL Queries

The search component executes **four separate SPARQL queries** against the Oxigraph store and combines the results:

#### Query 1: Subjects and Types

```sparql
SELECT DISTINCT ?subject ?type WHERE {
  ?subject a ?type .
}
ORDER BY ?subject
```

#### Query 2: Labels (Label, Composer)

```sparql
SELECT ?subject ?label ?type ?composer WHERE {
  ?subject <http://www.w3.org/2004/02/skos/core#prefLabel> ?title .
  OPTIONAL {
    ?subject <https://schema.org/composer> ?composer .
  }
  bind(coalesce(concat(?composer, ": ", ?title), ?title) AS ?label) .
}
```

This query extracts the primary label for each entity. If a composer is present in the index, the label is formatted as `"composer: title"`. Otherwise, only the title is used.

#### Query 3: Classifications

```sparql
SELECT ?subject ?classification WHERE {
  ?subject <http://www.w3.org/2004/02/skos/core#broader> ?classification .
}
```

Retrieves the broader (parent) classifications for each entity. These are stored in a `classificationsMap` indexed by subject.

#### Query 4: Alternative Labels

```sparql
SELECT ?subject ?altlabel WHERE {
  ?subject <http://www.w3.org/2004/02/skos/core#altLabel> ?altlabel .
}
```

Retrieves alternative labels for each entity. These are stored in an `altLabelsMap` indexed by subject.

### How Results are Combined

After executing all four queries, the results are merged into a single array of entry objects:

```js
{
  subject: "urn:uuid:works/1031334055",
  label: ["Suite in G Major"],
  type: "https://lod.academy/melod/vocab/ontology#WorkEntity",
  classifications: ["concert", "..."],
  altlabels: ["Alternative Name", "Alternate Title"],
  composer: ["Suite in G Major"]  // filled with the same values as label if no composer is part of the index
}
```

---

## Graph View

`adwlm-graph-view`

**File:** `modules/graph-view/index.js`

The Graph tab of the output panel. Shows the entities related to the currently open entity.

- **Properties:** `entity_to_edit`, `entity_type_definitions`, `ui_language`
- **Data:** reads `skos:related` and `skos:prefLabel` from the shared index store (incoming and outgoing relations)
- **Views:** SVG graph (pan/zoom, expand a node, at most 8 children before a "+N more" node) or list view, with a type filter and legend; colours per type are defined in `TYPE_COLORS`
- **Events:** listens to `adwlm-index-store:loaded`; the eye icon on a node dispatches `adwlm-graph-view:preview-entity` (`{ relativePath }`) to open the side panel of the entity editor

---

## RDF-XML Converter

**Directory:** `modules/rdf-xml-converter/`

Converts entity data from JSON-LD (produced by `shacl-form.serialize("application/ld+json")`) to MEI/XML or TEI/XML for the XML output tab.

**Structure:**

- `converters/` — one JavaScript module per entity type (e.g. `person.js`, `performance-event.js`). Each exports a class (e.g. `PersonConverter`) with a static `toXML(jsonLd)` method that collects the values from the flat JSON-LD array into a data object.
- `templates/` — one module per entity type (e.g. `person.js`) exporting a function `generate<Type>XML(data)` that returns the formatted MEI/XML or TEI/XML string.
- `types/index.js` — JSDoc `@typedef` definitions of the data objects (e.g. `PersonData`, `WorkData`).

**How it works:**

1. The form content changes → `adwlm-entity-editor` calls `shacl-form.serialize("application/ld+json")`
2. `js/index.js` receives the JSON-LD and looks up the correct converter by entity type IRI
3. The converter fills in the MEI/XML template and returns the result
4. The XML tab displays the result

Each converter is independent. Adding a new entity type requires only a new converter and template — no changes to shared orchestration code.
