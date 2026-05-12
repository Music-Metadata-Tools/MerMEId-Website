# The Interface

This page gives a complete walkthrough of the MerMEId MeLODy user interface. The editor is divided into three main areas: a left panel with search and file navigation, a central editing area, and a right output panel.

---

## Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│  BETA                MerMEId MeLODy                [GitHub]         │
├─────────────────┬───────────────────────────────┬───────────────────┤
│ Search Panel    │  Editor Panel                 │  Output Panel     │
│                 │                               │                   │
│ Entity Search   │  Entity Editor                │  XML | RDF |      │
│                 │                               │  Preview          │
│                 │  [toolbar]                    │                   │
│                 │  Entity type + IRI            │                   │
│                 │  shacl-form                   │                   │
│ Repository      │                               │                   │
│ Panel           │                               │                   │
│                 │                               │                   │
│ Filesystem      │                               │                   │
│ Manager         │                               │                   │
│                 │                               │                   │
│                 │                               │                   │
├─────────────────┴───────────────────────────────┴───────────────────┤
│  Footer                                                             │
└─────────────────────────────────────────────────────────────────────┘
```

The **header** shows the BETA badge (click it to read a popup with the beta disclaimer and contact information), the application title, and a GitHub link.

The **footer** shows the logos of the funding and partner institutions: DFG, CDMD, ZenMEM, and ADW Mainz.

---

## The Search Panel

**Component:** Entity Search (top of the left column)

The search panel let you find entities across the entire dataset without navigating through the file tree. You can search by names, titles, composers, alternative titles, or classifications.

### Controls

- **Type filter dropdown** — narrows your search to a specific entity type (Person, Place, Work, Institution, etc.). Select **All** to search across all entity types.
- **Search input field** — type a name, keyword, or phrase to find matching entities. The search is case-insensitive and looks across multiple fields:
  - **Primary labels** — the main name or title of an entity
  - **Alternative labels** — alternate names or spellings registered for an entity (e.g., if "J.S. Bach" and "Johann Sebastian Bach" are both registered) or alternative/uniform titles
  - **Classifications** — broader categories an entity belongs to
- **Results list** — shows matching entities with a label indicating their type. Click any result to open it in the editor. The file tree on the left will automatically navigate to that file.

### How the Search Index Works

- **Start with a partial name** — you don't need to type the full name; "Bach" will find all Bachs, "Sonata" will find all sonatas
- **Use type filters** — if you're looking for a specific person but the name is common, first filter by **Person**, then search
- **Look for alternative spellings** — if you can't find an entity by its common name, the search may know it by another spelling

### Important Note About Search Results

The search shows entities from the **last pushed state** of the repository, not your unsaved local changes. This means:

- If you've just created or edited an entity locally but haven't pushed yet, it won't show up in search results.
- New results appear after the repository is pushed and the server indexes are updated (this typically takes 1-2 minutes).
- To sync your local view with the latest published results, use the **Synchronise** button in the Repository Panel.

---

## The Repository Panel

**Component:** Filesystem Manager (below the search panel, left side)

The repository panel shows your cloned repositories, their folder structure, and the file staging area.

### Toolbar Buttons

| Icon | Action | Description |
| :--- | :--- | :--- |
| Folder-plus | **Add repository** | Opens the clone dialog to connect a new Git repository |
| Arrow-clockwise | **Synchronise** | Pulls the latest changes from the remote repository |
| Folder | **Rename repository** | Renames the display name shown in the editor (does not affect the remote) |
| Folder-minus | **Remove repository** | Removes the repository from the browser's virtual filesystem (does not delete the remote) |

These buttons operate on whichever repository is currently selected (highlighted in the tree).

### The Repository Tree

Each repository is shown as a collapsible tree. Click the arrow next to a folder to expand it. The tree is **lazy-loaded** — folder contents are fetched from the virtual filesystem only when you expand a node.

Each tree node is one of:

- **Repository root** (top level) — click to select the repository; the name highlights to confirm it is active
- **Folder** (e.g. `persons/`, `works/`) — expand to see the files inside
- **File** (e.g. `12345.ttl`) — click to open in the editor

Files that have been modified and saved (staged) are visually indicated in the tree so you can see which records have pending changes.

### The Share Files Section

At the bottom of the repository panel is the **Share files** section. This lists all files that have been saved locally but not yet pushed to the remote repository.

- Each staged file has a **checkbox** next to it.
- Check the boxes for the files you want to include in the next commit.
- Click the **"Share files"** button (cloud-upload icon) to commit and push the selected files.
- Click the **"Unstage files"** button (arrow-counterclockwise icon) to discard local changes and restore the file from the last committed state.

Deleted entities also appear here with a `-deleted` suffix in the filename.

---

## The Editor Panel

**Component:** Entity Editor (centre column)

The editor panel is the main working area. It displays the form for the currently open entity.

### Toolbar Buttons

| Icon | Action | Description |
| :--- | :--- | :--- |
| Plus | **New** | Opens the entity type dialog to create a new entity |
| Floppy disk / Save | **Save** | Saves the current entity to the virtual filesystem and stages it for git |
| Arrow-counterclockwise | **Undo** | Reverts the form to the last saved state (discards unsaved changes) |
| File-minus | **Remove entity** | Deletes the current entity (prompts for confirmation) |

### Entity Type and IRI Display

Below the toolbar, the editor shows:

- The **entity type** (e.g. `Person`, `Work`) as a label
- The **entity IRI** — the unique identifier for this entity (e.g. `urn:uuid:persons/2847362910`)

These are read-only and generated automatically when a new entity is created.

### The Form (shacl-form)

The form is rendered by the [shacl-form](https://github.com/ulb-darmstadt/shacl-form) web component. It is generated automatically from the SHACL shape file for the entity type. Each field in the form corresponds to an RDF property defined in the SHACL shape.

See [Working with Entities](entities.md) for a detailed description of how to fill in the form.

### Unsaved Changes Indicator

When you make any change to the form, the **Save** button turns blue and a dot indicator appears on or near the button. This is a reminder that the current state has not been written to the virtual filesystem yet. Clicking a different file in the tree without saving will trigger a warning.

---

## The Output Panel

**Component:** Entity Renderer (right column, tab group)

The output panel on the right side of the screen renders the current entity's data in different formats. It updates when the form content is saved, not automatically.

### XML Tab

Shows the entity data converted from JSON-LD to **MEI/XML** or **TEI/XML** format. This conversion is performed by entity-type-specific converter modules. The output is suitable for copying into MEI or TEI documents or passing to MEI-aware tools.

A **copy** button at the top of the tab copies the full XML content to the clipboard.

### RDF Tab

Shows the **RDF/Turtle** output — the exact content that is inside the `.ttl` file. This is produced directly by `shacl-form.serialize()` and represents the canonical stored format for the entity.

A **copy** button at the top of the tab copies the Turtle content to the clipboard.

### Preview Tab

Shows a **read-only rendering** of the current entity using a second `shacl-form` instance in `data-view` mode. This gives a cleaner, label-based view of the data without the editable input controls. It is useful for reviewing entries before saving.

---

## Mobile Layout

On small screens, the three-panel layout collapses. A **toggle button** allows switching between the left panel and the main editing area. The output panel may require scrolling or is hidden depending on screen width.

For the best experience on small screens, use landscape orientation. Full multi-panel layout requires a screen width of at least 1024px.
