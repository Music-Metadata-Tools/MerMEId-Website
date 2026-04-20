# Entity IRIs

Every entity in MerMEId MeLODy has a unique IRI (Internationalized Resource Identifier) that serves as its persistent identifier throughout the dataset and in all RDF output.

---

## Default Format: `urn:uuid:`

By default, entity IRIs use the `urn:uuid:` scheme combined with the entity folder name and a random integer:

```
urn:uuid:<folder>/<randomInteger>
```

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
@prefix skos:   <http://www.w3.org/2004/02/skos/core#> .
@prefix owl:    <http://www.w3.org/2002/07/owl#> .

<urn:uuid:persons/2847362910>
    a melod:Person ;
    schema:familyName "Telemann" ;
    schema:givenName "Georg Philipp" ;
    skos:prefLabel "Georg Philipp Telemann" ;
    owl:sameAs <http://d-nb.info/gnd/11862119X> .
```

Cross-references between entities also use IRIs directly:

```turtle
<urn:uuid:expressions/1234567890>
    a melod:Expression ;
    melod:isRealisationOf <urn:uuid:works/9876543210> ;
    melod:composer <urn:uuid:persons/2847362910> .
```

---

## The `dataset:` Namespace Patch

SHACL shape files use a special `dataset:` namespace as a placeholder for the published dataset index URL. At runtime, the editor replaces `dataset:` with the value of `datasetBaseUrl` from `config.json`. This allows the same SHACL shapes to work with any repository without modification.

For example, a Birth Place field in `person.shacl` might reference:

```turtle
sh:class melod:Place ;
melod:index <dataset:places.ttl> .
```

At runtime this becomes:

```
https://yourname.github.io/my-catalogue/datasets/places.ttl
```

See [Repository Configuration](../configuration/repo_config.md) for details on how `datasetBaseUrl` is configured.
