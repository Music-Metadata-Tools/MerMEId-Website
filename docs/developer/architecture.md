# Architecture Overview

---

## Application Architecture

MerMEId MeLODy is a **static, browser-only application**. There is no server-side component — all processing happens in the browser. The application is served as static HTML, CSS, and JavaScript files.

```
Browser
├── js/index.js                    ← orchestration layer (startup + event wiring)
├── adwlm-filesystem-manager       ← left panel: repositories + staging area
├── adwlm-entity-search            ← left panel: full-text search
├── adwlm-entity-editor            ← main area: form editor
│   └── shacl-form                 ← form renderer (external component)
└── FilesystemService (singleton)
    └── VirtualFilesystem
        ├── LightningFS            ← virtual filesystem in IndexedDB
        └── isomorphic-git         ← Git operations in-memory
```

Cross-origin Git requests are routed through a CORS proxy (`https://cors.isomorphic-git.org`). No data is processed server-side; the proxy only adds the necessary CORS headers to Git HTTP responses.

---

## Bootstrap Sequence

When the page loads, the following sequence occurs in `js/index.js`:

1. **Shoelace ready** — the page waits for Shoelace's custom elements (`sl-button`, `sl-tree`, `sl-dialog`, etc.) to be defined before proceeding.
2. **Fetch `editor-default.ttl`** — the orchestration script fetches `configuration/editor-default.ttl` from the editor server. This Turtle file defines all registered entity types.
3. **SPARQL query via Oxigraph** — the Turtle content is loaded into an in-browser Oxigraph triple store. A SPARQL SELECT query extracts entity type definitions: type IRI, display label, folder name, SHACL file location.
4. **Pass definitions to entity_editor** — the extracted definitions are set as the `entity_type_definitions` property on the `adwlm-entity-editor` component. The editor uses these to populate the "New entity" dialog and to load the correct SHACL shape for each file type.
5. **Components initialize** — web components connect to the DOM, set up internal state, and register listeners for the custom DOM events they consume.
6. **Output Panel wiring** — `js/index.js` connects the Output Panel tabs (XML, RDF, Preview) to the entity editor's output events. When the form content changes, the output panel updates automatically.

---

## Technology Stack

| Library | Version | Purpose |
| :--- | :--- | :--- |
| [Lit](https://lit.dev/) | latest | Web component base class (`LitElement`) for all editor components |
| [Shoelace](https://shoelace.style/) | latest | UI components: `sl-button`, `sl-tree`, `sl-dialog`, `sl-tab`, etc. |
| [isomorphic-git](https://isomorphic-git.org/) | 1.27.1 | Git operations (clone, pull, push, commit, status) in the browser |
| [LightningFS](https://github.com/isomorphic-git/lightning-fs) | bundled | Virtual filesystem persisted in `IndexedDB` |
| [shacl-form](https://github.com/ulb-darmstadt/shacl-form) | latest | SHACL-driven form renderer — generates HTML forms from SHACL shape files |
| [Oxigraph](https://github.com/oxigraph/oxigraph) | 0.4.5 | In-browser RDF triple store + SPARQL 1.1 engine (entity search + startup config loading) |

All dependencies are loaded directly from CDNs (`cdn.jsdelivr.net`, `unpkg.com`). No build step or package manager is required to run or develop the editor.

---

## Module Structure

```
js/
└── index.js                        ← orchestration: loads editor-default.ttl, wires components

modules/
├── entity-editor/
│   ├── index.js                        ← adwlm-entity-editor component
│   └── entity-types-dialog/index.js    ← dialog for selecting a new entity type
├── entity-search/
│   └── index.js                        ← adwlm-entity-search component
├── filesystem-manager/
│   ├── index.js                        ← adwlm-filesystem-manager component
│   ├── constants.js                    ← shared constants (CORS proxy URL, scheme names)
│   ├── credentials-helper.js           ← token handling utilities
│   ├── add-repository-dialog/index.js  ← dialog for cloning a repository
│   ├── rename-filesystem-entry-dialog/ ← dialog for renaming repositories
│   └── virtual-filesystem/index.js     ← VirtualFilesystem class (Git + FS operations)
├── rdf-xml-converter/
│   ├── converters/                     ← one converter module per entity type
│   ├── templates/                      ← MEI/XML output templates (one per entity type)
│   └── types/index.js                  ← maps entity type IRIs to converter modules
└── services/
    └── filesystem-service.js           ← FilesystemService singleton
```

