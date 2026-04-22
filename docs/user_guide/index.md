# Getting Started

This guide covers everything you need to get from zero to your first saved entity in MerMEId MeLODy.

---

## Prerequisites

### Browser Recommendation

MerMEId MeLODy runs entirely in the browser. **Chromium-based browsers (Chrome or Edge) are recommended** for the best experience. Firefox works for most operations, but the File System Access API (used for local filesystem mode) is not supported in Firefox.

### Git Host Account

You need an account on a Git hosting platform to create data repositories — [GitHub](https://github.com) or [GitLab](https://gitlab.com) are both supported. The editor communicates with any HTTPS-accessible Git host. To edit data in the repository you don't need a Git Aaccount, only the data owner needs one.

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
4. Set Repository acces to **"Only select repositories"** and choose project-specific repository
5. Give it a descriptive name (e.g. `mermeid-melody`).
6. Set an expiration date.
7. Select the **`repo`** scope (full control of private repositories).
8. Click **"Generate token"** and copy the result immediately — it is shown only once.

**GitLab — creating a token user-based: (GitLab Account required)**

1. Go to your user **Settings → Access Tokens**.
2. Click **"Add new token"**.
3. Select the scope **`write_repository`**.
4. Set an expiry date if required by your organisation.
5. Click **"Create personal access token"** and copy the result immediately — it is shown only once.

**GitLab — creating a token project-specific: (without GitLab account)**

1. Go to your repositories **Settings → Access Tokens**.
2. Click **"Add new token"**.
3. Select the scope **`write_repository`**.
4. Set an expiry date if required by your organisation.
5. Click **"Create personal access token"** and copy the result immediately — it is shown only once.

Store the token somewhere safe (e.g. a password manager).

---

## Creating a Data Repository

All metadata lives in a Git repository with a predefined folder structure. The quickest way to create one is to use the official template:

1. Go to the [MerMEId MeLODy Template](https://github.com/Music-Metadata-Tools/MerMEId-MeLODy-Template) on GitHub or to the [GitLab MerMEId MeLODy Template](https://gitlab.rlp.net/adwmainz/nfdi4culture/cdmd/project_templates/mermeid-template).
2. Click **"Use this template"** → **"Create a new repository"**.
3. Choose an owner and give the repository a meaningful name (e.g. `mozart-catalogue`).
4. Set the visibility to **Public**. On GitHub, a public repository is required for the search to work (GitHub Pages is only free for public repositories). On GitLab the repository can be private, but the Pages need to be configured as Public for everyone.
5. Click **"Create repository"**.
6. Activate Pages so the search index gets published:
    - **GitHub:** go to **Settings → Pages → Source** and select **"GitHub Actions"**.
    - **GitLab:** go to **Deploy → Pages -> Domains and Settings**. Uncheck 'Use unique domain'. For private repositories: go to **Settings → General → Visibility** and set the Pages visibility to 'Everyone'.

The template includes the correct folder structure, a CI/CD pipeline that publishes the search index after every push and sparql queries for each dataset used as indexes. Once that first pipeline run completes, you need to tell the editor where to find the index — open `configuration/config.json` in your repository and replace the placeholder in `datasetBaseUrl` with your Pages URL:

- **GitHub:** `https://<owner>.github.io/<repo-name>/datasets/`
- **GitLab:** `https://<namespace>.gitlab.io/<repo-name>/datasets/`

For example, if your GitHub username is `jsmith` and your repository is named `mozart-catalogue`, the value would be `https://jsmith.github.io/mozart-catalogue/datasets/`. You can find your exact Pages URL in the repository settings after the first pipeline run. For more details see [Repository Configuration](../configuration/repo_config.md).

The template contains a small amount of sample data you can delete once you are comfortable with the editor.

---

## Adding a Repository to the Editor

Once your data repository exists and you have a token, connect the editor to it:

1. Open [MerMEId MeLODy](https://music-metadata-tools.github.io/MerMEId-MeLODy/) in your browser.
2. In the left panel, click the **"Add repository"** button (folder-plus icon, top-left toolbar).
3. Enter the **repository folder name** as display name — this is only used inside the editor (e.g. `mozart-catalogue`). Spaces are not allowed.
4. Enter the **repository URL** — the HTTPS clone URL from GitHub or GitLab. It must start with `https://` and end with `.git` (e.g. `https://github.com/yourname/mozart-catalogue.git`).
5. Enter your **username** (your GitHub or GitLab username, or choose one, if you don't have an account).
6. Enter your **personal access token**.
7. Click **"Next"**, select the branch you want to work on (usually `main`), and click **"Clone"**.

The editor will clone the repository. Once finished, the repository appears in the left panel. Click on its name to select it — the name is highlighted when the repository is active.

---

## Your First Entity

Once you have a repository selected:

1. Click the **"New"** button in the toolbar to open the entity type dialog.
2. Select an entity type (start with **Person** or **Place** — they have fewer required fields).
3. A blank form opens in the main area. Fill in the required fields (marked with an asterisk).
4. Click **"Save"** — the button turns blue with a dot indicator when there are unsaved changes.
5. In the bottom left corner you see **Share files**. Click on it, check all files you want to share and click **Share files**.

The file is now saved to the Git repository, see [Git Workflow](user_guide/git_workflow.md) for further information.

For a full description of all entity types and their fields, see [Working with Entities](user_guide/entities.md) and [Data Model](data_model.md).

---

## Running the Editor Locally

The editor is a static HTML/JavaScript application — no build step is required. To run a local copy:

**Option 1: Python**

Clone the [MerMEId MeLODy repository](https://github.com/Music-Metadata-Tools/MerMEId-MeLODy) and serve it with Python's built-in server:

```bash
git clone https://github.com/Music-Metadata-Tools/MerMEId-MeLODy.git
cd MerMEId-MeLODy
python3 -m http.server 8080
```

Then open `http://localhost:8080` in your browser.

**Option 2: VS Code Live Server**

Open the cloned folder in [VS Code](https://code.visualstudio.com/) and install the [Live Server](https://marketplace.visualstudio.com/items?itemName=ritwickdey.LiveServer) extension. Click **"Go Live"** in the status bar.

Note that all Git operations (clone, push, pull) still go through the CORS proxy ([cors.isomorphic-git.org](https://cors.isomorphic-git.org)) even when running locally, because browsers block direct cross-origin Git HTTP requests. That means that the local use ist still not completely offline ready.
