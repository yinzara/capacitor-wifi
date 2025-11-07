# Publishing Guide for @geniesmart/ln-capacitor-wifi

## Published Package Information

- **Package Name**: `@geniesmart/ln-capacitor-wifi`
- **Current Version**: 0.3.0
- **Registry**: Google Artifact Registry
- **Repository**: `npm-packages`
- **Location**: `us-central1`
- **Project**: `geniesmart-cloud-dev`

## Installation in Consumer Projects

To use this package in your Capacitor projects, add the following to your project's `.npmrc`:

```
@geniesmart:registry=https://us-central1-npm.pkg.dev/geniesmart-cloud-dev/npm-packages/
//us-central1-npm.pkg.dev/geniesmart-cloud-dev/npm-packages/:always-auth=true
```

Then install the package:

```bash
npm install @geniesmart/ln-capacitor-wifi
```

## Publishing New Versions

### 1. Update Version

Edit `package.json` and bump the version:

```bash
npm version patch  # 0.3.0 -> 0.3.1
npm version minor  # 0.3.0 -> 0.4.0
npm version major  # 0.3.0 -> 1.0.0
```

### 2. Build the Package

```bash
npm run build
```

### 3. Publish to Artifact Registry

Use the following command with your gcloud user credentials:

```bash
TOKEN=$(gcloud auth print-access-token) && npm publish --//us-central1-npm.pkg.dev/geniesmart-cloud-dev/npm-packages/:_authToken=$TOKEN
```

### Alternative: Using npx for Authentication

You can also set up persistent authentication:

```bash
npx google-artifactregistry-auth
```

Then simply run:

```bash
npm publish
```

## Verification

Check published versions:

```bash
gcloud artifacts versions list \
  --package=@geniesmart/ln-capacitor-wifi \
  --repository=npm-packages \
  --location=us-central1 \
  --format="table(name,createTime)"
```

## Required Permissions

To publish packages, you need the following IAM role:
- `roles/artifactregistry.writer` on the `npm-packages` repository

Grant permissions:

```bash
gcloud artifacts repositories add-iam-policy-binding npm-packages \
  --location=us-central1 \
  --member="user:EMAIL@DOMAIN" \
  --role="roles/artifactregistry.writer"
```

## Configuration Files

### .npmrc (Project Level)
```
//us-central1-npm.pkg.dev/geniesmart-cloud-dev/npm-packages/:always-auth=true
@geniesmart:registry=https://us-central1-npm.pkg.dev/geniesmart-cloud-dev/npm-packages/
```

### package.json (publishConfig)
```json
{
  "publishConfig": {
    "registry": "https://us-central1-npm.pkg.dev/geniesmart-cloud-dev/npm-packages/"
  }
}
```

## Capacitor 7 Migration Completed

This package has been upgraded from Capacitor 5 to Capacitor 7:

### Platform Updates
- **iOS**: Deployment target updated to 14.0+
- **Android**: SDK 35, Gradle 8.11.1, Java 21
- **TypeScript**: 5.x with skipLibCheck enabled
- **Rollup**: 5.x

### Breaking Changes
- Requires Node.js 20+
- Requires iOS 14.0+ (was 13.0+)
- Requires Android SDK 35 (was 33)
- Requires Java 21 for Android builds
- Requires Xcode 16.0+ for iOS builds
- Peer dependency updated to `@capacitor/core@^7.0.0`

## Package Scope Changed

The package was renamed from `ln-capacitor-wifi` to `@geniesmart/ln-capacitor-wifi` to clearly indicate it's a private package in your organization's registry.

## View Package in Cloud Console

https://console.cloud.google.com/artifacts/npm/geniesmart-cloud-dev/us-central1/npm-packages/@geniesmart/ln-capacitor-wifi?project=geniesmart-cloud-dev