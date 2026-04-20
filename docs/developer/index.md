# Developer Reference

This section documents the technical architecture of MerMEId MeLODy for developers who want to understand, contribute to, or extend the editor.

---

## Sections

- [Architecture Overview](architecture.md) — bootstrap sequence, technology stack, and module structure
- [Components](components.md) — detailed reference for each web component: properties, events, and behaviour
- [Services](services.md) — `FilesystemService` and `VirtualFilesystem`: the Git and filesystem layer
- [Event Communication](events.md) — the full event map between components
- [Extending the Editor](extending.md) — how to add new entity types, SHACL shapes, and MEI/XML converters

---

## At a Glance

MerMEId MeLODy is a **static, browser-only application** — there is no backend server. All operations run entirely in the browser:

- Git operations are performed in-memory using [isomorphic-git](https://isomorphic-git.org/) with [LightningFS](https://github.com/isomorphic-git/lightning-fs) (an `IndexedDB`-backed virtual filesystem)
- The UI is built with [Lit](https://lit.dev/) web components and [Shoelace](https://shoelace.style/)
- Forms are generated automatically from SHACL shapes by the [shacl-form](https://github.com/ulb-darmstadt/shacl-form) component
- RDF data is queried via an in-browser [Oxigraph](https://github.com/oxigraph/oxigraph) SPARQL engine
- All dependencies are loaded from CDNs — no build step is required

Components communicate exclusively through **custom DOM events**. There are no direct references between components, keeping the architecture loosely coupled and each component independently testable.

---

## Quick Start for Contributors

```bash
git clone https://github.com/Music-Metadata-Tools/MerMEId-MeLODy.git
cd MerMEId-MeLODy
python3 -m http.server 8080
```

Open `http://localhost:8080`. The editor runs directly from the source files — no compilation needed.

For running tests:

```bash
npm install
npm test
```

See the [Community → Contributing](../community/contributing.md) page for the full contribution workflow.
