# Extension Workflows

Internal reusable GitHub Actions workflow for automating browser extension publishing to Chrome Web Store, Firefox Add-ons, and Microsoft Edge Add-ons.

Used by: Download Nexus, Tab Lifecycle Manager, and future extensions.

**One workflow. Three app stores. Zero manual uploads.**

## Quick Start

1. **[Get credentials](./docs/CREDENTIALS.md)** from each app store (one-time per extension)
2. **[Set up secrets](./docs/SETUP.md)** in your extension's GitHub repository
3. **Add workflow** to your extension repo (`.github/workflows/publish.yml`)
4. **Tag and push**: `npm version patch && git push --tags`

See [Usage Example](./docs/USAGE.md) for Download Nexus integration.

---

## What This Does

When you push a version tag (`v1.0.0`), the workflow automatically:

✅ Builds your extension  
✅ Publishes to Chrome Web Store  
✅ Publishes to Firefox Add-ons  
✅ Publishes to Microsoft Edge Add-ons  
✅ Creates a GitHub Release  
✅ Posts a summary with links  

All in one action. No manual uploads. No FTP. No web dashboard clicks.

**Note:** This workflow publishes **code/versions only**. Store listing metadata (name, description, screenshots, category) must be updated manually in each store's developer dashboard.

---

## Features

- **Reusable**: One workflow template for all extensions
- **Flexible**: Publish to any combination of stores (Chrome, Firefox, Edge, or all three)
- **Safe**: Uses GitHub Secrets for all credentials; never commits tokens
- **Observable**: Detailed logs and workflow summaries for each run
- **Resilient**: Continues if one store fails; doesn't block others
- **Scalable**: Works for 1 extension or 50+

---

## Documentation

| Document | Purpose |
|----------|---------|
| [CREDENTIALS.md](./docs/CREDENTIALS.md) | How to obtain API keys from each store |
| [SETUP.md](./docs/SETUP.md) | How to integrate the workflow into your extension repo |
| [USAGE.md](./docs/USAGE.md) | Concrete example (Download Nexus) and multi-extension setup |

---

## Architecture

```
extension-workflows/  (this repo)
└── .github/workflows/
    └── publish.yml     (reusable workflow)

your-extension/       (your repo)
└── .github/workflows/
    └── publish.yml    (calls the reusable workflow above)
```

Each extension repo references the shared workflow via:

```yaml
jobs:
  publish:
    uses: username/extension-workflows/.github/workflows/publish.yml@main
    with:
      extension_name: "Your Extension"
    secrets: inherit
```

---

## Supported Stores

### Chrome Web Store
- **First upload**: Manual (via web UI)
- **Automated updates**: Yes
- **Review time**: 1-7 days
- **Requirement**: OAuth2 credentials (Google APIs)

### Firefox Add-ons
- **First upload**: Manual (via web UI)
- **Automated updates**: Yes
- **Review time**: 1-2 days typically
- **Requirement**: API key + secret (addons.mozilla.org)

### Microsoft Edge Add-ons
- **First upload**: Manual (via Partner Center)
- **Automated updates**: Yes
- **Review time**: 1-3 days
- **Requirement**: Azure AD credentials

---

## Requirements

### For Your Extension Repo

- `package.json` with `version` field (semantic versioning)
- `npm run build` script that outputs to `dist/`
- Extension already listed on each store (first upload is manual)
- GitHub repository with Actions enabled

### For This Repo

- Separate GitHub repository (`extension-workflows`)
- Extension repos reference it via: `uses: username/extension-workflows/.github/workflows/publish.yml@main`

---

## Workflow Configuration

The reusable workflow accepts these inputs:

```yaml
inputs:
  extension_name:          # Required: Display name for releases
  build_command:           # npm script to build (default: "build")
  chrome_zip_path:         # Path to Chrome build (default: dist/chrome-mv3)
  firefox_zip_path:        # Path to Firefox build (default: dist/firefox)
  edge_zip_path:           # Path to Edge build (default: dist/chrome-mv3)
  publish_chrome:          # Publish to Chrome? (default: true)
  publish_firefox:         # Publish to Firefox? (default: true)
  publish_edge:            # Publish to Edge? (default: true)
```

See [SETUP.md](./docs/SETUP.md#configuration-options) for details.

---

## Secrets Required

Each extension repo needs these GitHub Secrets:

```
CHROME_EXTENSION_ID, CHROME_CLIENT_ID, CHROME_CLIENT_SECRET, CHROME_REFRESH_TOKEN
FIREFOX_ADDON_GUID, FIREFOX_API_KEY, FIREFOX_API_SECRET
EDGE_PRODUCT_ID, EDGE_CLIENT_ID, EDGE_CLIENT_SECRET, EDGE_ACCESS_TOKEN_URL
```

See [CREDENTIALS.md](./docs/CREDENTIALS.md) for how to obtain these values.

---

## Example Workflow Output

```
✅ Build extension
✅ Publish to Chrome Web Store
✅ Publish to Firefox Add-ons
✅ Publish to Microsoft Edge Add-ons
✅ Create GitHub Release

## Publishing Summary
Extension: Download Nexus
Version: v1.1.6
Stores Published To:
- Chrome Web Store: ✅
- Firefox Add-ons: ✅
- Edge Add-ons: ✅
```

---

## Troubleshooting

**Workflow not triggered?**
- Verify tag format: `v1.0.0` (matches `v*.*.*`)
- Check Actions tab to see if workflow ran
- Confirm `.github/workflows/publish.yml` exists in your extension repo

**Credentials not found?**
- Verify all secrets are added to GitHub repository (Settings → Secrets)
- Use exact names (case-sensitive)
- Regenerate secrets if you suspect they're incorrect

**Build fails?**
- Run `npm run build` locally to verify it works
- Check that build output paths match configured paths
- Review GitHub Actions logs for specific errors

**One store fails but I need to retry?**
- The workflow continues even if one store fails
- Click "Re-run failed jobs" to retry
- Or manually publish using the store's CLI (see [USAGE.md](./docs/USAGE.md))

See [SETUP.md Troubleshooting](./docs/SETUP.md#troubleshooting) for more.

---

## Security

- ✅ All credentials stored in GitHub Secrets (never in code)
- ✅ Workflow uses `secrets: inherit` (safe for reusable workflows)
- ✅ Minimal permissions (only what's needed to publish)
- ✅ No third-party containers (uses official GitHub Actions)
- ⚠️ Rotate credentials annually
- ⚠️ Don't use personal accounts for API credentials; use service accounts

---

## Testing Locally

To test the build locally before pushing:

```bash
# In your extension repo
npm run build
ls dist/chrome-mv3/     # Verify build outputs
```

The workflow does the same thing, just automated.

---

## Multi-Extension Setup

Once you have this working for one extension, adding others is simple:

1. Create new extension repo
2. Add `.github/workflows/publish.yml` (copy from first extension, change `extension_name`)
3. Add the same secrets to the new repo
4. Tag and push

Both extensions share the same reusable workflow in this `extension-workflows` repo.

---

## Workflow Trigger

The workflow triggers on git tags matching `v*.*.*`:

```bash
# Trigger the workflow
npm version patch      # Bumps version and creates tag
git push --tags

# Or manually
git tag v1.0.1
git push origin v1.0.1
```

---

## What Happens After Publish

Each app store has its own review process:

| Store | Review Time | Auto-Publish? | Notes |
|-------|-------------|---------------|-------|
| Chrome | 1-7 days | Yes (after review) | Most unpredictable |
| Firefox | 1-2 days | Yes | Fastest typically |
| Edge | 1-3 days | Yes | Reliable & fast |

You'll see updates in each store's developer dashboard as they process your submission.

---

## Improving the Workflow

Need to update the workflow? Add support for a new store? Improve the docs?

- Test changes locally with an extension first
- Update docs when adding features or changing behavior
- Keep it simple—this is internal infrastructure

---

## Getting Started

👉 **First time?** Start with [CREDENTIALS.md](./docs/CREDENTIALS.md)  
👉 **Setting up an extension?** Read [SETUP.md](./docs/SETUP.md)  
👉 **Want a concrete example?** See [USAGE.md](./docs/USAGE.md)  

---

**Questions?** Check the docs or review the workflow definition in `.github/workflows/publish.yml`.
