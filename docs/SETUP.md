# Setting Up App Store Publishing for Your Extension

This guide explains how to integrate the reusable publish workflow into your extension repository.

## Step 1: Add Repository Secrets

Before setting up the workflow, add all required secrets to your GitHub repository.

See [CREDENTIALS.md](./CREDENTIALS.md) for how to obtain these values.

**In your GitHub repo:**
1. Settings → Secrets and variables → Actions
2. Add these secrets:

```
CHROME_EXTENSION_ID
CHROME_CLIENT_ID
CHROME_CLIENT_SECRET
CHROME_REFRESH_TOKEN
FIREFOX_ADDON_GUID
FIREFOX_API_KEY
FIREFOX_API_SECRET
EDGE_PRODUCT_ID
EDGE_CLIENT_ID
EDGE_CLIENT_SECRET
EDGE_ACCESS_TOKEN_URL
```

## Step 2: Create Workflow in Your Extension Repo

In your extension repository, create `.github/workflows/publish.yml`:

```yaml
name: Publish to App Stores

on:
  push:
    tags:
      - "v*.*.*"  # Triggers on version tags: v1.0.0, v1.1.0, etc.

jobs:
  publish:
    uses: username/extension-workflows/.github/workflows/publish.yml@main
    with:
      extension_name: "Your Extension Name"
      build_command: "build"
      chrome_zip_path: "dist/chrome-mv3"
      firefox_zip_path: "dist/firefox"
      edge_zip_path: "dist/chrome-mv3"
      publish_chrome: true
      publish_firefox: true
      publish_edge: true
    secrets: inherit
```

### Configuration Options

| Option | Default | Description |
|--------|---------|-------------|
| `extension_name` | (required) | Display name for release notes |
| `build_command` | `build` | npm script that builds your extension |
| `chrome_zip_path` | `dist/chrome-mv3` | Path to built Chrome extension directory |
| `firefox_zip_path` | `dist/firefox` | Path to built Firefox extension directory |
| `edge_zip_path` | `dist/chrome-mv3` | Path to built Edge extension directory |
| `publish_chrome` | `true` | Whether to publish to Chrome Web Store |
| `publish_firefox` | `true` | Whether to publish to Firefox Add-ons |
| `publish_edge` | `true` | Whether to publish to Edge Add-ons |

### Example: Selective Publishing

If you only want to publish to some stores initially:

```yaml
jobs:
  publish:
    uses: username/extension-workflows/.github/workflows/publish.yml@main
    with:
      extension_name: "My Extension"
      publish_chrome: true
      publish_firefox: false        # Skip Firefox
      publish_edge: false           # Skip Edge
    secrets: inherit
```

## Step 3: Verify Your Build Process

The workflow expects certain build outputs. Verify your `package.json` build scripts:

```json
{
  "scripts": {
    "build": "node scripts/build.js chrome",
    "build:firefox": "node scripts/build.js firefox",
    "build:edge": "node scripts/build.js chrome"
  }
}
```

The `build` script should produce these directories:
- `dist/chrome-mv3/` — Chrome Web Store submission
- `dist/firefox/` — Firefox Add-ons submission (XPI format)
- `dist/chrome-mv3/` — Edge Add-ons submission (uses Chrome format)

## Step 4: Update package.json Version

The workflow extracts the version from `package.json`. Ensure it follows semantic versioning:

```json
{
  "version": "1.0.0"
}
```

## Step 5: Create a Git Tag and Push

To trigger the publishing workflow:

```bash
npm version patch    # Bumps version (1.0.0 → 1.0.1) and creates tag
git push --tags
```

Or manually:

```bash
# Update version in package.json manually
git tag v1.0.1
git push origin v1.0.1
```

The workflow will:
1. ✅ Checkout your code
2. ✅ Install dependencies
3. ✅ Build the extension
4. ✅ Publish to all configured stores
5. ✅ Create a GitHub Release
6. ✅ Post a summary to the workflow run

## Step 6: Monitor the Workflow Run

1. Go to your repo → "Actions" tab
2. Find the "Publish to App Stores" workflow run
3. Check the logs for each store's status
4. Review the summary at the bottom

### What to Expect

**Chrome Web Store**: Takes 1-7 days for review  
**Firefox Add-ons**: Typically 1-2 days for review  
**Edge Add-ons**: Usually 1-3 days for review

## Troubleshooting

### "Workflow not triggered on tag push"
- Verify the tag format matches `v*.*.*` (e.g., `v1.0.0`)
- Check that your workflow file is in `.github/workflows/` and is valid YAML

### "Secrets not found"
- Verify all secrets are added to your GitHub repository
- Use exact secret names (case-sensitive)
- Run workflows → re-run failed job to pick up newly added secrets

### "Build command not found"
- Verify `npm run build` works locally
- Check that your build script in `package.json` is correct
- Ensure the output directories match the configured paths

### "Extension not found on store"
- Verify the extension IDs/GUIDs are correct
- Ensure the extension is already published on each store (initial upload must be manual)
- Check that credentials have the right permissions

### "Rate limited"
- Don't publish more than once per hour per extension
- Space out releases to allow store review times

## Next Steps

Once you have this working for one extension, you can:

1. **Reuse for other extensions** by copying `.github/workflows/publish.yml` and updating the `with:` section
2. **Add to this repository** by creating a shared workflow configuration
3. **Automate version bumping** with additional GitHub Actions

---

**Questions?** See [CREDENTIALS.md](./CREDENTIALS.md) for credential setup or GitHub Actions documentation for workflow syntax.
