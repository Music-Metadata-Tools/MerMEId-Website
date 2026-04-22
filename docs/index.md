# MerMEId MeLODy

**MerMEId MeLODy** is an open-source, browser-based metadata editor for creating and managing music-related metadata as Linked Open Data (LOD) with MEI output. It is the successor to [MerMEId](https://github.com/Edirom/MerMEId), the established metadata editor for thematic work catalogues originally developed at the Danish Centre for Music Editing.

The editor produces [RDF/Turtle](https://www.w3.org/TR/turtle/) output based on a FRBR-inspired data model, placing music metadata within the broader Linked Data landscape. It is designed for musicologists, music librarians, and developers who need to create structured, interoperable, and FAIR-compliant metadata for musical works, sources, persons, and events.

> **MerMEId MeLODy is currently in beta.** Functionality is complete enough for real cataloguing work, but the interface and data model may still change before a stable release. Feedback is very welcome at [cdmd@adwmainz.de](mailto:cdmd@adwmainz.de) or as Github [Issue](https://github.com/Music-Metadata-Tools/MerMEId-MeLODy/issues).

---

## Quick Start

<div class="text-center">
<a href="https://music-metadata-tools.github.io/MerMEId-MeLODy/" class="btn btn-primary" role="button">Open the Live Editor</a>
<a href="getting_started/" class="btn btn-primary" role="button">Getting Started Guide</a>
<a href="https://github.com/Music-Metadata-Tools/MerMEId-MeLODy" class="btn btn-default" role="button">View on GitHub</a>
</div>

---

## How It Works

MerMEId MeLODy connects your browser directly to a Git repository where all data files live. You clone the repository into the browser's virtual filesystem (backed by IndexedDB), edit entities using automatically generated forms, and push changes back to the remote — all without leaving the browser tab.

Each entity (person, work, source, place, etc.) is stored as a separate `.ttl` file inside a typed subfolder. The forms are generated on the fly from SHACL shape files stored in the repository, so the data model is fully transparent and customisable.

To get started you need a data repository. The quickest way is to create one from the [GitHub MerMEId MeLODy Template](https://github.com/Music-Metadata-Tools/MerMEId-MeLODy-Template) on GitHub or from the [GitLab MerMEId MeLODy Template](https://gitlab.rlp.net/adwmainz/nfdi4culture/cdmd/project_templates/mermeid-template) on GitLab RLP.

---

## Who Is It For?

- **Musicologists** creating or maintaining thematic catalogues of composers' works or sources
- **Music librarians** who need to describe sources, editions, and manuscripts as Linked Open Data
- **Developers** looking to extend or integrate a SHACL-driven RDF editor into a digital humanities workflow

---

## Key Features

- **Browser-only** — no installation, no backend server; everything runs in the browser
- **Git-backed storage** — data is stored in a standard Git repository on GitHub, GitLab, or any HTTPS-accessible Git host
- **RDF/Turtle output** — all metadata is stored as Linked Open Data, ready for SPARQL endpoints and LOD integration
- **MEI output** — MEI entity snippets and full MEI work files
- **SHACL-driven forms** — data entry forms are generated automatically from [SHACL shapes](https://www.w3.org/TR/shacl/) through the [shacl-forms](https://github.com/ULB-Darmstadt/shacl-form) webcomponent of the ULB Darmstadt, making the data model easy to read, validate, and extend
- **FRBR-based data model** — covers Work, Expression, Manifestation, and Item, plus Persons, Places, Venues, Events, Performance events, Instrumentations, Bibliography, and Letters
- **Cross-entity linking** — link entities to each other and preview linked records in a floating side panel
- **No build step** — the source code is plain HTML and JavaScript

---

## Beta Status

MerMEId MeLODy is **beta software** (version 0.8, released 2026-03-24). It is actively used in ongoing cataloguing projects, but you should:

- Back up your data repository regularly (it is a standard Git repository — cloning it elsewhere is sufficient)
- Expect occasional breaking changes between versions
- Report issues or unexpected behaviour on [GitHub Issues](https://github.com/Music-Metadata-Tools/MerMEId-MeLODy/issues) or by email to [cdmd@adwmainz.de](mailto:cdmd@adwmainz.de)

---

## Funding

MerMEId MeLODy is funded by the **Deutsche Forschungsgemeinschaft (DFG)** through the project [MerMEIding to the future](https://gepris.dfg.de/gepris/projekt/528785591), project number 528785591. It is developed by the Centre for Digital Music Documentation ([CDMD](https://www.adwmainz.de/forschung/centre-for-digital-music-documentation.html)) at the [Akademie der Wissenschaften und der Literatur | Mainz](https://www.adwmainz.de/) and the Zentrum Musik–Edition–Medien ([ZenMEM](https://zenmem.uni-paderborn.de/)) at the University of Paderborn.
