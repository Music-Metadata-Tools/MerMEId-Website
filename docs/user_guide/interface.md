# The Interface

This page gives a complete walkthrough of the MerMEId MeLODy user interface. The editor is divided into three main areas: a left panel with search and file navigation, a central editing area, and a right output panel.

---

## Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│  BETA          MerMEId MeLODy        [Language: EN] [Docs] [GitHub] │
├─────────────────┬───────────────────────────────┬───────────────────┤
│ Search Panel    │  Editor Panel                 │  Output Panel     │
│                 │                               │                   │
│ Entity Search   │  Entity Editor                │  XML | RDF |      │
│                 │                               │  Preview | Graph  │
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

The **header** shows the CDMD logo with the BETA badge (click it to read a popup with the beta disclaimer and contact information), the application title, and three buttons:

- **Language** — switches the user interface between English and German (**Language: EN** / **Language: DE**). The selected language is used for buttons, entity type names and the labels in the forms. Some dialogs and messages are currently only available in English. The selection is not remembered after reloading the page.
- **Docs** — opens this documentation.
- **GitHub** — opens the source code repository of the editor.

The **footer** shows the logos of the funding and partner institutions: DFG, CDMD, ZenMEM, and ADW Mainz.

---

## The Search Panel

**Component:** Entity Search (top of the left column)

The search panel let you find entities across the entire dataset without navigating through the file tree. You can search by names, titles, composers, alternative titles, or classifications.

### Controls

- **Type filter dropdown** — narrows your search to a specific entity type (Person, Place, Work, Institution, Data Collection, etc.). Select **All** to search across all entity types.
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

The search reads the index files in the `indexes/` folder of the selected repository. The editor updates these index files itself whenever you save, delete or unstage an entity, and when you synchronise. This means:

- Entities you have just created or edited appear in the search results **immediately after saving**, even before you share them.
- Changes made by other people appear after you **synchronize** the repository.
- To reload the search indexes manually, use the **Reload indexes** button (arrow-clockwise icon, far left in the editor toolbar).

---

## The Repository Panel

**Component:** Filesystem Manager (below the search panel, left side)

The repository panel shows your cloned repositories, their folder structure, and the file staging area.

### Toolbar Buttons

| Icon | Action | Description |
| :--- | :--- | :--- |
| Folder-plus | **Add repository** | Opens the dialog to connect a new Git repository or a local folder |
| Folder-minus | **Remove repository** | Removes the repository from the browser's storage (does not delete the remote) |
| Folder | **Rename repository** | Renames the display name shown in the editor (does not affect the remote) |
| Cloud-download | **Synchronize repository** | Downloads the latest changes from the remote repository |
| File-minus | **Remove entity** | Deletes the entity file selected in the tree |
| Gear | **Repository settings** | Changes the username and personal access token stored for the repository |
| Journal-bookmark | **Catalog metadata** | Opens the metadata form of the repository's data catalog |

These buttons operate on whichever repository is currently selected (highlighted in the tree). They are enabled once you have clicked or expanded a repository. **Synchronize repository** is disabled for local folders without a configured remote.

### The Repository Settings Dialog

Opens a dialog with the **username** and **personal access token** used for the selected repository. Use it to add a token to a repository that was added without one, or to replace an expired token. Click **Save** to store the new values.

### The Catalog Metadata Dialog

Each repository contains one data catalog (`dataCatalogs/catalog.ttl`) that describes the catalogue as a whole — its name, contributors with their roles, and a description. The dialog shows the catalog in a form; **Save** is enabled once the form is valid. Saving stages the file in the Share Files section.

### The Repository Tree

Each repository is shown as a collapsible tree. Click the arrow next to a folder to expand it. The tree is **lazy-loaded** — folder contents are read from the browser's storage only when you expand a node.

Each tree node is one of:

- **Repository root** (top level) — click to select the repository; the name highlights to confirm it is active
- **Folder** (e.g. `persons/`, `works/`) — expand to see the files inside
- **File** (e.g. `12345.ttl`) — click to open in the editor

Files that have been modified and saved (staged) are visually indicated in the tree so you can see which records have pending changes.

### The Share Files Section

At the bottom of the repository panel is the **Share files** section. This lists all files that have been saved locally but not yet pushed to the remote repository.

- Each staged file is listed with its path (e.g. `persons/2847362910.ttl`) and a **checkbox** next to it. Click the file name to open it in the editor.
- Check the boxes for the files you want to include in the next commit, or use the **Select all** button (check-square icon) to select or deselect all files.
- Optionally enter a **Commit-Message**. If you leave it empty, the date and your username are used.
- Click the **"Share files"** button (cloud-upload icon) to commit and push the selected files. The button is enabled once at least one file is selected.
- Click the **"Unstage files"** button (arrow-counterclockwise icon) to discard local changes and restore the files to the last synchronised state.

Deleted entities also appear here with a `-deleted` suffix in the filename, if the file already existed in the remote repository.

---

## The Editor Panel

**Component:** Entity Editor (centre column)

The editor panel is the main working area. It displays the form for the currently open entity.

### Toolbar Buttons

| Icon | Action | Description |
| :--- | :--- | :--- |
| Arrow-clockwise | **Reload indexes** | Reloads the search and dropdown indexes from the repository's `indexes/` folder |
| File-plus | **New** | Opens the entity type dialog to create a new entity |
| Floppy disk | **Save** | Saves the current entity to the browser's storage and stages it for sharing |
| File-minus | **Delete** | Deletes the current entity (prompts for confirmation) |
| Arrow-counterclockwise | **Undo** | Discards all unsaved changes and resets the form to the state in which the entity was loaded |

### Entity Type and IRI Display

Below the toolbar, the editor shows:

- The **entity type** (e.g. `Person`, `Work`) as a label
- The **entity IRI** — the unique identifier for this entity (e.g. `urn:uuid:persons/2847362910`)

These are read-only and generated automatically when a new entity is created.

### The Form (shacl-form)

The form is rendered by the [shacl-form](https://github.com/ulb-darmstadt/shacl-form) web component (in the MerMEId-specific version [mermeid-shacl-form](https://github.com/Music-Metadata-Tools/mermeid-shacl-form)). It is generated automatically from the SHACL shape file for the entity type. Each field in the form corresponds to an RDF property defined in the SHACL shape.

See [Working with Entities](entities.md) for a detailed description of how to fill in the form.

### Unsaved Changes Indicator

When you make any change to the form, the **Save** button turns blue and a dot indicator appears on or near the button. This is a reminder that the current state has not been written to the virtual filesystem yet. Clicking a different file in the tree without saving will trigger a warning.

---

## The Output Panel

**Component:** Entity Renderer (right column, tab group)

The output panel on the right side of the screen renders the current entity's data in different formats. The XML, RDF and Preview tabs update when an entity is opened or saved, not while you are typing. The **Preview** tab is shown by default.

### XML Tab

Shows the entity data converted from JSON-LD to **MEI/XML** or **TEI/XML** format. This conversion is performed by entity-type-specific converter modules. The output is suitable for copying into MEI or TEI documents or passing to MEI-aware tools.

A **copy** button at the top of the tab copies the full XML content to the clipboard.

### RDF Tab

Shows the **RDF/Turtle** output — the exact content that is inside the `.ttl` file. This is produced directly by `shacl-form.serialize()` and represents the canonical stored format for the entity.

A **copy** button at the top of the tab copies the Turtle content to the clipboard.

### Preview Tab

Shows a **read-only rendering** of the current entity using a second `shacl-form` instance in `data-view` mode. This gives a cleaner, label-based view of the data without the editable input controls. It is useful for reviewing entries before pushing.

### Graph Tab

Shows the **related entities** of the current entity as an interactive graph. The relations are taken from the repository's indexes, so entities linking to the current entity are shown as well as entities it links to.

- Nodes are coloured by entity type; a legend and a **type filter** ("Related: All / …") let you restrict the view to one type.
- Click a node to expand its own relations. If an entity has many relations, only the first ones are shown together with a **"+N more"** node.
- Use the zoom buttons to zoom in, zoom out or reset the view, and switch between the **graph** and a **list** view.
- The eye icon on a node opens the entity in the floating preview panel (see [Previewing Linked Entities](entities.md#previewing-linked-entities)).

---

## Mobile Layout

On small screens, the three-panel layout collapses. A **toggle button** allows switching between the left panel and the main editing area. The output panel may require scrolling or is hidden depending on screen width.

For the best experience on small screens, use landscape orientation. Full multi-panel layout requires a screen width of more than 900px.
