# Roadmap

This page gives a brief overview of planned and in-progress features for MerMEId MeLODy. The roadmap is maintained on [GitHub Discussions](https://github.com/Music-Metadata-Tools/MerMEId-MeLODy/discussions) and [GitHub Issues](https://github.com/Music-Metadata-Tools/MerMEId-MeLODy/issues) — those are the best places to follow progress, vote on priorities, or suggest new features.

---

## Features in progress

### Import

- **Import from MEI/XML** — The MEI2RDF tool enables conversion of existing MerMEId MEI (MEI 4.0) XML catalogues into the RDF/Turtle format used by MerMEId MeLODy. The tool uses a three-phase conversion process: Phase 1 detects and registers musical entities from MEI documents; Phase 2 enriches MEI files with entity references; Phase 3 generates RDF triples organized by entity type. Supports both GUI and CLI modes with configurable entity schemas and mappings, making it practical for projects migrating from the original MerMEId to the new RDF-based system. Currently in development and not yet publicly accessible, but will be available for open use in the future.

### Export

- **Export to various formats** — export scripts for producing formatted outputs from the RDF data, including MEI/XML, HTML catalogues, CSV, and other formats as needed by specific projects.

### Planned Features

- **Import from other formats** — importers for additional metadata formats (e.g. MARC, CSV, existing LOD datasets).
- Language Selector in the editor to switch the languages of the field labels and descriptions
- Multiple tabs to open more than one entity
- Split panel for the three components (Filesystem manager, Entity Editor and Entity Renderer) to adjust the size of each panel.

*[PLACEHOLDER: add further planned features here]*

---

## Contributing to the Roadmap

If you have a feature request or want to discuss priorities, please open a thread on [GitHub Discussions](https://github.com/Music-Metadata-Tools/MerMEId-MeLODy/discussions) or a feature request on [GitHub Issues](https://github.com/Music-Metadata-Tools/MerMEId-MeLODy/issues). The development team reviews requests regularly and welcomes input from the community.
