# Event Communication

Components in MerMEId MeLODy communicate exclusively through custom DOM events dispatched on the `window` object. There are no direct references between components — this keeps each component independently testable and replaceable.

---

## Event Map

```
                    ┌──────────────────────────────────┐
                    │     adwlm-filesystem-manager      │
                    │                                   │
  User selects ───► │  dispatches: entity-to-edit      │──────────────────────────►┐
  file in tree      │  dispatches: repository-selected │─────────────────────────►┐│
                    │  dispatches: clear-entity-editor  │                          ││
                    │  dispatches: reload-indexes       │───────────────────────►┐ ││
                    └──────────────────────────────────┘                         │ ││
                              ▲                                                   │ ││
                              │ entity-to-save                                    │ ││
                              │ entity-to-delete                                  │ ││
                              │ unsaved-changes                                   │ ││
                    ┌──────────────────────────────────┐                         │ ││
                    │        adwlm-entity-editor        │                         │ ││
                    │                                   │◄────────────────────────┘ ││
                    │  User edits form ────────────────►│  consumes: entity-to-edit  ││
                    │  Save button ────────────────────►│  dispatches: entity-to-save││
                    └──────────────────────────────────┘                            ││
                                                                                    ││
                    ┌──────────────────────────────────┐                            ││
                    │        adwlm-entity-search        │◄───────────────────────────┘│
                    │                                   │  consumes: repository-selected
                    │  Fetches index TTLs from URL ─────│  consumes: reload-indexes ◄─┘
                    │  SPARQL query via Oxigraph        │
                    │  Click ─────────────────────────►│  calls: filesystemManager
                    └──────────────────────────────────┘         .selectEntityInTree()
```

---

## Complete Event Reference

### Events from `adwlm-filesystem-manager`

| Event name | Dispatched when | Payload |
| :--- | :--- | :--- |
| `adwlm-filesystem-manager:entity-to-edit` | A `.ttl` file is selected in the repository tree | `{ contents: String, path: String }` |
| `adwlm-filesystem-manager:repository-selected` | A repository is clicked or auto-selected on load | `{ repositoryPath: String }` |
| `adwlm-filesystem-manager:clear-entity-editor` | The currently open entity file is deleted | — |
| `adwlm-filesystem-manager:reload-indexes` | After a push or pull completes | — |

---

### Events from `adwlm-entity-editor`

| Event name | Dispatched when | Payload |
| :--- | :--- | :--- |
| `adwlm-entity-editor:entity-to-save` | The Save button is clicked | `{ entity_iri, rdf_contents, json_ld_contents, path, shapesUrl }` |
| `adwlm-entity-editor:unsaved-changes` | The unsaved-changes state changes | `{ hasUnsavedChanges: Boolean }` |
| `adwlm-entity-editor:cached-config` | The repository `config.json` is read | `{ datasetBaseUrl, projectDomain }` |
| `adwlm-quick-add:entity-to-save` | A quick-add form is saved | same as `entity-to-save` |

---

### Events from `adwlm-entity-types-dialog`

| Event name | Dispatched when | Payload |
| :--- | :--- | :--- |
| `adwlm-entity-types-dialog:entity-to-add` | User selects a type in the "New entity" dialog | `{ entity_type: String, shapesUrl: String }` |

---

### Events consumed by `adwlm-filesystem-manager`

| Event name | Source | Effect |
| :--- | :--- | :--- |
| `adwlm-entity-editor:entity-to-save` | Entity editor | Saves the file to the virtual FS and stages it |
| `adwlm-entity-editor:unsaved-changes` | Entity editor | Blocks synchronise if unsaved changes are present |
| `adwlm-entity-editor:entity-to-delete` | Entity editor | Deletes the entity file from the virtual FS |
| `entity-selected` | `window` (from search) | Navigates the tree to the given file path |

---

### Events consumed by `adwlm-entity-editor`

| Event name | Source | Effect |
| :--- | :--- | :--- |
| `adwlm-filesystem-manager:entity-to-edit` | Filesystem manager | Loads the entity Turtle into the form |
| `adwlm-filesystem-manager:clear-entity-editor` | Filesystem manager | Clears the current form |
| `adwlm-filesystem-manager:repository-selected` | Filesystem manager | Updates the selected repository path; clears config cache |
| `adwlm-entity-types-dialog:entity-to-add` | Entity types dialog | Creates a new empty entity of the selected type |
| `shacl-form:quick-add` | `shacl-form` (shadow DOM) | Opens the quick-add dialog for a linked entity field |

---

### Events consumed by `adwlm-entity-search`

| Event name | Source | Effect |
| :--- | :--- | :--- |
| `adwlm-entity-editor:cached-config` | Entity editor | Sets the dataset URL and loads indexes |
| `adwlm-filesystem-manager:reload-indexes` | Filesystem manager | Clears the Oxigraph store and reloads all index files |

---

## Dispatching and Listening

All events are dispatched on `window` with `bubbles: true` and `composed: true` so they cross shadow DOM boundaries:

```js
window.dispatchEvent(new CustomEvent("adwlm-filesystem-manager:entity-to-edit", {
    bubbles: true,
    composed: true,
    detail: { contents, path }
}));
```

Components listen in their `connectedCallback()` or as part of their Lit `render()` lifecycle:

```js
window.addEventListener("adwlm-filesystem-manager:entity-to-edit", (e) => {
    this._loadEntity(e.detail.contents, e.detail.path);
});
```

The consistent naming convention `<component-name>:<action>` makes it straightforward to trace which component dispatches and which consumes any given event.
