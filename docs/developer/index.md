# Developer Reference

This section documents the technical architecture of MerMEId MeLODy for developers who want to understand, contribute to, or extend the editor.

---

## Sections

- [Architecture Overview](architecture.md) — bootstrap sequence, technology stack, and module structure
- [Components](components.md) — detailed reference for each web component: properties, events, and behaviour
- [Services](services.md) — `FilesystemService` and `VirtualFilesystem`: the Git and filesystem layer
- [Event Communication](events.md) — the full event map between components
- [SHACL Shapes](shacl_shapes.md) — how to define and extend form fields using SHACL
- [Extending the Editor](extending.md) — how to add new entity types, SHACL shapes, and MEI/XML converters
- [Repository Configuration](repo_config.md) — `config.json` fields: dataset URL and IRI prefix

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

The editor is a static HTML/JS application — no build step is needed for most work. To run it locally:

```bash
git clone https://github.com/Music-Metadata-Tools/MerMEId-MeLODy.git
cd MerMEId-MeLODy
python3 -m http.server 8080
```

Then open `http://localhost:8080`.

**If you have write access** to the repository, create a branch directly (name it after the issue, e.g. `issue-33`).

**If you are an external contributor**, fork the repository first, then create a branch in your fork. Open a Pull Request against `main` of the original repository when ready.

Commit messages should be short and written in the imperative: *"Add person converter"*, not *"Added"* or *"Adding"*.

---

## Running the Tests

If you are working on core functionality (filesystem management, git operations), run the test suite:

```bash
npm install
npm test
```

Requires [Node.js](https://nodejs.org) v22 or later.

For integration tests (real clone/pull/push operations), you need a test repository and a GitHub personal access token. Create a repository from the [MerMEId MeLODy Template](https://github.com/Music-Metadata-Tools/MerMEId-MeLODy-Template) and add a `.env` file in the project root:

```
GIT_TOKEN=<your GitHub personal access token>
GIT_USERNAME=<your GitHub username>
TEST_REPO_URL=<URL of your test repository>
```

Integration tests are automatically skipped if these variables are not set.

---

## Pull Request Review

PRs are merged into `main` by a maintainer once:

- Unit tests pass (run automatically for all PRs)
- At least one maintainer has approved
- There are no requested changes or conflicts

We use **squash and merge** to keep the commit history clean.

---