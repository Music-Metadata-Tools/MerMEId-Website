# Git Workflow

MerMEId MeLODy uses standard Git operations to store and synchronise your data. This page explains how the Git integration works and how to manage your repositories effectively.

---

## How Storage Works

MerMEId MeLODy stores all data in a **virtual filesystem** running inside your browser. This virtual filesystem is persisted in the browser's **IndexedDB** — a built-in browser database that survives page refreshes and browser restarts.

The editor provides a full Git implementation. Every Git operation (clone, pull, commit, push) is executed entirely in the browser.

### Credentials Storage

Your repository folder name, repository url and git username are stored in the browser's local storage.

These values are written to the local storage during the clone operation and read back before adding a new repository. They persist across browser sessions (as long as you do not clear the browser storage). They are **not** sent anywhere except to your Git host during authenticated requests.

### Why a CORS Proxy?

Browsers block cross-origin HTTP requests by default. Git hosts (GitHub, GitLab) do not add the CORS headers that browsers require for cross-origin requests from JavaScript. To work around this, all Git HTTP traffic is routed through a **CORS proxy** at `https://cors.isomorphic-git.org`. This proxy simply forwards the request to the target host and adds the required CORS headers. Your credentials are sent directly to the Git host; the proxy only handles the transport.

---

## Adding a Repository

Adding a repository in MerMEId MeLODy is equivalent to `git clone`. Here is what happens step by step:

1. Open the **"Add repository"** dialog (folder-plus icon in the repository panel toolbar).
2. Enter the display name, repository URL, username, and personal access token.
3. Click **"Next"** .
4. Select a branch (usually `main`) and click **"Clone"**.
5. The editor calls a shallow clone — only the latest commit on the selected branch is downloaded. This keeps the initial clone fast even for large repositories.
6. Credentials are stored in the browser's local storage.
7. The repository appears in the left panel. Click its name to select it.

**Important:** you must click the repository name to select (highlight) it before you can create new entities or open files. A repository that has been cloned but not selected is shown in the tree but is not active.

---

## Selecting a Repository

Click the **repository name** (the top-level node in the tree) to select it. The name is highlighted when the repository is active. Only one repository can be active at a time. All entity creation, editing, and saving operations apply to the currently selected repository.

---

## Sharing Changes

"Sharing" files means committing and pushing them to the remote Git repository. This is equivalent to `git commit` + `git push`.

### Staging Queue

Every time you save an entity, the file is added to the **Share Files** staging queue at the bottom of the repository panel. The queue shows all files that have been modified locally but not yet pushed.

### Selective Commit

You do not have to push all staged files at once. The staging queue allows **selective commit**:

1. In the Share Files section, check the boxes next to the files you want to include in this push.
2. Click the **"Share files"** button (cloud-upload icon).
3. The editor will auto-pull first (to check for remote changes — see Conflict Detection below).
4. A commit is created containing only the checked files, and pushed to the remote.

Files that were not checked remain in the staging queue for a future push.

### Auto-Pull Before Push

Before committing and pushing, MerMEId MeLODy automatically checks whether the remote has any new commits. If the remote is ahead:

- If the remote changes do **not** overlap with your staged files, the pull proceeds silently and the push follows.
- If the remote changes **do** overlap with one or more of your staged files (a conflict), the push is blocked and you are warned. You have to unstage the conflict file, to synchronize properly.

---

## Synchronising

"Synchronising" means pulling the latest changes from the remote repository. This is equivalent to `git pull`.

### How to Synchronise

Click the **"Synchronise"** button (arrow-clockwise icon) in the repository panel toolbar.

### Conflict Detection

Before pulling, the editor performs a **conflict check**:

1. The editor fetches the remote HEAD (the latest commit hash on the remote branch).
2. It finds the common ancestor commit between the local and remote branches.
3. It compares the set of files changed on the remote since the common ancestor against your locally staged files.
4. If there is overlap (the same file has been modified both remotely and locally), the pull is **blocked** and a warning is shown listing the conflicting files.

If conflicts are detected, you must first unstage your local changes for those files (see below), then synchronise again.

If no conflicts are found, the pull proceeds and the local working directory is updated to match the remote.

### Unsaved Changes Warning

If you have unsaved changes in the editor (the Save button is blue), you will be warned before synchronising. Save or discard the current changes first.

---

## Unstaging Files

To discard local saved changes for a specific file and restore it to the last committed state:

1. In the Share Files section, select the file you want to unstage.
2. Click the **"Unstage files"** button (arrow-counterclockwise icon).
3. The file is removed from the Share Files queue and the working directory copy is restored to the last committed state.

This is equivalent to `git restore --staged <file> && git restore <file>` in standard Git.

---

## Removing a Repository

Click the **"Remove repository"** button (folder-minus icon) in the repository panel toolbar. After confirmation, the repository is removed from the browser's virtual filesystem (IndexedDB).

**Important:** removing a repository from the editor does **not** delete anything on the remote. The Git repository on GitHub or GitLab is completely unaffected. You can re-add it at any time using the Add Repository dialog.

---

## Renaming a Repository

Click the **"Rename repository"** button (folder icon) in the repository panel toolbar. Enter a new display name. This only changes the label shown in the editor — it does not rename the remote repository or affect anything in the Git history.

---

## Git Command Reference

For users familiar with Git, the following table maps MerMEId MeLODy operations to their standard Git equivalents:

| MerMEId MeLODy operation | Git equivalent | Notes |
| :--- | :--- | :--- |
| Add repository | `git clone --depth 1 --single-branch` | Shallow clone, one branch only |
| Synchronise | `git pull` | Preceded by conflict check |
| Save | write file + `git add` + `git updateIndex` | Local only, no network |
| Share files | `git commit` + `git push` | Selective — only checked files |
| Unstage files | `git resetIndex` + `git checkout` | Restores from HEAD |
| Remove repository | `rm -rf <repo>` (local only) | Remote is unaffected |
