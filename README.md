# React Native Android CI/CD Pipeline - Complete Guide

## 📋 Overview

This CI/CD pipeline automates the build, test, and deployment process for your React Native Android application using GitHub Actions. It ensures code quality through automated testing and provides consistent builds across different environments.

### What This Pipeline Does:
- ✅ Automatically runs linting and tests on every push/PR
- 📦 Builds debug APKs for testing on `develop` and `main` branches
- 🚀 Creates signed release APKs and AABs when pushing to `main`
- 🔄 Caches dependencies to speed up builds
- 📤 Uploads build artifacts for easy download

---

## 🏗️ Pipeline Architecture

```
Push/PR to main/develop
         ↓
    ┌────────────┐
    │  Test Job  │  ← Lint & Unit Tests
    └────────────┘
         ↓
    ┌────────────────────────────┐
    │                            │
    │  android-build (Debug)     │  android-release (Release)
    │  ↓                         │  ↓
    │  Builds Debug APK          │  Builds Signed APK & AAB
    │  ↓                         │  ↓
    │  Uploads Artifact          │  (Ready for Play Store)
    └────────────────────────────┘
```

**When Each Job Runs:**
- **test**: On every push and pull request to `main` or `develop`
- **android-build**: After tests pass, on every push/PR
- **android-release**: Only on pushes to `main` branch (not on PRs)

---

## 🚀 Quick Start Setup

### Step 1: Add Workflow File

Create `.github/workflows/android.yml` and paste your workflow code.

### Step 2: Add Required Scripts to package.json

```json
{
  "scripts": {
    "lint": "eslint .",
    "test": "jest"
  }
}
```

### Step 3: Generate Android Keystore

```bash
keytool -genkeypair -v -storetype PKCS12 \
  -keystore release.keystore \
  -alias my-key-alias \
  -keyalg RSA \
  -keysize 2048 \
  -validity 10000
```

**Save these securely:**
- Keystore password
- Key alias
- Key password

### Step 4: Configure Gradle Signing

Add to `android/app/build.gradle`:

```gradle
android {
    signingConfigs {
        release {
            if (project.hasProperty('MYAPP_RELEASE_STORE_FILE')) {
                storeFile file(MYAPP_RELEASE_STORE_FILE)
                storePassword MYAPP_RELEASE_STORE_PASSWORD
                keyAlias MYAPP_RELEASE_KEY_ALIAS
                keyPassword MYAPP_RELEASE_KEY_PASSWORD
            }
        }
    }
    buildTypes {
        release {
            signingConfig signingConfigs.release
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}
```

### Step 5: Add GitHub Secrets

Go to **Settings → Secrets and variables → Actions → New repository secret**

**Convert Keystore to Base64:**
```bash
# macOS/Linux
base64 -i release.keystore | pbcopy

# Windows PowerShell
[Convert]::ToBase64String([IO.File]::ReadAllBytes("release.keystore")) | Set-Clipboard
```

**Add these 4 secrets:**

| Secret Name | Value |
|-------------|-------|
| `ANDROID_KEYSTORE_BASE64` | Base64 encoded keystore |
| `KEY_ALIAS` | Your key alias (e.g., my-key-alias) |
| `KEYSTORE_PASSWORD` | Your keystore password |
| `KEY_PASSWORD` | Your key password |

### Step 6: Add .keystore to .gitignore

```bash
echo "*.keystore" >> .gitignore
git add .gitignore
git commit -m "Add keystore to gitignore"
```

---

## 📖 Pipeline Jobs Explained

### Job 1: Test (Quality Gate)
- Runs linting with `npm run lint`
- Executes unit tests with `npm test`
- Must pass before builds start
- Runs on every push and PR

### Job 2: Android Build (Debug)
- Runs after tests pass
- Builds unsigned debug APK
- Uploads APK as artifact
- For testing on devices

### Job 3: Android Release (Production)
- Only runs on `main` branch pushes
- Decodes keystore from secrets
- Builds signed release APK
- Builds AAB for Play Store
- Ready for production deployment

---

## 💻 How to Use

### For Daily Development

```bash
# 1. Create feature branch
git checkout -b feature/new-feature

# 2. Make changes
# ... edit files ...

# 3. Test locally first
npm run lint
npm test

# 4. Commit and push
git add .
git commit -m "feat: Add new feature"
git push origin feature/new-feature

# 5. Create PR to 'develop'
# Pipeline automatically runs tests and builds debug APK
```

### Download Build Artifacts

1. Go to **Actions** tab on GitHub
2. Click on your workflow run
3. Scroll to **Artifacts** section
4. Download `app-debug` artifact
5. Install: `adb install app-debug.apk`

### Release to Production

```bash
# Only for release managers
git checkout main
git merge develop
git push origin main

# Pipeline automatically creates signed APK and AAB
```

---

## 🐛 Common Issues & Solutions

### Issue 1: Tests Failed
```bash
# Run locally to debug
npm test

# Fix and push
git add .
git commit -m "fix: Resolve failing tests"
git push
```

### Issue 2: Lint Errors
```bash
# Check errors
npm run lint

# Auto-fix if possible
npm run lint -- --fix

# Commit fixes
git add .
git commit -m "style: Fix lint errors"
git push
```

### Issue 3: Gradle Build Failed
**Check:**
- All 4 GitHub secrets are set correctly
- No extra whitespace in secret values
- Keystore base64 encoding is correct

**Test locally:**
```bash
cd android
./gradlew assembleRelease \
  -PMYAPP_RELEASE_STORE_FILE=../release.keystore \
  -PMYAPP_RELEASE_KEY_ALIAS=my-key-alias \
  -PMYAPP_RELEASE_STORE_PASSWORD=yourpassword \
  -PMYAPP_RELEASE_KEY_PASSWORD=yourpassword
```

### Issue 4: Cannot Find Keystore
- Re-encode keystore without extra spaces
- Verify secret name is exactly `ANDROID_KEYSTORE_BASE64`
- Check if keystore was successfully decoded in logs

### Issue 5: Dependencies Installation Failed
```bash
# Clear and reinstall
rm -rf node_modules package-lock.json
npm install
npm ci --legacy-peer-deps

# Commit updated lock file
git add package-lock.json
git commit -m "chore: Update dependencies"
git push
```

---

## ✅ Best Practices

### Branching Strategy
```
main (production - signed releases)
  ↑
develop (staging - debug builds)
  ↑
feature/* (development)
```

### Before Pushing Code
```bash
npm run lint    # Check code style
npm test        # Run all tests
npm run build   # Verify build works
```

### Commit Message Format
```bash
# Good ✅
git commit -m "feat: Add user authentication"
git commit -m "fix: Resolve crash on startup"
git commit -m "test: Add login tests"

# Bad ❌
git commit -m "updates"
git commit -m "wip"
```

### Security Checklist
- ✅ Never commit `.keystore` files
- ✅ Never commit passwords
- ✅ Keep keystore backup secure
- ✅ Use different keystores for debug/release
- ✅ Rotate keys annually

---

## 🎓 Learning Path for Juniors

### Week 1: Basics
- Read this guide completely
- Understand Git workflow
- Watch a pipeline run
- Explore GitHub Actions tab

### Week 2: Practice
- Create a feature branch
- Make small changes
- Push and observe pipeline
- Download and test APK

### Week 3: Troubleshooting
- Break tests intentionally and fix
- Resolve lint errors
- Handle merge conflicts

### Week 4: Advanced
- Understand keystore management
- Learn Gradle build process
- Test release builds locally

---

## 🔧 Advanced Configuration

### Enable Play Store Deployment

**1. Create Service Account:**
- Go to [Google Play Console](https://play.google.com/console)
- Settings → API access
- Create service account
- Download JSON key

**2. Add GitHub Secret:**
```
GOOGLE_PLAY_SERVICE_ACCOUNT_JSON=<paste JSON content>
```

**3. Uncomment in workflow file:**
```yaml
- name: Deploy to Play Store
  uses: r0adkll/upload-google-play@v1
  with:
    serviceAccountJsonPlainText: ${{ secrets.GOOGLE_PLAY_SERVICE_ACCOUNT_JSON }}
    packageName: com.yourapp.package  # Update this!
    releaseFiles: android/app/build/outputs/bundle/release/app-release.aab
    track: internal
    status: completed
```

### Customize Node.js Version
```yaml
- name: Setup Node.js
  uses: actions/setup-node@v4
  with:
    node-version: '20'  # Change to '18' or '22' if needed
```

### Skip CI for Specific Commits
```bash
git commit -m "docs: Update README [skip ci]"
```

---

## 📊 Monitoring Pipeline

### Check Status
- Go to **Actions** tab
- View all workflow runs
- Click run for detailed logs

### Status Icons
- 🟢 Green: Success
- 🔴 Red: Failed
- 🟡 Yellow: In progress
- ⚪ Gray: Queued

### Artifacts Storage
- Stored for 90 days (default)
- Download from workflow run
- Can customize retention period

---

## 🛠️ Useful Commands

```bash
# Test Android build locally
cd android
./gradlew assembleDebug
./gradlew assembleRelease

# Install APK on device
adb install app-debug.apk

# View Android logs
adb logcat

# Clear Gradle cache
cd android
./gradlew clean

# Check outdated dependencies
npm outdated

# Update dependencies
npm update
```

---

## ❓ FAQ

**Q: How long does pipeline take?**  
A: 5-10 minutes typically. First run takes longer for caching.

**Q: Can I run on specific branches only?**  
A: Yes, modify `on:` section in workflow:
```yaml
on:
  push:
    branches: [main, develop, feature/*]
```

**Q: Where are artifacts stored?**  
A: GitHub Actions for 90 days by default.

**Q: Can I test pipeline locally?**  
A: Use [act](https://github.com/nektos/act):
```bash
brew install act
act -j test
```

**Q: How to get notified?**  
A: GitHub emails automatically on workflow completion.

---

## ✅ Pre-Launch Checklist

Before starting development:
- [ ] Workflow file created
- [ ] All 4 GitHub secrets added
- [ ] Keystore backed up securely
- [ ] Gradle signing configured
- [ ] `.gitignore` includes `*.keystore`
- [ ] `package.json` has lint and test scripts
- [ ] Local tests pass: `npm test`
- [ ] Local lint passes: `npm run lint`

---

## 📞 Getting Help

1. Check this README
2. Review GitHub Actions logs
3. Ask team lead
4. Create GitHub issue with error details

---

## 🔒 Security Notes

**NEVER commit these:**
- `*.keystore` files
- Passwords in code
- API keys or tokens
- Service account JSON

**ALWAYS:**
- Use GitHub Secrets for sensitive data
- Keep keystore backup offline
- Rotate credentials annually
- Review PR changes carefully

---

**Version:** 1.0.0  
**Last Updated:** October 2025  
**Maintained By:** [Your Team]

---

**Quick Reference Card:**

```bash
# Daily workflow
git checkout -b feature/xxx  → npm test → npm run lint → git push

# Check pipeline
GitHub → Actions → Click run → View logs

# Download APK
Actions → Artifacts → Download → adb install

# Release
Merge to main → Pipeline auto-creates signed APK/AAB

# Troubleshoot
Check logs → Fix locally → Push again
```

💡 **Remember:** The pipeline catches problems early. If it fails, that's good - it prevented a bad build from reaching users!