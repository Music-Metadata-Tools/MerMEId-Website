# Architecture Overview

---

## Application Architecture

MerMEId MeLODy is a **static, browser-only application**. There is no server-side component — all processing happens in the browser. The application is served as static HTML, CSS, and JavaScript files.

```
Browser
├── js/index.js                    ← orchestration layer (startup, language + event wiring)
├── modules/services/
│   ├── filesystem-service.js          ← singleton VirtualFilesystem shared by all components
│   └── index-store-service.js         ← shared Oxigraph store with all indexes/*.ttl (INDEXES list)
├── adwlm-filesystem-manager       ← left panel: repositories + staging area
│   ├── index.js                        ← adwlm-filesystem-manager component
│   ├── api-provider.js                 ← GitHub/GitLab REST + GraphQL provider (clone, pull, push, branches)
│   ├── constants.js                    ← shared constants (CORS proxy URL, scheme names)
│   ├── credentials-helper.js           ← token handling utilities
│   ├── oxigraph-node-adapter.js           ← #oxigraph shim for Node.js / Vitest
│   ├── add-repository-dialog/index.js  ← dialog for cloning a repository
│   ├── rename-filesystem-entry-dialog/ ← dialog for renaming repositories
│   ├── repository-settings-dialog/        ← dialog for changing username / personal access token
│   ├── catalog-metadata-dialog/           ← dialog for editing catalog metadata
│   ├── push-conflict-dialog/              ← dialog for resolving conflicts before sharing
│   └── virtual-filesystem/
│       ├── index.js                       ← VirtualFilesystem class (files, staging, push/pull, index generation)
│       ├── opfs-directory-filesystem.js   ← default storage (Origin Private File System)
│       └── fsa-directory-filesystem.js    ← fs adapter for local folders (File System Access API)
├── adwlm-entity-search            ← left panel: entity search
|   └── index.js                        ← adwlm-entity-search component
├── adwlm-entity-editor            ← main area: form editor
|   ├── index.js                        ← adwlm-entity-editor component and shacl-form renderer
│   └── entity-types-dialog/index.js    ← dialog for selecting a new entity type
├── adwlm-graph-view                   ← right panel: Graph tab
│   └── index.js                           ← adwlm-graph-view component
├── datasets-generator                 ← SPARQL CONSTRUCT queries, one per entity folder (index generation)
└── rdf-xml-converter              ← right panel: XML output
    ├── converters                     ← one converter module per entity type
    ├── templates                      ← MEI/XML output templates (one per entity type)
    └── types/index.js                  ← maps entity type IRIs to converter modules
```

Git hosts are reached through their REST/GraphQL APIs (`api-provider.js`). No data is processed server-side; the only external requests go to the Git host's API.

### Storage

Each repository is a directory in the browser's Origin Private File System (OPFS), or a local folder opened via the File System Access API. Besides the data files and the `.git/config` (username, token, branch, remote, lazy-loading setting), the editor keeps some bookkeeping files per repository:

| File | Purpose |
| :--- | :--- |
| `.dirty.json` | Files that have been changed or deleted locally and not yet shared |
| `.snapshot.json` | The last synchronised content of locally changed files (baseline for unstaging and conflict detection) |
| `.remote-tree.json` | Manifest of the remote files for lazy loading (files are downloaded on first access) |

---

## Bootstrap Sequence

When the page loads, the following sequence occurs:

1. **Shoelace ready** — a script in `index.html` waits for Shoelace's tab components (`sl-tab-group`, `sl-tab`, `sl-tab-panel`) to be defined and then adds the class `ready` to the body. `js/index.js` itself starts by initialising Oxigraph.
2. **Fetch `editor-default.ttl`** — the orchestration script fetches `configuration/editor-default.ttl` from the editor server. This Turtle file defines all registered entity types.
3. **SPARQL query via Oxigraph** — the Turtle content is loaded into an in-browser Oxigraph triple store. A SPARQL SELECT query extracts the entity type definitions — type IRI, display label in the current UI language, folder name, SHACL file location. Types without a label in the current language are not returned.
4. **Pass definitions to the components** — the extracted definitions are set as the `entity_type_definitions` property on `adwlm-entity-editor`, `adwlm-filesystem-manager` and `adwlm-graph-view`. The editor uses them to populate the "New entity" dialog and to load the correct SHACL shape for each file type; the filesystem manager uses the folder names to decide which folders are shown in the tree. The list of type IRIs is also exposed as `globalThis.__MERMEID_ENTITY_TYPE_ALLOWLIST__` for the quick add feature of shacl-form.
5. **Components initialize** — web components connect to the DOM, set up internal state, and register listeners for the custom DOM events they consume. The filesystem manager auto-selects the first repository and dispatches `adwlm-filesystem-manager:repository-selected`, which triggers loading the config and the search indexes.
6. **Output Panel wiring** — `js/index.js` connects the Output Panel tabs (XML, RDF, Preview, Graph) to the entity editor. The output panel is updated when an entity is opened (`adwlm-filesystem-manager:entity-to-edit`) or saved (`adwlm-entity-editor:entity-to-save`), not on every change in the form.

### Language Switch

`window.__MERMEID_setUiLanguage(lang)` (supported: `en`, `de`) is called by the language dropdown in the header. It reloads the entity type definitions in the new language, sets the `data-ui-language` attribute on all components and dispatches `mermeid-ui-language-changed` on `window`. Static texts in `index.html` are switched by showing or hiding elements with a `lang` attribute.

---

## Technology Stack

| Library | Version | Purpose |
| :--- | :--- | :--- |
| [Lit](https://lit.dev/) | latest | Web component base class (`LitElement`) for all editor components |
| [Shoelace](https://shoelace.style/) | 2.17.1 | UI components: `sl-button`, `sl-tree`, `sl-dialog`, `sl-tab`, etc. |
| [isomorphic-git](https://isomorphic-git.org/) | 1.37.4 | Local Git operations (init, config, staging, walking the working directory) |
| OPFS / File System Access API | browser | Storage of repositories (own adapters in `virtual-filesystem/`) |
| [shacl-form](https://github.com/ulb-darmstadt/shacl-form) | [mermeid-shacl-form](https://github.com/Music-Metadata-Tools/mermeid-shacl-form) fork | SHACL-driven form renderer — generates HTML forms from SHACL shape files + quick add and rich text editor plugin |
| [Oxigraph](https://github.com/oxigraph/oxigraph) | 0.5.8 / 0.4.5 | In-browser RDF triple store + SPARQL 1.1 engine. 0.5.8 (via `#oxigraph`) is used for index generation in the virtual filesystem; 0.4.5 is imported directly by `js/index.js` and the index store service (startup config loading, entity search, graph view) |
| [Vitest](https://vitest.dev/), memfs, dotenv | dev dependencies | Test suite |

All dependencies are loaded directly from CDNs (`cdn.jsdelivr.net`, `unpkg.com`). No build step or package manager is required to run or develop the editor; npm is only needed to run the tests.
