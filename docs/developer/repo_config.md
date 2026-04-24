# Repository Configuration

Each **data repository** contains a configuration file at `configuration/config.json`. This file must be present and correctly configured for the editor to function with that repository.

---

## Location

```
my-catalogue/
└── configuration/
    └── config.json   ← this file
```

The editor reads this path relative to the repository root when a repository is loaded. The path cannot be changed.

---

## Example

```json
{
  "datasetBaseUrl": "https://yourname.github.io/my-catalogue/datasets/",
  "projectDomain": "urn:uuid:"
}
```

---

## Fields

### `datasetBaseUrl`

The base URL where the published dataset index files are hosted. This is typically a GitHub Pages or GitLab Pages URL.

**Format:** a full HTTPS URL ending with a `/`

**Example:**
```json
"datasetBaseUrl": "https://yourname.github.io/my-catalogue/datasets/"
```

#### What Are Datasets?

Datasets are aggregated RDF/Turtle files built from your repository's data. They are **generated automatically by a CI/CD pipeline** after each push using SPARQL CONSTRUCT queries. Each dataset corresponds to an entity type:

- `persons.ttl` — all persons (indexed by full name)
- `works.ttl` — all works (indexed by title and composer)
- `institutions.ttl` — all institutions (indexed by name and abbreviation)
- `places.ttl` — all places (indexed by name)
- `venues.ttl` — all venues (indexed by name and location)
- `manifestations.ttl`, `expressions.ttl`, `items.ttl`, `bibliography.ttl`, etc.

For a complete list and how each dataset is built, see [Datasets Generation](datasets.md).

#### How Datasets Are Used

The `datasetBaseUrl` is used in two ways:

1. **Entity search** — the `adwlm-entity-search` component fetches and aggregates all dataset files from this URL into its its Oxigraph SPARQL store. When you search for "Bach" in the editor, it queries these pre-built indices rather than scanning individual entity files.

2. **Cross-reference dropdowns in forms** — linked entity fields (e.g., "Contributor, "Birth Place") use dataset URIs to populate the dropdown lists. SHACL shapes declare these fields using a `dataset:` namespace placeholder, which the editor expands at runtime to `https://yourname.github.io/my-catalogue/datasets/<entity-type>.ttl`.

**Both features depend on datasets being generated and published.**

**For GitHub repositories** created from the template, the URL follows this pattern:

```
https://<owner>.github.io/<repo-name>/datasets/
```

**For GitLab repositories**, it follows the URL configured in the CI/CD pipeline (typically a GitLab Pages URL).

!!! note
    The dataset index must be published (pushed and built by CI/CD) before the search and cross-reference features work. The search index always reflects the **last pushed** state, not local unsaved changes.

---

### `projectDomain`

The IRI prefix used when generating identifiers for new entities.

**Default value:** `"urn:uuid:"` — produces opaque identifiers like `urn:uuid:persons/2847362910`

**Alternative:** an HTTPS namespace — produces dereferenceable URIs like `https://example.org/persons/2847362910`

```json
"projectDomain": "https://example.org/"
```

See [Entity IRIs](../data_model/iris.md) for full details on IRI generation and the implications of changing this value.

!!! warning
    Changing `projectDomain` after entities have been created does **not** retroactively update existing entity IRIs. Existing entities keep their original IRIs. New entities will use the new prefix, resulting in a mixed-prefix dataset. Decide on your IRI scheme before creating any entities.

---

## Typical Setup (GitHub)

If you created your repository from the [MerMEId MeLODy Template](https://github.com/Music-Metadata-Tools/MerMEId-MeLODy-Template) on GitHub, the typical `config.json` looks like:

```json
{
  "datasetBaseUrl": "https://Music-Metadata-Tools.github.io/my-catalogue/datasets/",
  "projectDomain": "urn:uuid:"
}
```

Replace `Music-Metadata-Tools` with your GitHub organisation or username and `my-catalogue` with your repository name.

Make sure GitHub Pages is enabled for your repository (Settings → Pages → Deploy from branch: `gh-pages`).
