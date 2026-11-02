# Entity IRIs

Every entity in MerMEId MeLODy has a unique IRI (Internationalized Resource Identifier) that serves as its persistent identifier throughout the dataset and in all RDF output.

---

## Default Format: `urn:uuid:`

By default, entity IRIs use the `urn:uuid:` scheme combined with the entity folder name and a random integer:

```
urn:uuid:<folder>/<IDprefix><randomInteger>
```

The `IDprefix` is optional and empty by default (see [Repository Configuration](../developer/repo_config.md#idprefix)).

**Examples:**

```
urn:uuid:persons/2847362910
urn:uuid:works/1031334055
urn:uuid:places/774035217
urn:uuid:expressions/9182736450
```

Each entity's IRI is also the name of its `.ttl` file in the repository:

```
persons/2847362910.ttl   →  IRI: urn:uuid:persons/2847362910
works/1031334055.ttl     →  IRI: urn:uuid:works/1031334055
```

---

## IRI Generation

New entity IRIs are generated using the Web Crypto API at entity creation time:

```js
let array = new Uint32Array(1);
self.crypto.getRandomValues(array);
return array[0]; // a random 32-bit unsigned integer
```

This gives approximately **4 billion** possible values per entity type. Collisions are statistically negligible for the scale of musicological catalogues.

### Fixed IRIs

Some IRIs are not random. Each repository has exactly one data catalog and two default data collections, which the editor creates automatically when a repository is selected:

```
urn:uuid:dataCatalogs/catalog
urn:uuid:dataCollections/works
urn:uuid:dataCollections/manifestations
```

The IRI is **permanent**: once an entity is created and committed to the repository, its IRI should not be changed. All other entities that reference it (via linked fields) store the IRI directly — changing it would break those references.

---

## Configurable Prefix: `projectDomain`

The IRI prefix can be changed from `urn:uuid:` to an HTTPS namespace by setting `projectDomain` in `configuration/config.json`:

```json
{
  "projectDomain": "https://example.org/"
}
```

With this configuration, new entities receive IRIs like:

```
https://example.org/persons/2847362910
https://example.org/works/1031334055
```

### When to use a custom domain

- If you want your entities to have **dereferenceable URIs** — HTTP(S) URIs that resolve to a web page or RDF document when looked up
- If your institution has a stable web presence and a namespace policy for Linked Data identifiers
- For catalogues intended to be integrated into the broader LOD ecosystem (e.g. accessible from Wikidata or other knowledge graphs)

### Important caveat

Changing `projectDomain` after entities have been created **does not retroactively update existing IRIs**. Existing entities keep their original `urn:uuid:` IRIs. Only newly created entities will use the new prefix. This means a mixed-prefix dataset is possible if the domain is changed mid-project — generally it is best to decide on the IRI scheme before creating any entities.

---

## IRIs in Turtle Output

Every Turtle file in the repository uses the entity's IRI as the subject of all statements:

```turtle
@prefix melod:  <https://lod.academy/melod/vocab/ontology#> .
@prefix schema: <https://schema.org/> .
@prefix owl:    <http://www.w3.org/2002/07/owl#> .

<urn:uuid:persons/2847362910>
    a melod:Person ;
    schema:familyName "Telemann" ;
    schema:givenName "Georg Philipp" ;
    owl:sameAs <http://d-nb.info/gnd/11862119X> .
```

Cross-references between entities also use IRIs directly:

```turtle
<urn:uuid:works/9876543210>
    a melod:Work ;
    lrmoo:R3_is_realised_in <urn:uuid:expressions/1234567890> ;
    melod:hasContribution [
        melod:hasAgent <urn:uuid:persons/2847362910> ;
        melod:hasRole <…> # role term from the MeLODy vocabulary, e.g. composer
    ] .
```

---

## Linked Entities in the Forms

Linked entity fields in the SHACL shapes declare the index class of the target type, e.g. `sh:class melod:PlaceEntity`. When a form is opened, the editor appends all files from the repository's `indexes/` folder to the SHACL shape, so `shacl-form` can offer every entity typed `melod:PlaceEntity` in the dropdown. The IRI of the selected entity is then stored in the Turtle file.

Earlier versions used a `dataset:` namespace in the shapes that was replaced at runtime with the `datasetBaseUrl`; this mechanism has been removed.

See [Repository Configuration](../developer/repo_config.md) and [SHACL Shapes](../developer/shacl_shapes.md) for details.
