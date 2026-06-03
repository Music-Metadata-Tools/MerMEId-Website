# Local Setup

MerMEId can be used directly through the hosted web application. However, some users may prefer to run all components locally, for example when working in restricted network environments, for long-term preservation purposes, or to avoid dependencies on external services.

This page describes how to:

* Run the editor locally
* Use a local version of the form editor
* Run Git operations without relying on the public Git proxy service

## Requirements

Depending on the setup you choose, you may need:

| Feature                       | Required Software      |
| ----------------------------- | ---------------------- |
| Editor only                   | Git, Python 3          |
| Editor with local form editor | Git, Python 3, Node.js |
| Fully offline setup           | Git, Python 3, Node.js |

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

By default, Git operations performed by MerMEId are routed through the public isomorphic-git proxy service:

```text
https://cors.isomorphic-git.org
```

This allows Git operations such as cloning, pulling, and pushing repositories directly from the browser.

If you would prefer not to rely on the public service, you can run your own local proxy instead.


## Option 1: Quick Start

The easiest option is to start the proxy directly with npm:

```bash
npm install -g @isomorphic-git/cors-proxy
cors-proxy run
```

The proxy will be available at:

```text
http://localhost:9999
```

## Option 2: Clone the Proxy Locally

If you prefer to keep a local copy of the proxy source code:

```bash
git clone https://github.com/isomorphic-git/cors-proxy.git
cd cors-proxy
npm install
npx cors-proxy start
```

The proxy will be available at:

```text
http://localhost:9999
```

## Configure MerMEId to Use the Local Proxy

Open:

```text
modules/filesystem-manager/constants.js
```

Replace:

```javascript
export const CORS_PROXY = 'https://cors.isomorphic-git.org'
```

with:

```javascript
export const CORS_PROXY = 'http://localhost:9999'
```

Restart the editor if it is already running.

All Git operations will now use your local proxy service instead of the public one.

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

Use this setup if you want all editor components and Git operations to run locally without depending on external services.

Requirements:

* Git
* Python 3
* Node.js
