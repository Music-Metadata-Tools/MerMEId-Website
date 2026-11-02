# Datasets Generation

This document explains how MerMEId MeLODy generates and uses datasets for search indexes, dropdown menus and the graph view. Understanding this process is essential for setting up a data repository, customizing search behavior, and extending the system with new entity types.

---

## Overview

**Datasets** are aggregated RDF/Turtle files generated from your repository's data. They serve three functions in the editor:

1. **Search indexes** — power the entity search functionality that loads in the editor
2. **Cross-reference dropdowns** — supply the linked entity fields (e.g., "Birth Place", "Contributor") in the editor forms
3. **Graph view** — provide the relations between entities

The index files are kept up to date in two ways:

- **In the editor** — whenever you save, delete, unstage or synchronise an entity, the editor updates the corresponding index file directly in the browser. The updated index file is staged and shared together with your data.
- **In the CI/CD pipeline** — after every push to `main`, a pipeline in the data repository regenerates all index files and commits them back to the repository.

Without index files, the editor can read and edit entities, but the search, the graph view and the cross-reference dropdowns are empty.

---

## How Datasets Are Generated

### In the Editor

When an entity is saved, deleted or changed by a synchronisation, `VirtualFilesystem.generate_indexes_for_saved_file()`:

1. Determines the entity folder from the file path (e.g. `persons`)
2. Fetches the SPARQL query `modules/datasets-generator/<folder>.sparql` from the editor
3. Runs the CONSTRUCT query on the saved file with Oxigraph in the browser
4. Replaces the triples of this entity in `indexes/<folder>.ttl` (or removes them for a deleted file); the index file is created if it does not exist
5. Stages the index file and reloads the search

Because only the changed file is processed, context that a query takes from other folders (e.g. the place name of a venue or institution) may be missing until the CI/CD pipeline regenerates the index.

### The Pipeline

The template repository includes a **GitHub Actions workflow** (`.github/workflows/pages.yml`, "Build and Push Indexes") or a **GitLab CI pipeline** (`.gitlab-ci.yml`) that regenerates all datasets after every push to `main`:

```
Push to main
    ↓
Job fetch_sparql_queries
  - Download the current SPARQL queries from the editor repository
    ↓
Job generate_datasets
  - Run datasets-generator.sh
  - For each dataset:
      - Collect all RDF files from their source directory
      - Execute a SPARQL CONSTRUCT query
      - Output aggregated .ttl file to indexes/
    ↓
Job push-indexes (GitLab: push_indexes)
  - Commit the indexes/ folder with "Updated indexes [skip ci]"
  - Push to main
    ↓
Synchronise in the editor to receive the updated indexes
```

The `[skip ci]` tag in the commit message prevents the pipeline from running again for its own commit.

### Tools Used

The generation process requires:

- **`static-publishing-backend`** Docker container — provides the RDF aggregator and SPARQL engine
- **SPARQL CONSTRUCT queries** — transform entity data into searchable datasets. The queries are maintained in the editor repository ([`MerMEId-MeLODy/modules/datasets-generator/`](https://github.com/Music-Metadata-Tools/MerMEId-MeLODy/tree/main/modules/datasets-generator)) and are downloaded at build time by `fetch-sparql-queries.sh`. Set the variable `EDITOR_REPO_REF` to use another branch or tag of the editor (default: `main`). Copies of the queries in the data repository are not used.
- **GitHub Actions / GitLab CI** — orchestrates the workflow on every push

All of these are configured in the template; you do not need to set anything up manually. The pipeline only needs permission to push to `main` (see [Getting Started](../user_guide/index.md#creating-a-data-repository)).

---

## Dataset Files

When the pipeline runs, it generates Turtle (.ttl) files in the `indexes/` directory. Each dataset corresponds to an entity type. All entries are typed with an index class `melod:<Type>Entity` (e.g. `melod:PersonEntity`), which the SHACL shapes use for linked entity fields.

| Dataset File | Source Directory | Entity Type | Used For |
|---|---|---|---|
| `persons.ttl` | `persons/` | Person | Search, cross-references (e.g., contributors, publishers) |
| `works.ttl` | `works/` | Work | Search, cross-references (e.g., related works) |
| `expressions.ttl` | `expressions/` | Expression | Search, cross-references (related expressions, works and manifestations)|
| `manifestations.ttl` | `manifestations/` | Manifestation | Search, cross-references (related manifestations) |
| `items.ttl` | `items/` | Item | Search, cross-references (related manifestations and items) |
| `institutions.ttl` | `institutions/` | Institution | Search, cross-references (e.g., holding institutions/repositories) |
| `places.ttl` | `places/` | Place | Search, cross-references (e.g., birth place) |
| `venues.ttl` | `venues/` | Venue | Search, cross-references (e.g. performance venue) |
| `events.ttl` | `events/` | Event | Search, cross-references (e.g. historic events of a items creation) |
| `performanceEvents.ttl` | `performanceEvents/` | Performance Event | Search, cross-references (e.g. first performance of an expression) |
| `instrumentations.ttl` | `instrumentations/` | Instrumentation | Search, cross-references (e.g. instrumentation of an expression) |
| `bibliography.ttl` | `bibliography/` | Bibliography | Search, cross-references (e.g. evidences of a performance or work) |
| `letters.ttl` | `letters/` | Letter | Search, cross-references |
| `dataCollections.ttl` | `dataCollections/` | Data Collection | Search, cross-references (data collections of works and manifestations) |
| `dataCatalogs.ttl` | `dataCatalogs/` | Data Catalog | Cross-references (data catalog of a data collection) |

The search and the graph view load the index files listed in `INDEXES` in `modules/services/index-store-service.js` (all of the above except `dataCatalogs.ttl`). For the form dropdowns, all files in `indexes/` are used.

---

## SPARQL Query Reference

Each dataset is built using a **SPARQL CONSTRUCT query** that transforms entity data into a simplified searchable form. These queries are located in the editor repository in [`modules/datasets-generator/`](https://github.com/Music-Metadata-Tools/MerMEId-MeLODy/tree/main/modules/datasets-generator).

All queries output the index class (`a melod:<Type>Entity`) and `skos:prefLabel`; most of them also output `skos:related` links to connected entities, which are used by the graph view.

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

**Purpose:** Index all works with titles, alternative titles and classifications.

**Output properties:**

- `skos:prefLabel` — main title
- `skos:altLabel` — uniform title, alternative titles and other titles (if present)
- `skos:broader` — work classification (if present)
- `skos:related` — expressions, contributors, work parts, bibliography and data collections

**Query logic:**

- Extracts the main title (type: `MainTitle`)
- Optionally includes uniform, alternative and other titles as searchable variants
- Sorted by main title

---

### Expressions Dataset

**File:** `expressions.sparql`

**Purpose:** Index all expressions with labels including position and incipit text for identification.

**Output properties:**

- `skos:prefLabel` — expression label with optional position and incipit text
- `skos:broader` — classification (if present)
- `skos:related` — performances, first performance, place of creation, contributors, parts, components, historic events, instrumentation, bibliography, MEI incipit

**Query logic:**

- Uses `rdfs:label` as base label (falls back to the IRI)
- Optionally prepends the position and appends the incipit text in parentheses for disambiguation
- Sorted by label

---

### Manifestations Dataset

**File:** `manifestations.sparql`

**Purpose:** Index all manifestations with their titles and classifications.

**Output properties:**

- `skos:prefLabel` — manifestation title (with title type)
- `skos:broader` — classification (if present)
- `skos:related` — embodied expressions, parts, data collections, contributors, publisher, place of publication

**Query logic:**

- Extracts title from title subject (falls back to the IRI)
- Optionally includes classification for broader search
- Sorted by title

---

### Items Dataset

**File:** `items.sparql`

**Purpose:** Index all items with titles and classifications.

**Output properties:**

- `skos:prefLabel` — item label
- `skos:broader` — classification
- `skos:related` — manifestation, current keeper, parts, hands, provenance agents

**Query logic:**

- Uses item label from `rdfs:label` (falls back to the IRI)
- Includes classification
- Sorted by label

---

### Institutions Dataset

**File:** `institutions.sparql`

**Purpose:** Index all institutions with abbreviations or RISM sigla.

**Output properties:**

- `skos:prefLabel` — label with optional abbreviation or RISM siglum
- `skos:broader` — ("Organization" or "RISM") and place name, if present
- `skos:related` — place

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
- `skos:related` — place

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
- `skos:related` — place, contributors, bibliography

**Query logic:**

- Extracts event name from `rdfs:label`
- Sorted by name

---

### Performance Events Dataset

**File:** `performanceEvents.sparql`

**Purpose:** Index all performance events with dates and venues.

**Output properties:**

- `skos:prefLabel` — performance label with optional date and venue
- `skos:related` — contributors, venue, bibliography

**Query logic:**

- Uses performance event name as base
- Optionally appends date and venue information for context
- Combined label format: `"Name: Date Place"`
- Sorted by label

---

### Instrumentations Dataset

**File:** `instrumentations.sparql`

**Purpose:** Index all instrumentations.

**Output properties:**

- `skos:prefLabel` — instrumentation name
- `skos:related` — instrumentation groups

**Query logic:**

- Extracts name from `rdfs:label` (falls back to the IRI)
- Sorted alphabetically

---

### Bibliography Dataset

**File:** `bibliography.sparql`

**Purpose:** Index all bibliography entries with abbreviations or titles.

**Output properties:**

- `skos:prefLabel` — bibliography label with abbreviation or title
- `skos:related` — authors, editors, publisher, place, keeper

**Query logic:**

- Prefers `melod:hasAbbreviation` if present
- Falls back to title if no abbreviation exists
- Sorted by label

---

### Letters Dataset

**File:** `letters.sparql`

**Purpose:** Index all letters.

**Output properties:**

- `skos:prefLabel` — title of the letter
- `skos:related` — participants, places, holding institution

**Query logic:**

- Extracts the title from `rdfs:label`
- Sorted by label

---

### Data Collections Dataset

**File:** `dataCollections.sparql`

**Purpose:** Index all data collections.

**Output properties:**

- `skos:prefLabel` — name of the data collection (class `melod:DataCollectionEntity`)

---

### Data Catalogs Dataset

**File:** `dataCatalogs.sparql`

**Purpose:** Index the data catalog of the repository.

**Output properties:**

- `skos:prefLabel` — name of the data catalog (class `melod:DataCatalog`)

---

## The Generation Script

The file `modules/datasets-generator/datasets-generator.sh` in the data repository (see the [GitHub template](https://github.com/Music-Metadata-Tools/MerMEId-MeLODy-Template/blob/main/modules/datasets-generator/datasets-generator.sh)) orchestrates the entire process. It:

1. Sets up directory paths for all entity types
2. For each dataset, calls the `rdf-data-aggregator` tool with:

    - **Source directory:** where entity files are located (e.g., `persons/`)
    - **File pattern:** which files to include (e.g., `*.ttl`)
    - **SPARQL query:** the transformation logic, downloaded before by `fetch-sparql-queries.sh` (e.g., `sparql-queries/persons.sparql`)
    - **Output path:** where to write the result (e.g., `indexes/persons.ttl`)

3. Times each operation for performance monitoring

Example invocation:
```bash
/static-publishing-backend rdf-data-aggregator \
  $persons_dir_path/ \
  "*.ttl" \
  $datasets_generator_dir_path/persons.sparql \
  $indexes_dir_path/persons.ttl
```

The script `modules/datasets-generator/fetch-sparql-queries.sh` downloads all queries listed in its loop from `https://raw.githubusercontent.com/music-metadata-tools/MerMEId-MeLODy/$EDITOR_REPO_REF/modules/datasets-generator/`.

---

## Committing the Indexes

After datasets are generated, the pipeline commits them to the repository:

1. Datasets are created in the `indexes/` directory
2. The job `push-indexes` (GitLab: `push_indexes`) commits the folder with the message `Updated indexes [skip ci]` — only for the `main` branch and only if something has changed
3. The commit is pushed to `main`; the editor receives the new indexes when you synchronise

**Required configuration:**

- **GitHub:** the workflow requests `contents: write` itself. If your organisation restricts workflow permissions or `main` is protected, allow GitHub Actions to push to `main`.
- **GitLab:** the job pushes with the CI/CD job token. Allow Git push requests with the job token in **Settings → CI/CD → Job token permissions**, or adapt the job to use a project access token.

---

## Data Validation

The GitHub template additionally includes a **Data Validation** workflow (`.github/workflows/validate.yml`). It is started manually (Actions → Data Validation → Run workflow) and validates the `.ttl` files of the data folders against the SHACL shapes of the editor (fetched from the `main` branch of the editor repository).

It can also be run locally with Node.js:

```bash
npm install
npm test
```

---

## Troubleshooting

### Datasets Are Not Generated or Not Up to Date

**Problem:** Dropdowns are empty or search doesn't work.

**Solutions:**

1. Check that the `indexes/` folder exists in your repository and contains files
2. Click **Reload indexes** in the editor toolbar
3. Synchronise the repository to receive indexes committed by the pipeline
4. Check that the latest pipeline run ("Build and Push Indexes") succeeded (Actions tab in GitHub, Build → Pipelines in GitLab) and that it was allowed to push to `main`
5. Verify that `datasetBaseUrl` in `config.json` is set (any non-empty value)

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
2. Verify the SPARQL query file exists in `modules/datasets-generator/` of the **editor** repository — the editor finds a query by the folder name (`<folder>.sparql`)
3. Ensure the query is listed in `fetch-sparql-queries.sh` and an entry is added to `datasets-generator.sh` of the data repository
4. Check that no syntax errors exists in the SPARQL query

---

## Related Topics

- [Repository Configuration](repo_config.md) — `datasetBaseUrl` and `projectDomain` settings
- [SHACL Shapes](shacl_shapes.md) — how form fields reference datasets
- [Entity Types](../data_model/entity_types.md) — data model and entity classes
- [Data Model Ontologies](../data_model/ontologies.md) — vocabularies used in datasets

---
