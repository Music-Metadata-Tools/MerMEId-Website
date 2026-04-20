# Working with Entities

This page covers the full lifecycle of an entity in MerMEId MeLODy — from creating a new record through filling in the form, linking to other entities, saving, undoing changes, and deleting.

---

## Creating a New Entity

1. Make sure a repository is **selected** (highlighted) in the left panel. If no repository is active, the "New" button is disabled.
2. Click the **"New"** button (plus icon) in the editor toolbar.
3. A dialog opens listing all available entity types. Select the type you want to create (e.g. **Person**, **Work**, **Place**).
4. Click **"Create"** (or double-click the type). The dialog closes and a blank form opens in the editor panel.

A new entity IRI is generated automatically at this point (see below). The entity does not exist as a file yet — it is only in memory until you save.

### Auto-Generated IRIs

Every entity gets a unique IRI when it is created. By default, IRIs use the `urn:uuid:` scheme and take the form:

```
urn:uuid:<folder>/<randomInteger>
```

For example, a new person entity might get the IRI `urn:uuid:persons/2847362910`. The random integer is generated using the Web Crypto API (`crypto.getRandomValues`), giving a 32-bit unsigned integer.

The IRI scheme can be changed per repository by setting the `projectDomain` field in `configuration/config.json`. For example, setting `projectDomain` to `https://example.org/` would produce IRIs like `https://example.org/persons/2847362910`.

The IRI is displayed below the toolbar in the editor panel and cannot be changed after creation.

---

## Filling in the Form

The form is generated automatically from the SHACL shape for the entity type. Different field types appear depending on the property being described.

### Field Types

**Text fields** — plain literal values. Type directly into the input. Most label and name fields are text fields (e.g. Surname, First name, title).

**IRI / Link fields** — expect a full IRI (URL) as value. The field may validate that the entered value matches the expected IRI pattern. For example, the `owl:sameAs` field on a Person expects a value like `http://d-nb.info/gnd/118816X`. An error is shown if the value does not match the required pattern `^http(s)?://.+?/..+?`.

**Dropdown / enum fields** — a fixed list of allowed values to choose from. For example, the Gender field on a Person offers `female`, `male`, and `diverse`. Click the dropdown to see the options.

**Date fields** — some date fields offer a choice between a **single date** and a **date span** (a start and end date). A toggle or radio control lets you switch between the two modes. Dates are entered in ISO 8601 format (YYYY-MM-DD or YYYY).

**Linked entity reference fields** — allow you to select another entity from the dataset (e.g. a Birth Place linking to a Place entity). These are described in detail in the next section.

### Required Fields

Fields marked with an asterisk (`*`) or highlighted as required must be filled before the entity can be saved without validation errors. The form will show an inline error message if a required field is empty when you attempt to save.

---

## Linking Entities

Many fields reference other entities — for example, a Person has a Birth Place (linking to a Place entity), and a Work has a Composer (linking to a Person entity). These cross-references are what make the data model a true Linked Open Data graph.

### How Cross-References Work

Linked entity fields appear as **dropdown menus** populated from the dataset's published index files. When you click on a linked entity field, a searchable list of existing entities of the appropriate type appears, loaded from the `datasetBaseUrl` configured in `configuration/config.json`.

Select the entity you want to link. The IRI of the selected entity is written into the Turtle file as the value of the property.

Because the dropdown draws from the **published** index (not from local unsaved files), a newly created entity will only appear in linked entity dropdowns after it has been pushed to the remote repository and the CI/CD pipeline has regenerated the index files.

### Quick Add

If the entity you want to link does not exist yet, you do not need to leave the current form to create it. The **Quick Add** feature lets you create a new linked entity inline:

1. In the linked entity field, look for the **"Quick Add"** button (or the `+` button next to the field).
2. Click it. A dialog opens with a form for the linked entity type (e.g. a Place form if you are filling in a Birth Place).
3. Fill in the required fields and click **"Save"** in the Quick Add dialog.
4. The new entity is saved to the virtual filesystem and its IRI is automatically selected in the parent form's field.

The Quick Add entity is staged for git in the same way as a manually saved entity — it will appear in the Share Files section and must be pushed to become available in the published index.

---

## Previewing Linked Entities

While editing, you can inspect any linked entity without leaving the current form:

1. In the form, find a field that shows a linked entity value (displayed as a clickable link, rendered as `<a class="uuid">`).
2. Click the link.
3. A **floating, draggable side panel** opens on the right side of the screen. It shows a read-only rendering of the linked entity using a second `shacl-form` in `data-view` mode.
4. The panel can be dragged to any position and resized.
5. At the top of the side panel, an **"Edit"** button opens the linked entity in the main editor (replacing the currently open form). Use with care if you have unsaved changes in the current form — you will be warned before switching.

The preview panel is useful for quickly checking the details of a linked entity without navigating away from your work.

---

## Saving

Click the **"Save"** button (or the floppy-disk icon) to save the current entity.

### Unsaved Changes Indicator

As soon as you make any change to the form, the Save button changes appearance (turns blue and shows a dot indicator). This visual cue reminds you that the current state has not been written to the virtual filesystem yet.

### What Happens When You Save

1. The `shacl-form` serialises the current form state as RDF/Turtle (`shacl-form.serialize()`).
2. The Turtle content is written to the corresponding `.ttl` file in the virtual filesystem (IndexedDB, via LightningFS).
3. The file is staged for git (`git add` + `git updateIndex`).
4. The file appears in the **Share Files** section in the repository panel.

Saving does **not** push anything to the remote Git repository. The data is stored only in the browser's IndexedDB until you explicitly share (push) the file.

---

## Undoing Changes

Click the **"Undo"** button (arrow-counterclockwise icon) in the toolbar to revert the current form to the last saved state.

Undo only works for changes made since the last save. It restores the content of the `.ttl` file from the virtual filesystem and reloads the form. It does **not** unstage the file if it had already been staged — use the Unstage button in the Share Files section for that.

---

## Deleting an Entity

1. Open the entity you want to delete by clicking on its file in the repository tree.
2. Click the **"Remove Entity"** button (file-minus icon) in the editor toolbar.
3. A confirmation dialog appears asking you to confirm the deletion. Click **"Confirm"** (or the equivalent confirm button).
4. The file is removed from the repository tree and the editor is cleared.
5. The deleted file appears in the **Share Files** section with a `-deleted` suffix in its filename (e.g. `persons/2847362910.ttl-deleted`).
6. To push the deletion to the remote repository, check the box next to the deleted file entry and click **"Share files"**.

If you change your mind before pushing, use the **Unstage files** button in the Share Files section to restore the file from the last committed state.

Note: deleting an entity locally and pushing the deletion removes the file from the Git repository. Any other entities that linked to the deleted entity will retain the IRI reference but it will no longer resolve to a file.
