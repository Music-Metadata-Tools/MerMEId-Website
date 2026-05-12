# Services

The service layer provides a shared, singleton access point to the virtual filesystem and all Git operations. Components never instantiate their own filesystem directly — they always go through the service.

---



### Credentials

Username and personal access token are stored in the git config of the cloned repository (`user.name` and `user.pat`), persisted inside the virtual filesystem in `IndexedDB`. They are read back via `git.getConfigAll()` before each push or pull operation. Credentials are never sent anywhere other than the Git server — the CORS proxy only forwards requests and adds CORS headers.

---

### CORS Proxy

Because browsers block cross-origin requests to Git servers, all Git HTTP traffic is routed through a CORS proxy. The default proxy URL is configured in `modules/filesystem-manager/constants.js`:

```js
export const CORS_PROXY = "https://cors.isomorphic-git.org";
```

This is the standard proxy maintained by the isomorphic-git project. It simply forwards Git HTTP requests and adds the necessary `Access-Control-*` response headers. No data is stored or logged by the proxy.

If you need to use a different proxy (e.g. a self-hosted instance), update the `CORS_PROXY` constant.

---

### Local Filesystem Mode

`VirtualFilesystem` also supports direct access to the local filesystem via the [File System Access API](https://developer.chrome.com/docs/capabilities/web-apis/file-system-access). When a directory handle is registered via `add_local_repository()`, all read/write operations for that repository bypass isomorphic-git and operate directly on the local files.

**Use case:** local workflows for larger repositories, git actions need to be executed locally (not within the browser).

**Browser support:** best supported in Chromium-based browsers (Chrome, Edge). Firefox has limited support; Safari does not support the writable variant of the API.

---

### Conflict Detection (`canPullSafely`)

Before any push or pull operation, `canPullSafely(repo, changedFiles)` is called with the list of locally staged files. It:

1. Fetches the remote `HEAD` reference
2. Walks the commit graph to find the most recent common ancestor commit between the local branch and the remote
3. Lists all files changed in the remote since that common ancestor
4. Checks for overlap with the locally staged files

If any file appears in both sets, the operation is blocked and the user receives a warning listing the conflicting files. This prevents silent data loss from a non-fast-forward situation.
