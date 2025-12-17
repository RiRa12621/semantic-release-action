# Simple Semantic Release Action

A lightweight, "batteries-included" GitHub Action that handles Semantic Versioning, Git tagging, GitHub Releases, and artifact uploading in one go.

It reads your commit history (Conventional Commits), calculates the next version (Major, Minor, or Patch), and automates the release process.

## 🚀 Features

* **Automatic Versioning:** Calculates the next Semantic Version based on commit messages (`fix:`, `feat:`, `BREAKING CHANGE:`).
* **Git Tagging:** Pushes the new tag to your repo automatically.
* **GitHub Releases:** Generates a release with a changelog derived from commit messages.
* **Artifact Uploading:** (Optional) Upload binaries (DMG, EXE, DEB, etc.) to the release.
* **Dry Run Mode:** Calculate the version *before* building, so you can inject the correct version number into your build artifacts.

## 📦 Usage

### Basic Usage
If you just want to tag and release when you push to main:

```yaml
steps:
  - uses: actions/checkout@v4
    with:
      fetch-depth: 0 # CRITICAL: Required to calculate version history

  - uses: rira12621/simple-semantic-release@v1
    with:
      github_token: ${{ secrets.GITHUB_TOKEN }}
```

### Advanced: Build & Upload Artifacts
This pattern is perfect for compiled apps (Go, Rust, Swift, etc.). It calculates the version first (Dry Run), builds the app with that version, and then releases it.

```yaml
jobs:
  release:
    runs-on: macos-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      # 1. Calculate Version (Dry Run)
      - name: Calculate Version
        id: versioning
        uses: rira12621/simple-semantic-release@v1
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          dry_run: true

      # 2. Build your app using the calculated version
      - name: Build App
        run: |
          VERSION="${{ steps.versioning.outputs.version }}"
          echo "Building version $VERSION..."
          # Example build command
          go build -ldflags "-X main.version=$VERSION" -o myapp

      # 3. Publish Release & Upload Artifact
      - name: Publish Release
        uses: rira12621/simple-semantic-release@v1
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          artifacts: "myapp"
```

## ⚙️ Inputs

| Input | Description | Required | Default |
| :--- | :--- | :---: | :---: |
| `github_token` | The standard `GITHUB_TOKEN` or a PAT. Required to create tags/releases. | **Yes** | N/A |
| `artifacts` | Path to files to upload. Supports wildcards (e.g. `*.dmg`, `dist/*`). | No | `''` |
| `dry_run` | If `true`, calculates output version but does **not** tag or release. | No | `false` |

## 📤 Outputs

| Output | Description |
| :--- | :--- |
| `version` | The calculated next version string (e.g., `v1.2.3`). |
| `upload_url` | The URL for uploading assets to the created release. |

## ❗ Prerequisites

**1. `fetch-depth: 0`**
You must set `fetch-depth: 0` in your `actions/checkout` step. By default, GitHub only fetches the latest commit, which makes it impossible to calculate the previous tag.

```yaml
- uses: actions/checkout@v4
  with:
    fetch-depth: 0
```

**2. Commit Permissions**
Your workflow must have write permission to the repository contents to push tags and creates releases.

```yaml
permissions:
  contents: write
```

## 📏 Commit Message Rules
This action follows a simplified Conventional Commits specification:

* **Major Version** (`v1.0.0` -> `v2.0.0`): Commit message contains `!` or `BREAKING CHANGE`.
* **Minor Version** (`v1.0.0` -> `v1.1.0`): Commit message starts with `feat`.
* **Patch Version** (`v1.0.0` -> `v1.0.1`): Commit message starts with `fix`.

Examples:
* `fix: crash on startup` -> Patch bump
* `feat: add dark mode` -> Minor bump
* `feat!: rewrite API` -> Major bump
