# Getting Credentials for App Store Publishing

This guide walks through obtaining the necessary credentials to automate publishing to Chrome Web Store, Firefox Add-ons, and Microsoft Edge Add-ons.

## Chrome Web Store

### Prerequisites
- A Google account
- The extension already published to Chrome Web Store (manual first upload required)
- Extension ID (visible in Chrome Web Store URL: `https://chromewebstore.google.com/detail/{EXTENSION_ID}`)

### Steps

1. **Get your Extension ID**
   - Go to [Chrome Web Store](https://chromewebstore.google.com)
   - Find your extension
   - Copy the ID from the URL: `...detail/{EXTENSION_ID}`
   - Store this value as `CHROME_EXTENSION_ID` secret

2. **Create OAuth2 Credentials**
   - Go to [Google Cloud Console](https://console.cloud.google.com/)
   - Create a new project (or use existing)
   - Enable "Chrome Web Store API"
   - Go to "Credentials" → "Create Credentials" → "OAuth 2.0 Client ID"
   - Choose "Desktop application"
   - Download the JSON file
   - From the JSON, extract:
     - `client_id` → Store as `CHROME_CLIENT_ID`
     - `client_secret` → Store as `CHROME_CLIENT_SECRET`

3. **Get Refresh Token**
   - Use [@fregante's token generator](https://github.com/fregante/chrome-webstore-upload-cli#readme):
     ```bash
     npx chrome-webstore-upload-cli auth https://accounts.google.com/o/oauth2/token
     ```
   - Follow the browser prompt to authorize
   - Copy the refresh token → Store as `CHROME_REFRESH_TOKEN`

⚠️ **Important**: Keep these credentials secret. Never commit them to git.

---

## Firefox Add-ons

### Prerequisites
- A Firefox Account (https://accounts.firefox.com)
- The extension already listed on [addons.mozilla.org](https://addons.mozilla.org) (manual first upload required)

### Steps

1. **Get Extension GUID**
   - Go to [addons.mozilla.org/developers](https://addons.mozilla.org/en-US/developers/)
   - Find your extension
   - Copy the GUID from the URL or extension details
   - Store as `FIREFOX_ADDON_GUID`

2. **Generate API Credentials**
   - Go to [addons.mozilla.org/developers/addon/api/key/](https://addons.mozilla.org/en-US/developers/addon/api/key/)
   - Click "Generate New Credentials"
   - Set expiration (e.g., 1 year)
   - Copy:
     - **API Key** → Store as `FIREFOX_API_KEY`
     - **API Secret** → Store as `FIREFOX_API_SECRET`

⚠️ **Important**: Firefox API credentials are sensitive. Use GitHub Secrets.

---

## Microsoft Edge Add-ons

### Prerequisites
- A Microsoft account
- The extension already submitted to Microsoft Edge Add-ons (manual first upload required)
- Product ID (from Microsoft Partner Center)

### Steps

1. **Get Product ID**
   - Go to [Microsoft Partner Center](https://partner.microsoft.com/dashboard)
   - Sign in with your Microsoft account
   - Navigate to your extension in "Edge Add-ons"
   - Find the **Product ID** in the extension details
   - Store as `EDGE_PRODUCT_ID`

2. **Create Azure AD App Registration**
   - Go to [Azure Portal](https://portal.azure.com) → "Azure Active Directory"
   - Click "App registrations" → "New registration"
   - Register an app (any name, e.g., "extension-publisher")
   - Copy the **Application (client) ID** → Store as `EDGE_CLIENT_ID`
   - Go to "Certificates & secrets" → "New client secret"
   - Copy the secret value → Store as `EDGE_CLIENT_SECRET`
   - Copy the **Directory (tenant) ID** → Use in next step

3. **Get Access Token URL**
   - Use your Directory (tenant) ID from above
   - Access Token URL format: `https://login.microsoftonline.com/{DIRECTORY_ID}/oauth2/v2.0/token`
   - Store as `EDGE_ACCESS_TOKEN_URL`

4. **Grant Permissions**
   - In Azure AD app, go to "API permissions"
   - Add permission: `Microsoft Edge Add-ons API`
   - Grant admin consent if prompted

---

## Summary: Secrets to Store in GitHub

For **each extension repository**, add these secrets to GitHub:

```
CHROME_EXTENSION_ID       (from Chrome Web Store URL)
CHROME_CLIENT_ID          (from Google Cloud Console)
CHROME_CLIENT_SECRET      (from Google Cloud Console)
CHROME_REFRESH_TOKEN      (from Chrome WebStore Upload CLI)

FIREFOX_ADDON_GUID        (from addons.mozilla.org)
FIREFOX_API_KEY           (from addons.mozilla.org/developers)
FIREFOX_API_SECRET        (from addons.mozilla.org/developers)

EDGE_PRODUCT_ID           (from Microsoft Partner Center)
EDGE_CLIENT_ID            (from Azure AD app)
EDGE_CLIENT_SECRET        (from Azure AD app)
EDGE_ACCESS_TOKEN_URL     (constructed from Azure Directory ID)
```

### How to Add Secrets to GitHub

1. Go to your extension repo on GitHub
2. Settings → Secrets and variables → Actions
3. Click "New repository secret"
4. Add each secret with the name and value
5. Repeat for all secrets listed above

---

## Security Best Practices

- ✅ Use GitHub Secrets for all credentials (never commit to git)
- ✅ Rotate tokens annually
- ✅ Use service accounts / API-only accounts when possible (not personal)
- ✅ Limit API scope to "publish" only
- ✅ Enable 2FA on accounts that generate credentials
- ✅ Review GitHub Actions logs carefully (don't log secrets)
- ⚠️ Regenerate secrets immediately if committed accidentally

---

## Troubleshooting

**"Invalid client_id/client_secret"**
- Verify you're using the correct values from Google Cloud Console
- Ensure the Chrome Web Store API is enabled in your project

**"Addon not found" (Firefox)**
- Verify the GUID is correct (should be `{xxxxx@xxxxx}` format or email-like)
- Ensure the add-on is listed on addons.mozilla.org

**"Unauthorized" (Edge)**
- Verify Product ID is correct
- Ensure Azure AD app has the correct API permissions
- Check that admin consent was granted
