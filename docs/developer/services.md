# Services

The service layer provides shared, singleton access points to the virtual filesystem, all Git operations and the search indexes. Components never instantiate their own filesystem directly — they always go through the service.

---

### Credentials

Username and personal access token are stored in the git config of the repository (`user.name` and `user.pat`). For repositories added by URL, this config lives in the browser's Origin Private File System (OPFS). For **local folders**, it is written into the folder's own `.git/config` on disk — the token is then stored there in plain text.

The credentials are read back before each push or pull operation and can be changed at any time through the repository settings dialog. The token is only sent to the GitHub or GitLab API (as `Authorization: Bearer` or `PRIVATE-TOKEN` header).

The add-repository dialog additionally remembers the last folder name, URL and username — but not the token — in `localStorage` (key `lastGitCredentials`, see `credentials-helper.js`).

---

### Git Host API

All network operations go directly to the APIs of the Git host, implemented in `modules/filesystem-manager/api-provider.js`:

- `parseRepoUrl()` normalises HTTPS, SSH and GitLab web URLs. Hosts containing `github` are treated as GitHub, all other hosts as GitLab (API v4). Other Git hosts are not supported.
- `createProvider()` returns a GitHub or GitLab provider for listing branches, fetching the file tree, downloading files and fetching single remote files.
- `pushViaApi()` creates one commit with all selected files — via the GraphQL mutation `createCommitOnBranch` on GitHub or the Commits API (`POST /projects/:id/repository/commits`) on GitLab.
- Requests that are rate-limited (HTTP 429) are retried automatically, respecting `Retry-After`.

The constant `CORS_PROXY` in `modules/filesystem-manager/constants.js` is still defined, but only used by legacy code paths that are no longer called.

---

### Lazy Loading

When a repository is added with lazy loading (default), only the remote file tree is fetched and written to `.remote-tree.json`. Each file is downloaded the first time it is read. Without lazy loading, all files are downloaded at once. The setting is stored per repository in the git config (`lazy.enabled`).

---

### Local Filesystem Mode

`VirtualFilesystem` also supports direct access to the local filesystem via the [File System Access API](https://developer.chrome.com/docs/capabilities/web-apis/file-system-access). When a directory handle is registered via `add_local_repository()`, the picked folder is used as a normal Git working directory (`fsa-directory-filesystem.js`); if it contains no `.git`, `git init` is run. The folder handles are kept in IndexedDB (`local-repository-store.js`); after a browser restart, permission is requested again via `ensure_local_repository_access()`.

Sharing and synchronising work exactly as for repositories added by URL (through the GitHub/GitLab API, using the remote from `.git/config`). The bookkeeping files `.dirty.json` and `.snapshot.json` are kept in OPFS under the repository name.

**Use case:** local workflows for larger repositories, or when you also want to use your local Git client.

**Browser support:** best supported in Chromium-based browsers (Chrome, Edge). Firefox and Safari do not support the directory picker of the API.

---

### Conflict Detection (`checkPushConflicts`)

Before sharing, `checkPushConflicts()` is called with the list of selected files. For each file that has a synchronised baseline in `.snapshot.json`, it:

1. Fetches the current version of the file from the remote via the API
2. Compares it with the baseline from the last synchronisation
3. Reports the file as a conflict if the remote version has changed or the file has been deleted remotely

Conflicting files are shown in the `adwlm-push-conflict-dialog`, which renders the local and the remote version side by side with `shacl-form`. For each file the user chooses to keep the local version, take the remote version (`resolveConflictWithRemote()`), or skip the file for this push. New files that do not exist remotely never cause a conflict.
