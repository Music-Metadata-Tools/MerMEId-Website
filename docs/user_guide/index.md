# Getting Started

This guide covers everything you need to get from zero to your first saved entity in MerMEId MeLODy.

---

## Prerequisites

### Browser Recommendation

MerMEId MeLODy runs entirely in the browser. **Chromium-based browsers (Chrome or Edge) are recommended** for the best experience. Firefox works for most operations, but the File System Access API (used for local filesystem mode) is not supported in Firefox.

### Git Host Account

You need an account on a Git hosting platform to create data repositories — [GitHub](https://github.com) or [GitLab](https://gitlab.com) (gitlab.com or a self-hosted GitLab instance such as GitLab RLP) are supported. The editor communicates directly with the GitHub or GitLab API; other Git hosts are not supported right now. To edit data in the repository you don't need a Git account, only the data owner needs one.

### Personal Access Token

The editor authenticates to your Git host using either a project-specific or a user-based **personal access token** (PAT). You need to create one or ask an admin to create one before adding a repository.

**GitHub — creating a token user-based: (GitHub account required)**

1. Go to **Settings → Developer settings → Personal access tokens → Tokens (classic)**.
2. Click **"Generate new token (classic)"**.
3. Give it a descriptive name (e.g. `mermeid-melody`).
4. Set an expiration date.
5. Select the **`repo`** scope (full control of private repositories).
6. Click **"Generate token"** and copy the result immediately — it is shown only once.

**GitHub — creating a token project-specific: (for users without GitHub account, Organization required)**

1. Go to **Settings → Developer settings → Personal access tokens → Fine-grained tokens**.
2. Click **"Generate new token"**.
3. Choose organization as resource owner
4. Set Repository access to **"Only select repositories"** and choose project-specific repository
5. Give it a descriptive name (e.g. `mermeid-melody`).
6. Set an expiration date.
7. Under **Repository permissions**, set **Contents** to **"Read and write"** (Metadata: Read-only is added automatically).
8. Click **"Generate token"** and copy the result immediately — it is shown only once.

**GitLab — creating a token user-based: (GitLab Account required)**

1. Go to your user **Settings → Access Tokens**.
2. Click **"Add new token"**.
3. Select the scope **`api`**.
4. Set an expiry date if required by your organisation.
5. Click **"Create personal access token"** and copy the result immediately — it is shown only once.

**GitLab — creating a token project-specific: (without GitLab account)**

1. Go to your repositories **Settings → Access Tokens**.
2. Click **"Add new token"**.
3. Choose the role **Maintainer** (or higher) and select the scope **`api`**.
4. Set an expiry date if required by your organisation.
5. Click **"Create project access token"** and copy the result immediately — it is shown only once.

Store the token somewhere safe (e.g. a password manager).

---

## Creating a Data Repository

All metadata lives in a Git repository with a predefined folder structure. The quickest way to create one is to use the official template:

1. Go to the [MerMEId MeLODy Template](https://github.com/Music-Metadata-Tools/MerMEId-MeLODy-Template) on GitHub or to the [GitLab MerMEId MeLODy Template](https://gitlab.rlp.net/adwmainz/nfdi4culture/cdmd/project_templates/mermeid-template).
2. Click **"Use this template"** → **"Create a new repository"** (on GitLab: create a new project from the template or fork it).
3. Choose an owner and give the repository a meaningful name (e.g. `mozart-catalogue`).
4. Choose the visibility. The repository can be **public or private**.
5. Click **"Create repository"**.
6. Make sure the CI/CD pipeline is allowed to push the generated indexes back to the repository:
    - **GitHub:** nothing to do — the workflow requests write access itself. If your organisation restricts workflow permissions, allow GitHub Actions to write to the repository (**Settings → Actions → General**).
    - **GitLab:** go to **Settings → CI/CD → Job token permissions** and allow Git push requests with the CI/CD job token (or adapt the `push_indexes` job to use a project access token).

The template includes the correct folder structure, sample data, the configuration file `configuration/config.json`, and a CI/CD pipeline that regenerates the search indexes in the `indexes/` folder after every push to `main`. The SPARQL queries used to build the indexes are fetched from the editor repository during the pipeline run, so they always match the current editor version.

The editor itself also keeps the files in `indexes/` up to date whenever you save, delete, or synchronise entities, so you can start working right away. The pipeline makes sure the indexes stay consistent when files are changed outside the editor.

The file `configuration/config.json` contains the IRI prefix domain for new entities (`projectDomain`). For more details see [Repository Configuration](../developer/repo_config.md).

The template contains a small amount of sample data you can delete once you are comfortable with the editor.

---

## Adding a Repository to the Editor

Once your data repository exists and you have a token, connect the editor to it:

1. Open [MerMEId MeLODy](https://music-metadata-tools.github.io/MerMEId-MeLODy/) in your browser.
2. In the left panel, click the **"Add repository"** button (folder-plus icon, top-left toolbar).
3. Enter the **repository folder name** as display name — this is only used inside the editor (e.g. `mozart-catalogue`). It must start with a letter or digit and may only contain letters, digits, `_`, `-` and `.` (no spaces).
4. Enter the **repository URL** — e.g. `https://github.com/yourname/mozart-catalogue`. The URL may end with `.git` or not; copying the address from the browser's address bar works as well. SSH URLs (`git@github.com:...`) are converted automatically.
5. Enter your **username** (your GitHub or GitLab username, or choose one, if you don't have an account).
6. Enter your **personal access token**.
7. Click **"Next"** and select the branch you want to work on. If the repository has only one branch, or a branch named `main`, it is preselected.
8. Decide whether to **"Use lazy loading"** (enabled by default). With lazy loading, only the list of files is downloaded and each file is fetched when you open it for the first time — recommended for large repositories. Turn it off to download all files at once.
9. Click **"Clone"**. A progress indicator is shown while the files are loaded.

The folder name, URL and username of the last added repository are prefilled the next time you open the dialog.

Once finished, the repository appears in the left panel and is selected automatically. Click on its name to select another repository — the name is highlighted when the repository is active.

When a repository is selected for the first time, the editor creates the catalog files `dataCatalogs/catalog.ttl`, `dataCollections/works.ttl` and `dataCollections/manifestations.ttl` if they are missing. They appear in the **Share files** section and can be shared like any other file.

### Adding a Local Folder

Instead of a remote URL, you can also work on a folder on your computer (Chromium-based browsers only). On the first page of the dialog, click the folder-plus button below **"Or add a local folder instead"** and choose the folder in the browser's folder picker. If the folder is not yet a Git repository, it is initialised. Optionally enter username and token so you can synchronise and share with the remote configured in the folder.

After a browser restart, the editor asks you again for permission to access the folder when you click the repository.

---

## Your First Entity

Once you have a repository selected:

1. Click the **"New"** button in the editor toolbar to open the entity type dialog.
2. Select an entity type (start with **Person** or **Place** — they have fewer required fields).
3. A blank form opens in the main area. Fill in the required fields (marked with an asterisk).
4. Click **"Save"** — the button becomes active and shows a dot indicator when there are unsaved changes. Saving is only possible when all required fields are filled and valid.
5. In the bottom left corner you see **Share files**. Check all files you want to share, optionally enter a commit message, and click the **Share files** button.

The file is now saved to the Git repository, see [Git Workflow](git_workflow.md) for further information.

For a full description of all entity types and their fields, see [Working with Entities](entities.md) and [Data Model](../data_model/index.md).

---
