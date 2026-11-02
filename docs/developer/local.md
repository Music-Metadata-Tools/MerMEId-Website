# Local Setup

MerMEId can be used directly through the hosted web application. However, some users may prefer to run all components locally, for example when working in restricted network environments, for long-term preservation purposes, or to avoid dependencies on external services.

This page describes how to:

* Run the editor locally
* Use a local version of the form editor

## Requirements

Depending on the setup you choose, you may need:

| Feature                       | Required Software      |
| ----------------------------- | ---------------------- |
| Editor only                   | Git, Python 3          |
| Editor with local form editor | Git, Python 3, Node.js |
| Fully offline setup           | Git, Python 3, Node.js |
| Running the tests             | Git, Node.js (v22 or later) |

You can verify that the required software is installed:

```bash
git --version
python3 --version
node --version
npm --version
```

---

# Running the Editor Locally

The editor is a static web application and does not require a build step.

## Option 1: Using Python

Clone the repository and start a local web server:

```bash
git clone https://github.com/Music-Metadata-Tools/MerMEId-MeLODy.git
cd MerMEId-MeLODy
python3 -m http.server 8080
```

Open the editor in your browser:

```text
http://localhost:8080
```

## Option 2: Using VS Code Live Server

1. Clone the repository.
2. Open the project folder in VS Code.
3. Install the Live Server extension.
4. Click **Go Live** in the VS Code status bar.

The editor will open automatically in your browser.

---

# Using a Local Version of the Form Editor

By default, MerMEId loads the form editor from the published online version. If you would like to use a local copy instead, both repositories should be placed in the same parent directory.

Example:

```text
projects/
├── MerMEId-MeLODy/
└── mermeid-shacl-form/
```

## 1. Clone the Repositories

From the shared parent directory:

```bash
git clone https://github.com/Music-Metadata-Tools/MerMEId-MeLODy.git

git clone https://github.com/Music-Metadata-Tools/mermeid-shacl-form.git
cd mermeid-shacl-form
git checkout development
npm install
```

## 2. Build the Form Editor

```bash
npm run build-mermeid
```

This creates:

```text
dist/form-mermeid.js
```

## 3. Configure MerMEId

Open:

```text
MerMEId-MeLODy/index.html
```

Replace the published URL:

```html
<script type="module" src="https://music-metadata-tools.github.io/mermeid-shacl-form/dist/form-mermeid.js"></script>
```

with:

```html
<script type="module" src="/mermeid-shacl-form/dist/form-mermeid.js"></script>
```

and update any matching import statements accordingly.

## 4. Start a Local Server

From the shared parent directory:

```bash
python3 -m http.server 8080
```

## 5. Open the Editor

```text
http://localhost:8080/MerMEId-MeLODy
```

Whenever you rebuild the form editor with:

```bash
npm run build-mermeid
```

simply reload the browser. No server restart is required.

---

# Fully Offline Setup

Earlier versions of MerMEId MeLODy routed all Git operations through the public isomorphic-git CORS proxy (`https://cors.isomorphic-git.org`). **This is no longer necessary.** Cloning, pulling, pushing and listing branches now call the GitHub API (`api.github.com`) or the GitLab API (`https://<host>/api/v4` and `/api/graphql`) directly from the browser. Both APIs allow cross-origin requests, so no proxy service is involved.

This means:

* When you run the editor and the form editor locally (see above), the only external requests are those to your Git host's API and to the CDNs that serve the libraries (Shoelace, Lit, isomorphic-git, Oxigraph).
* Self-hosted GitLab instances work as long as their API allows cross-origin requests.
* If you want to avoid the CDNs as well, download the libraries referenced in the importmap and `<script>` tags of `index.html` and point the URLs to your local copies.

The constant `CORS_PROXY` in `modules/filesystem-manager/constants.js` still exists, but it is only used by legacy code paths that are no longer called. You do not need to change it.

## Working on a Local Folder

In Chromium-based browsers, the **Add repository** dialog can open a folder on your disk instead of loading a repository from a URL. The editor then reads and writes the files directly in that folder. If the folder has no `.git` directory, the editor runs `git init`. Sharing and synchronising still go through the GitHub/GitLab API, using the remote URL in `.git/config` (SSH remotes are converted to HTTPS for the API calls). You can also commit and push with your local Git client.

---

# Choosing the Right Setup

### Editor Only

Use this setup if you simply want to run MerMEId on your local machine.

Requirements:

* Git
* Python 3

### Editor + Local Form Editor

Use this setup if you want to use a locally hosted version of the form editor instead of the published online version.

Requirements:

* Git
* Python 3
* Node.js

### Fully Offline Setup

Use this setup if you want all editor components to run locally without depending on external services. Note that sharing and synchronising always require network access to your Git host's API.

Requirements:

* Git
* Python 3
* Node.js
