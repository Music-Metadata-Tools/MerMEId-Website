# Datasets Generation

This document explains how MerMEId MeLODy generates and uses datasets for search indices and dropdown menus. Understanding this process is essential for setting up a data repository, customizing search behavior, and extending the system with new entity types.

---

## Overview

**Datasets** are aggregated RDF/Turtle files generated from your repository's data. They serve two critical functions in the editor:

1. **Search indices** — powers the entity search functionality that loads in the editor
2. **Cross-reference dropdowns** — supplies the linked entity fields (e.g., "Birth Place", "Contributor") in the editor forms

Without published datasets, the editor can read and edit entities locally, but the search and cross-reference features are not working.

---

## How Datasets Are Generated

### The Pipeline

The template repository includes a **GitHub Actions workflow** that automatically generates datasets after every push:

```
Push
    ↓
GitHub Actions: generate_datasets job
    ↓
Run datasets-generator.sh script
    ↓
For each dataset:
  - Collect all RDF files from their source directory
  - Execute a SPARQL CONSTRUCT query
  - Output aggregated .ttl file to public/datasets/
    ↓
Upload public/ folder to GitHub Pages
    ↓
Datasets are published at your Pages URL
```

### Tools Used

The generation process requires:

- **`static-publishing-backend`** Docker container — provides the RDF aggregator and SPARQL engine
- **SPARQL CONSTRUCT queries** — transform entity data into searchable datasets
- **GitHub Actions** — orchestrates the workflow on every push

All of these are configured in the template; you do not need to set anything up manually.

---

## Dataset Files

When the pipeline runs, it generates Turtle (.ttl) files in the `public/datasets/` directory. Each dataset corresponds to an entity type and is published to your GitHub Pages URL.

| Dataset File | Source Directory | Entity Type | Used For |
|---|---|---|---|
| `persons.ttl` | `persons/` | Person | Search, cross-references (e.g., contributors, publishers) |
| `works.ttl` | `works/` | Work | Search, cross-references (e.g., related works) |
| `expressions.ttl` | `expressions/` | Expression | Search, cross-references (related expressions, works and manifestations)|
| `manifestations.ttl` | `manifestations/` | Manifestation | Search, cross-references (related manifestations) |
| `items.ttl` | `items/` | Item | Search, cross-references (related manifestations and items) |
| `institutions.ttl` | `institutions/` | Institution | Search, cross-references (e.g., holding institutions/repositories) |
| `places.ttl` | `places/` | Place | Search, cross-references (e.g., birth place) |
| `venues.ttl` | `venues/`, `performanceEvents/` | Venue | Search, cross-references (e.g. performance venue) |
| `events.ttl` | `events/` | Event | Search, cross-references (e.g. historic events of a items creation) |
| `performanceEvents.ttl` | `performanceEvents/` | Performance Event | Search, cross-references (e.g. first performance of an expression) |
| `instrumentations.ttl` | `instrumentations/` | Instrumentation | Search, cross-references (e.g. instrumentation of an expression) |
| `bibliography.ttl` | `bibliography/` | Bibliography | Search, cross-references (e.g. evidences of a performance or work) |

---

## SPARQL Query Reference

Each dataset is built using a **SPARQL CONSTRUCT query** that transforms entity data into a simplified searchable form. These queries are located in [`modules/datasets-generator/`](https://github.com/Music-Metadata-Tools/MerMEId-MeLODy-Template/tree/main/modules/datasets-generator).

### Persons Dataset

**File:** `persons.sparql`

**Purpose:** Index all persons with their full names.

**Output properties:**

- `skos:prefLabel` — formatted name (e.g., "Bach, Johann Sebastian")

**Query logic:**

- Combines `schema:familyName` and `schema:givenName` into a displayable label
- Falls back to family name only if given name is missing
- Sorted alphabetically by full name

---

### Works Dataset

**File:** `works.sparql`

**Purpose:** Index all works with titles, alternative titles, and composer information.

**Output properties:**

- `skos:prefLabel` — main title
- `skos:altLabel` — uniform title and alternative title (if present)
- `skos:broader` — work classification (if present)
- `schema:composer` — composer name (if linked)

**Query logic:**

- Extracts the main title (type: `MainTitle`)
- Optionally includes uniform title and alternative titles as searchable variants
- Optionally extracts composer information via a contribution relationship
- Sorted by main title

---

### Expressions Dataset

**File:** `expressions.sparql`

**Purpose:** Index all expressions with labels including incipit text for identification.

**Output properties:**

- `skos:prefLabel` — expression label with optional incipit text

**Query logic:**

- Uses either `skos:prefLabel` or `rdfs:label` as base label
- Optionally appends incipit text in parentheses for disambiguation
- Sorted by label

---

### Manifestations Dataset

**File:** `manifestations.sparql`

**Purpose:** Index all manifestations with their titles and classifications.

**Output properties:**

- `skos:prefLabel` — manifestation title
- `skos:broader` — classification (if present)

**Query logic:**

- Extracts title from title subject
- Optionally includes classification for broader search
- Sorted by title

---

### Items Dataset

**File:** `items.sparql`

**Purpose:** Index all items with titles and classifications.

**Output properties:**

- `skos:prefLabel` — item label
- `skos:broader` — classification

**Query logic:**

- Uses item label from `rdfs:label`
- Includes classification
- Sorted by label

---

### Institutions Dataset

**File:** `institutions.sparql`

**Purpose:** Index all institutions with abbreviations or RISM sigla.

**Output properties:**

- `skos:prefLabel` — label with optional abbreviation or RISM siglum
- `skos:broader` — RISM identifier if present

**Query logic:**

- Uses institution name as base
- Prepends abbreviation or RISM siglum if available: `"DB (Name)"`
- Falls back to name only if no abbreviation or rism siglum
- Sorted by label

---

### Places Dataset

**File:** `places.sparql`

**Purpose:** Index all places with their names.

**Output properties:**

- `skos:prefLabel` — place name

**Query logic:**

- Extracts place name from `schema:name`
- Sorted alphabetically

---

### Venues Dataset

**File:** `venues.sparql`

**Purpose:** Index all venues with their names and locations.

**Output properties:**

- `skos:prefLabel` — venue name
- `skos:broader` — place name (if venue is located in a place)

**Query logic:**

- Extracts venue name from `schema:name`
- Optionally includes containing place via `schema:containedInPlace`
- Sorted by venue name

---

### Events Dataset

**File:** `events.sparql`

**Purpose:** Index all events with their names.

**Output properties:**

- `skos:prefLabel` — event name

**Query logic:**

- Extracts event name from `rdfs:label`
- Sorted by name

---

### Performance Events Dataset

**File:** `performanceEvents.sparql`

**Purpose:** Index all performance events with dates and venues.

**Output properties:**

- `skos:prefLabel` — performance label with optional date and venue

**Query logic:**

- Uses performance event name as base
- Optionally appends date and place information for context
- Combined label format: `"Name: Date Place"`
- Sorted by label

---

### Instrumentations Dataset

**File:** `instrumentations.sparql`

**Purpose:** Index all instrumentations.

**Output properties:**

- `skos:prefLabel` — instrumentation name

**Query logic:**

- Extracts name from `rdfs:label`
- Sorted alphabetically

---

### Bibliography Dataset

**File:** `bibliography.sparql`

**Purpose:** Index all bibliography entries with abbreviations or titles.

**Output properties:**

- `skos:prefLabel` — bibliography label with abbreviation or title

**Query logic:**

- Prefers `melod:hasAbbreviation` if present
- Falls back to title if no abbreviation exists
- Sorted by label

---

## The Generation Script

The file [`modules/datasets-generator/datasets-generator.sh`](https://github.com/Music-Metadata-Tools/MerMEId-MeLODy-Template/blob/main/modules/datasets-generator/datasets-generator.sh) orchestrates the entire process. It:

1. Sets up directory paths for all entity types
2. For each dataset, calls the `rdf-data-aggregator` tool with:

   - **Source directory:** where entity files are located (e.g., `persons/`)
   - **File pattern:** which files to include (e.g., `*.ttl`)
   - **SPARQL query:** the transformation logic (e.g., `persons.sparql`)
   - **Output path:** where to write the result (e.g., `public/datasets/persons.ttl`)

3. Times each operation for performance monitoring

Example invocation:
```bash
/static-publishing-backend rdf-data-aggregator \
  $persons_dir_path/ \
  "*.ttl" \
  $datasets_generator_dir_path/persons.sparql \
  $datasets_dir_path/persons.ttl
```

---

## GitHub Pages Publishing

After datasets are generated, the GitHub Actions workflow publishes them to GitHub Pages:

1. Datasets are created in `public/datasets/` directory
2. The artifact is uploaded to GitHub Pages
3. They become available at your configured `datasetBaseUrl` URL

**Required configuration:**

- GitHub Pages must be enabled for your repository (Settings → Pages → Source: GitHub Actions)
- The repository must be **public** (for free GitHub Pages hosting)

For GitLab repositories, configure Pages in the CI/CD pipeline settings.

---

## Adding a New Dataset

To add a new dataset type (e.g., for a custom entity type):

### 1. Create a SPARQL Query

Add a new `.sparql` file in `modules/datasets-generator/` that follows the pattern:

```sparql
prefix melod: <https://lod.academy/melod/vocab/ontology#>
prefix skos: <http://www.w3.org/2004/02/skos/core#>
prefix schema: <https://schema.org/>

construct {
  ?iri a melod:CustomEntity ;
    skos:prefLabel ?label .
} where {
  select ?iri ?label where {
    ?iri a melod:CustomEntity .
    ?iri schema:name ?label .
  }
  order by ?label
}
```

**Key requirements:**
- Use `CONSTRUCT` (not `SELECT`) to output RDF
- Always output `skos:prefLabel` for search indexing
- Include the entity type in the output
- Use `OPTIONAL` for optional properties

### 2. Update the Generation Script

Add an entry to `datasets-generator.sh`:

```bash
echo "construct the custom dataset"
time /static-publishing-backend rdf-data-aggregator \
  $custom_dir_path/ "*.ttl" \
  $datasets_generator_dir_path/custom.sparql \
  $datasets_dir_path/custom.ttl
```

### 3. Update Configuration

If the dataset is used in form dropdowns:

- Update SHACL shapes to reference the dataset URL (e.g., `dataset:custom.ttl`)
- Ensure `datasetBaseUrl` in `config.json` is correctly configured

### 4. Test Locally

You can test datasets generation locally using the template's Docker workflow:

```bash
# Clone the template
git clone https://github.com/Music-Metadata-Tools/MerMEId-MeLODy-Template.git
cd MerMEId-MeLODy-Template

# Run the generator script (requires Docker)
docker run --rm -v $(pwd):/repo registry.gitlab.rlp.net/adwmainz/nfdi4culture/cdmd/static-publishing-backend/static-publishing-backend:latest \
  /repo/modules/datasets-generator/datasets-generator.sh
```

Generated files will appear in `public/datasets/`.

---

## Troubleshooting

### Datasets Are Not Published

**Problem:** Dropdowns are empty or search doesn't work.

**Solutions:**

1. Check that GitHub Pages is enabled (Settings → Pages → Source: GitHub Actions)
2. Verify the `datasetBaseUrl` in `config.json` matches your actual Pages URL
3. Check that the latest pipeline run succeeded (Actions tab in GitHub)
4. Clear the editor's browser cache (Ctrl+Shift+Delete)

### SPARQL Query Errors

**Problem:** Pipeline fails with a SPARQL error.

**Solutions:**

1. Check the Actions logs for the specific error
2. Verify namespace prefixes are defined (`prefix melod:`, `prefix skos:`, etc.)
3. Ensure the entity type in the query matches your actual entity classes
4. Test the query locally in an RDF editor or SPARQL IDE

### Dataset File Not Generated

**Problem:** A specific dataset file is missing.

**Solutions:**

1. Check the source directory exists and contains `.ttl` files
2. Verify the SPARQL query file exists in `modules/datasets-generator/`
3. Ensure the entry is added to `datasets-generator.sh`
4. Check that no syntax errors exists in the SPARQL query

---

## Related Topics

- [Repository Configuration](repo_config.md) — `datasetBaseUrl` and `projectDomain` settings
- [SHACL Shapes](shacl_shapes.md) — how form fields reference datasets
- [Entity Types](../data_model/entity_types.md) — data model and entity classes
- [Data Model Ontologies](../data_model/ontologies.md) — vocabularies used in datasets

---
