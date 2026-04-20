# Contributing

We warmly welcome contributions of all kinds — you do not need to be a developer to help.

---

## Ways to Contribute

- **Use the tool and give feedback** — try the editor, catalogue some music, and tell us what works and what doesn't
- **Report bugs** — if something breaks or behaves unexpectedly, please open an issue
- **Suggest improvements** — have an idea for a new feature or a cleaner workflow? Open a discussion or issue
- **Improve documentation** — spotted something unclear or missing in these docs? Pull requests to the website repository are very welcome
- **Contribute code** — fix a bug, add a feature, or write a test

No contribution is too small. Even a typo fix or a clarifying sentence in the documentation helps.

---

## Bug Reports and Feature Requests

Please open a [GitHub issue](https://github.com/Music-Metadata-Tools/MerMEId-MeLODy/issues/new) for bugs or feature requests. Check the existing issues first to avoid duplicates. Issues labelled [good first issue](https://github.com/Music-Metadata-Tools/MerMEId-MeLODy/issues?q=state%3Aopen+label%3A%22good+first+issue%22) are a good starting point — they are well-scoped and don't require deep familiarity with the codebase.

---

## Making Code Changes

The editor is a static HTML/JS application — no build step is needed for most work. To run it locally:

```bash
git clone https://github.com/Music-Metadata-Tools/MerMEId-MeLODy.git
cd MerMEId-MeLODy
python3 -m http.server 8080
```

Then open `http://localhost:8080`.

**If you have write access** to the repository, create a branch directly (name it after the issue, e.g. `issue-33`).

**If you are an external contributor**, fork the repository first, then create a branch in your fork. Open a Pull Request against `main` of the original repository when ready.

Commit messages should be short and written in the imperative: *"Add person converter"*, not *"Added"* or *"Adding"*.

---

## Running the Tests

If you are working on core functionality (filesystem management, git operations), run the test suite:

```bash
npm install
npm test
```

Requires [Node.js](https://nodejs.org) v22 or later.

For integration tests (real clone/pull/push operations), you need a test repository and a GitHub personal access token. Create a repository from the [MerMEId MeLODy Template](https://github.com/Music-Metadata-Tools/MerMEId-MeLODy-Template) and add a `.env` file in the project root:

```
GIT_TOKEN=<your GitHub personal access token>
GIT_USERNAME=<your GitHub username>
TEST_REPO_URL=<URL of your test repository>
```

Integration tests are automatically skipped if these variables are not set.

---

## Pull Request Review

PRs are merged into `main` by a maintainer once:

- Unit tests pass (run automatically for all PRs)
- At least one maintainer has approved
- There are no requested changes or conflicts

We use **squash and merge** to keep the commit history clean.

---

## Code of Conduct

This project follows a [Contributor Code of Conduct](https://github.com/Music-Metadata-Tools/MerMEId-MeLODy/blob/main/CODE_OF_CONDUCT.md). By participating, you agree to abide by its terms. We are committed to a welcoming, inclusive environment for everyone regardless of background or experience level.
