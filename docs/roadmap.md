# Roadmap

This page gives a brief overview of planned and in-progress features for MerMEId MeLODy. The roadmap is maintained on [GitHub Discussions](https://github.com/Music-Metadata-Tools/MerMEId-MeLODy/discussions) and [GitHub Issues](https://github.com/Music-Metadata-Tools/MerMEId-MeLODy/issues) — those are the best places to follow progress, vote on priorities, or suggest new features.

---

## Features in progress

### Import

**Import from MEI/XML** 

- The MEI2RDF tool enables conversion of existing MerMEId MEI (MEI 4.0) XML catalogues into the RDF/Turtle format used by MerMEId MeLODy. 

The tool uses a three-phase conversion process:

1. Detection and deduplication of entities from MEI documents
2. Enrichment of MEI files with entity IRIs
3. Generation of RDF triples organized by entity type

- Supports both GUI and CLI modes with configurable entity schemas and mappings, making it practical for projects migrating from the original MerMEId to the new RDF-based system
- Currently in development and not yet publicly accessible, but will be available for open use in the future.

### Export

**Export to MEI/XML and HTML** 

- The RDF Template Engine enables transformation of RDF/Turtle data into multiple structured output formats. The engine is built on a flexible pipeline architecture using SPARQL queries and Jinja2 templates. It extracts data from RDF graphs using customizable SPARQL queries, constructs context objects with hierarchical entity relationships (Works → Expressions → Manifestations → Items), and renders them through modular Jinja2 templates. 
- The engine will support three output modes: individual entities, complete files with related resources, and register listings of all entities of a type. Outputs can be generated in MEI (Music Encoding Initiative) XML format for use in the original MerMEId or other tools, HTML catalogues for web publication, or TEI (Text Encoding Initiative) XML for some entities like persons and places. The system is extensible with custom SPARQL queries for entity extraction and custom templates for output generation. The tool can run locally via CLI for batch processing or be integrated into CI/CD pipelines via the `static-publishing-backend` Docker container for automated publishing
- Currently in development and not yet publicly accessible, but will be available for open use in the future. 
- Future versions may also support additional export formats such as CSV and other structured formats as needed by specific projects.

### Planned Features

- **Import from other formats** — importers for additional metadata formats (e.g. MARC, CSV, existing LOD datasets).
- Language Selector in the editor to switch the languages of the field labels and descriptions
- Multiple tabs to open more than one entity
- Split panel for the three components (Filesystem manager, Entity Editor and Entity Renderer) to adjust the size of each panel.

*[PLACEHOLDER: add further planned features here]*

---

## Contributing to the Roadmap

If you have a feature request or want to discuss priorities, please open a thread on [GitHub Discussions](https://github.com/Music-Metadata-Tools/MerMEId-MeLODy/discussions) or a feature request on [GitHub Issues](https://github.com/Music-Metadata-Tools/MerMEId-MeLODy/issues). The development team reviews requests regularly and welcomes input from the community.
