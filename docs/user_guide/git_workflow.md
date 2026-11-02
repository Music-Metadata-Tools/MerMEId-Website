# Git Workflow

MerMEId MeLODy uses a Git repository on GitHub or GitLab to store and synchronise your data. This page explains how the Git integration works and how to manage your repositories effectively.

---

## How Storage Works

MerMEId MeLODy stores all data in a **virtual filesystem** running inside your browser. This virtual filesystem is persisted in the browser's **Origin Private File System (OPFS)** — a private storage area of the browser that survives page refreshes and browser restarts. Alternatively, you can work directly on a [local folder](index.md#adding-a-local-folder) on your computer.

The editor communicates directly with the **GitHub or GitLab API** to download files and to create commits. Every operation (adding, synchronising, sharing) is executed entirely in the browser.

### Credentials Storage

Your repository folder name, repository url and git username are stored in the browser's local storage.

These values are written to the local storage during the clone operation and read back before adding a new repository. They persist across browser sessions (as long as you do not clear the browser storage). They are **not** sent anywhere except to your Git host during authenticated requests.

---

## Adding a Repository

Adding a repository in MerMEId MeLODy is equivalent to `git clone`. Here is what happens step by step:

1. Open the **"Add repository"** dialog (folder-plus icon in the repository panel toolbar).
2. Enter the display name, repository URL, and — if required — username and personal access token.
3. Click **"Next"**.
4. Select a branch (usually `main`), decide whether to use **lazy loading**, and click **"Clone"**.
5. The editor downloads the files via the API:
    - With **lazy loading** (default), only the list of files is downloaded. Each file is fetched the first time you open it. This keeps adding a repository fast even for large repositories.
    - Without lazy loading, all files are downloaded at once.
6. Credentials are stored in the browser's local storage.
7. The repository appears in the left panel. Click its name to select it.

---

## Selecting a Repository

Click the **repository name** (the top-level node in the tree) to select it. The name is highlighted when the repository is active. Only one repository can be active at a time. All entity creation, editing, and saving operations apply to the currently selected repository.

When the page is loaded, the first repository is selected automatically for the search.

---

## Sharing Changes

"Sharing" files means committing and pushing them to the remote Git repository. This is equivalent to `git commit` + `git push`.

### Staging Queue

Every time you save an entity, the file is added to the **Share Files** staging queue at the bottom of the repository panel. The queue shows all files that have been modified locally but not yet pushed.

### Selective Commit

You do not have to push all staged files at once. The staging queue allows **selective commit**:

1. In the Share Files section, check the boxes next to the files you want to include in this push (or use **Select all**).
2. Optionally enter a commit message. Otherwise the current date and your username are used.
3. Click the **"Share files"** button (cloud-upload icon).
4. The editor checks whether the selected files have been changed online in the meantime (see Conflict Check below).
5. A commit is created containing only the checked files, and pushed to the remote.

Files that were not checked remain in the staging queue for a future push.

### Conflict Check Before Sharing

Before committing and pushing, MerMEId MeLODy compares each selected file with its current version in the remote repository:

- If none of the selected files have been changed online since your last synchronisation, the files are shared immediately.
- If one or more files **have** been changed online (or deleted online), the **Sharing conflicts** dialog opens.

The Sharing conflicts dialog lists every affected file and shows your version ("Your version (not yet shared)") next to the current online version, with the differences highlighted. For each file you choose one option:

- **Keep my version** (or **Delete file**, if you deleted it locally) — your version is shared and overwrites the online version.
- **Use online version** — your local changes are discarded and the online version is loaded. Nothing is shared for this file.
- **Decide later** — the file is skipped for now and stays in the staging queue.

If several files are affected, the buttons **All: Keep my version** and **All: Use online version** apply the choice to all of them. Once a decision has been made for every file, click **Apply and share**. **Cancel** aborts sharing.

New files that do not exist online yet never cause a conflict.

---

## Synchronising

"Synchronising" means downloading the latest changes from the remote repository. This is similar to `git pull`.

### How to Synchronise

Click the **"Synchronize repository"** button (cloud-download icon) in the repository panel toolbar. A personal access token is required for synchronising; if the repository has none, add one in the **Repository settings**.

### What Happens During Synchronisation

1. The editor downloads all files that have been changed online since the last synchronisation and removes files that have been deleted online.
2. Your staged (not yet shared) files are **kept** and re-applied afterwards, so you don't lose any local work.
3. The index files are updated for the changed files and the repository tree is refreshed.

Synchronising does not check for conflicts. If a staged file has also been changed online, the differences are resolved the next time you share the file (see [Conflict Check Before Sharing](#conflict-check-before-sharing)).

### Unsaved Changes Warning

If you have unsaved changes in the editor (the Save button is blue), you will be warned before synchronising. Save or discard the current changes first.

---

## Unstaging Files

To discard local saved changes for a specific file and restore it to the last synchronised state:

1. In the Share Files section, check the box next to the file you want to unstage.
2. Click the **"Unstage files"** button (arrow-counterclockwise icon).
3. The file is removed from the Share Files queue and the local copy is restored to the version from your last synchronisation. A new file that was never shared is removed completely. The index and the open form are updated accordingly.

This is equivalent to `git restore --staged <file> && git restore <file>` in standard Git.

---

## Removing a Repository

Click the **"Remove repository"** button (folder-minus icon) in the repository panel toolbar. The repository is removed immediately from the browser's storage — there is no confirmation dialog, so make sure you have shared all changes you want to keep. For local folders, only the reference in the editor is removed; the files on your disk are not touched.

**Important:** removing a repository from the editor does **not** delete anything on the remote. The Git repository on GitHub or GitLab is completely unaffected. You can re-add it at any time using the Add Repository dialog.

---

## Renaming a Repository

Click the **"Rename repository"** button (folder icon) in the repository panel toolbar. Enter a new display name. This only changes the label shown in the editor — it does not rename the remote repository or affect anything in the Git history.

---

## Git Command Reference

For users familiar with Git, the following table maps MerMEId MeLODy operations to their standard Git equivalents:

| MerMEId MeLODy operation | Git equivalent | Notes |
| :--- | :--- | :--- |
| Add repository | `git clone --depth 1 --single-branch` | Files are downloaded via the API (lazy or all at once); no Git history |
| Synchronise | `git pull` | Downloads changed files via the API; staged local files are kept |
| Save | write file + `git add` | Local only, no network; also updates the index files |
| Share files | `git commit` + `git push` | Selective — only checked files; conflict check per file; one commit via the GitHub/GitLab API |
| Unstage files | `git restore --staged` + `git restore` | Restores the last synchronised version |
| Remove repository | `rm -rf <repo>` (local only) | Remote is unaffected |
