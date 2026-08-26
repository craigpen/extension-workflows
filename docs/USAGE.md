# Usage: Integrating with Download Nexus

This is a concrete example of how to integrate the reusable publish workflow with the Download Nexus extension.

## For Download Nexus

### 1. Create `.github/workflows/publish.yml`

In your `download-nexus` repository, add:

```yaml
name: Publish to App Stores

on:
  push:
    tags:
      - "v*.*.*"

jobs:
  publish:
    uses: username/extension-workflows/.github/workflows/publish.yml@main
    with:
      extension_name: "Download Nexus"
      build_command: "build"
      chrome_zip_path: "dist/chrome-mv3"
      firefox_zip_path: "dist/chrome-mv3"
      edge_zip_path: "dist/chrome-mv3"
      publish_chrome: true
      publish_firefox: true
      publish_edge: true
    secrets: inherit
```

### 2. Add Repository Secrets

In your `download-nexus` GitHub repo settings, add all required secrets (see [CREDENTIALS.md](./CREDENTIALS.md)):

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

### 3. Verify Build Output

Check that your current build creates the expected output:

```bash
npm run build
ls -la dist/

# Expected output:
# dist/chrome-mv3/manifest.json
# dist/chrome-mv3/popup.html
# dist/chrome-mv3/background.js
# etc.
```

### 4. Prepare Your First Release

Update `package.json` version to your intended release version:

```json
{
  "version": "1.1.6"
}
```

### 5. Create and Push the Git Tag

```bash
# Option A: Let npm handle versioning
npm version patch  # Bumps 1.1.5 → 1.1.6
git push --tags

# Option B: Manual tagging
git tag v1.1.6
git push origin v1.1.6
```

### 6. Monitor the Workflow

1. Go to your repo on GitHub
2. Click "Actions" tab
3. Find "Publish to App Stores" workflow
4. Watch the logs as it builds and publishes

### 7. Verify Publication

After the workflow completes:

- **Chrome Web Store**: Check your extension details page; may take 1-7 days to appear publicly
- **Firefox Add-ons**: Check your Add-ons page; may take 1-2 days
- **Edge Add-ons**: Check your Edge Add-ons listing; may take 1-3 days

---

## Adding New Extensions

Once this is working for Download Nexus, you can reuse it for other extensions:

### For Extension #2 (e.g., `Tab Lifecycle Manager`)

1. **In the new extension repo**, create `.github/workflows/publish.yml`:

```yaml
name: Publish to App Stores

on:
  push:
    tags:
      - "v*.*.*"

jobs:
  publish:
    uses: username/extension-workflows/.github/workflows/publish.yml@main
    with:
      extension_name: "Tab Lifecycle Manager"  # ← Different name
      build_command: "build"
      chrome_zip_path: "dist/chrome-mv3"
      firefox_zip_path: "dist/chrome-mv3"
      edge_zip_path: "dist/chrome-mv3"
      publish_chrome: true
      publish_firefox: true
      publish_edge: true
    secrets: inherit
```

2. Add the same repository secrets to the new repo

3. Tag and push:

```bash
npm version minor
git push --tags
```

That's it! The workflow automatically:
- Builds your extension
- Publishes to all three stores
- Creates a GitHub Release
- Posts a summary

---

## Manual Publishing (If Workflow Fails)

If the automated workflow fails and you need to publish manually:

### Chrome Web Store

```bash
npm i -g chrome-webstore-upload-cli

chrome-webstore-upload-cli upload \
  --source dist/chrome-mv3 \
  --extension-id $CHROME_EXTENSION_ID \
  --client-id $CHROME_CLIENT_ID \
  --client-secret $CHROME_CLIENT_SECRET \
  --refresh-token $CHROME_REFRESH_TOKEN

# Publish after upload
chrome-webstore-upload-cli publish \
  --extension-id $CHROME_EXTENSION_ID \
  --client-id $CHROME_CLIENT_ID \
  --client-secret $CHROME_CLIENT_SECRET \
  --refresh-token $CHROME_REFRESH_TOKEN
```

### Firefox Add-ons

```bash
npm i -g web-ext

web-ext sign \
  --api-key $FIREFOX_API_KEY \
  --api-secret $FIREFOX_API_SECRET \
  --source-dir dist/chrome-mv3
```

### Edge Add-ons

```bash
npm i -g edge-addons-upload-cli

edge-addons-upload-cli upload \
  --zip dist/chrome-mv3 \
  --product-id $EDGE_PRODUCT_ID \
  --client-id $EDGE_CLIENT_ID \
  --client-secret $EDGE_CLIENT_SECRET \
  --access-token-url $EDGE_ACCESS_TOKEN_URL
```

---

## Key Points to Remember

✅ **Always bump version first** — Workflow reads from `package.json`  
✅ **Use semantic versioning** — `v1.2.3` format (MAJOR.MINOR.PATCH)  
✅ **Tag format matters** — Must be `v*.*.*` to trigger workflow  
✅ **Store review takes time** — Don't push multiple versions in quick succession  
✅ **Monitor the workflow** — Check GitHub Actions logs for issues  
✅ **Keep secrets secure** — Use GitHub Secrets, never commit credentials  

---

**See [SETUP.md](./SETUP.md) for full configuration details or [CREDENTIALS.md](./CREDENTIALS.md) for credential setup.**
