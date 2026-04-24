# MerMEId MeLODy

**MerMEId MeLODy** is an open-source, browser-based metadata editor designed specifically for creating well-formed [MEI (Music Encoding Initiative)](https://music-encoding.org/) metadata for thematic work and source catalogues. It is the successor to [MerMEId](https://github.com/Edirom/MerMEId), the established metadata editor for thematic work catalogues originally developed at the Danish Centre for Music Editing.

The editor produces complex [RDF/Turtle](https://www.w3.org/TR/turtle/) output that directly cooperates with MEI structures to ensure standards-compliant metadata. The RDF data model is precisely aligned with the [MEI Schema (v5.1)](https://music-encoding.org/guidelines/v5/content/index.html) and the FRBR (Functional Requirements for Bibliographic Records) conceptual model, so all structured metadata can be reliably transformed into MEI/XML for use in MEI-based workflows, research infrastructure, and digital editions. It is designed for musicologists, music librarians, and developers who need to create structured, interoperable, and FAIR-compliant metadata for musical works, sources, persons or events.

> **MerMEId MeLODy is currently in beta.** Functionality is complete enough for real cataloguing work, but the interface and data model may still change before a stable release. Feedback is very welcome at [cdmd@adwmainz.de](mailto:cdmd@adwmainz.de) or as Github [Issue](https://github.com/Music-Metadata-Tools/MerMEId-MeLODy/issues).

---


<div class="text-center">
<a href="https://music-metadata-tools.github.io/MerMEId-MeLODy/" class="btn btn-primary" role="button">Open the Live Editor</a>
<a href="user_guide/" class="btn btn-primary" role="button">Getting Started Guide</a>
<a href="https://github.com/Music-Metadata-Tools/MerMEId-MeLODy" class="btn btn-default" role="button">View on GitHub</a>
</div>

---

## How It Works

MerMEId MeLODy connects your browser directly to a Git repository where all data files live. You clone the repository into the browser's virtual filesystem (backed by IndexedDB), edit entities using automatically generated forms, and push changes back to the remote — all without leaving the browser tab.

Each entity (person, work, source, place, etc.) is stored as a separate `.ttl` file inside a typed subfolder using the MeLODy ontology, which is structurally aligned with MEI elements and FRBR hierarchies. The forms are generated on the fly from SHACL shape files stored in the repository, ensuring that your data entry follows MEI-compatible structures. The data model is fully transparent and customisable — you can adjust which fields appear in forms while maintaining compliance with MEI output requirements.

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
- **FRBR + MEI-based data model** — implements the MEI-compatible FRBR hierarchy (Work, Expression, Manifestation, Item) plus contextual entities (Persons, Places, Venues, Events, Performance events, Instrumentations, Bibliography, Letters); the MeLODy ontology structure mirrors both FRBR levels and MEI elements to ensure semantic precision
- **Cross-entity linking** — link entities to each other and preview linked records in a floating side panel
- **No build step** — the source code is plain HTML and JavaScript

---

## Beta Status

MerMEId MeLODy is **beta software** (version 0.8, released 2026-03-24). It is actively used in ongoing cataloguing projects, but you should:

- Back up your data repository regularly (it is a standard Git repository — cloning it elsewhere is sufficient)
- Expect occasional breaking changes between versions
- Report issues or unexpected behaviour on [GitHub Issues](https://github.com/Music-Metadata-Tools/MerMEId-MeLODy/issues) or by email to [cdmd@adwmainz.de](mailto:cdmd@adwmainz.de)

---


## Citing MerMEId MeLODy

If you use MerMEId MeLODy in your research, please cite it as:

> Licciulli, Carlo; Teodorescu, Claudius; Schmitz, Annabella; Zwick, Robert; Richts-Matthaei, Kristina (2026). *MerMEId MeLODy* (Version 0.8). MIT License. [https://github.com/Music-Metadata-Tools/MerMEId-MeLODy](https://github.com/Music-Metadata-Tools/MerMEId-MeLODy)

A `CITATION.cff` file is included in the repository for automated citation tools (Zotero, GitHub citation export, etc.).

---


## Funding

MerMEId MeLODy is funded by the **Deutsche Forschungsgemeinschaft (DFG)** through the project [MerMEIding to the future](https://gepris.dfg.de/gepris/projekt/528785591), project number 528785591. It is developed by the Centre for Digital Music Documentation ([CDMD](https://www.adwmainz.de/forschung/centre-for-digital-music-documentation.html)) at the [Akademie der Wissenschaften und der Literatur | Mainz](https://www.adwmainz.de/) and the Zentrum Musik–Edition–Medien ([ZenMEM](https://zenmem.uni-paderborn.de/)) at the University of Paderborn.
