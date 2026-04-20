# Configuration

MerMEId MeLODy is configured at two distinct levels: the **editor** itself (which entity types are available and how their forms look) and the **data repository** (where data is published and what IRI scheme to use). SHACL shape files provide a third layer of configuration, driving the form fields for each entity type.

---

## Sections

- [Editor Configuration](editor_config.md) — the `editor-default.ttl` file: registering entity types and linking SHACL shapes
- [Repository Configuration](repo_config.md) — the `config.json` file: dataset URL and IRI prefix
- [SHACL Shapes](shacl_shapes.md) — how forms are defined and how to write or extend them

---

## At a Glance

```
Editor repository
└── configuration/
    ├── editor-default.ttl    ← which entity types exist and which SHACL shape each uses
    └── *.shacl               ← one shape file per entity type, defines form fields

Data repository
└── configuration/
    ├── config.json           ← datasetBaseUrl and projectDomain
    └── *.shacl               ← copy of shape files (or referenced by URL)
```

No build step is required for most configuration changes. Editing a SHACL shape or `editor-default.ttl` takes effect immediately on the next page load.
