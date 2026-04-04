# Roqya App — Android AAB (TWA)

This project wraps the PWA at **https://roqya-app.web.app** into an Android App Bundle (.aab) using a Trusted Web Activity (TWA).

## How to Build (GitHub Actions)

### Step 1 — Push this folder to a GitHub repository

Create a new repo on GitHub and push this entire folder as the root of your repository.

### Step 2 — Generate a signing keystore

You need a keystore to sign your app for the Play Store. Run this command on your computer (requires Java):

```bash
keytool -genkey -v \
  -keystore roqya-release.jks \
  -alias roqya \
  -keyalg RSA \
  -keysize 2048 \
  -validity 10000
```

Fill in the prompts. Keep the `.jks` file and the passwords safe — you'll need them every time you publish.

### Step 3 — Add GitHub Secrets

In your GitHub repo, go to **Settings → Secrets and variables → Actions** and add:

| Secret name        | Value                                                              |
|--------------------|--------------------------------------------------------------------|
| `KEYSTORE_BASE64`  | Your keystore file base64-encoded (see command below)              |
| `KEYSTORE_PASSWORD`| The password you set for the keystore store                        |
| `KEY_ALIAS`        | The alias you used (e.g. `roqya`)                                  |
| `KEY_PASSWORD`     | The password you set for the key                                   |

To get the base64 value of your keystore:
```bash
base64 -i roqya-release.jks | pbcopy   # Mac (copies to clipboard)
base64 -w 0 roqya-release.jks          # Linux
```

### Step 4 — Trigger the build

Push any commit to `main` or `master`. GitHub Actions will automatically:
1. Set up Java 17 and Android SDK
2. Sign your app with your keystore
3. Build the `.aab`
4. Upload it as a downloadable artifact (available for 30 days)

Go to **Actions → Build Android AAB → your run → Artifacts** to download `roqya-app-release.zip` which contains the `.aab`.

### Step 5 — Link your PWA domain (Digital Asset Links)

For the TWA to work without showing the browser URL bar, you must verify domain ownership.

1. Go to the [Asset Links Tool](https://developers.google.com/digital-asset-links/tools/generator) or generate manually.
2. Add this file to your PWA host at: `https://roqya-app.web.app/.well-known/assetlinks.json`

```json
[{
  "relation": ["delegate_permission/common.handle_all_urls"],
  "target": {
    "namespace": "android_app",
    "package_name": "com.tyriaq.roqya",
    "sha256_cert_fingerprints": ["YOUR_SHA256_FINGERPRINT_HERE"]
  }
}]
```

To get your SHA-256 fingerprint from the keystore:
```bash
keytool -list -v -keystore roqya-release.jks -alias roqya
```

Copy the `SHA256:` line (formatted as `AA:BB:CC:...`) into the JSON above.

### Step 6 — Upload to Play Store

Upload the downloaded `.aab` file in the [Google Play Console](https://play.google.com/console) under **Production → Create new release**.

## App details

- **Package name**: `com.tyriaq.roqya`
- **PWA URL**: `https://roqya-app.web.app`
- **Theme color**: `#0D0B08`
- **Min Android SDK**: 21 (Android 5.0+)
