# 🚀 Play Store CI/CD Setup Guide

## ✅ What Has Been Fixed in Your CI/CD

### 1. **Play Store Deployment Step** - ✅ ENABLED
- Uncommented the Play Store deployment step
- Fixed package name from `com.yourapp.package` → `com.reactnativecicd`
- Deployment now happens automatically after AAB build

### 2. **Android SDK Setup** - ✅ ADDED
- Added Android SDK setup to the `android-release` job (was missing)
- Required for building Android apps

### 3. **Node.js Version** - ✅ FIXED
- Standardized to Node.js 20 (was inconsistent `lts/*`)
- Ensures consistent builds

### 4. **Artifact Uploads** - ✅ ENABLED
- Release APK and AAB are now uploaded as artifacts
- You can download them from GitHub Actions

---

## 🔴 What You Still Need to Do

### **Step 1: Create Google Play Service Account** (REQUIRED)

1. **Go to Google Play Console:**
   - Visit: https://play.google.com/console
   - Sign in with your Google account

2. **Navigate to API Access:**
   - Go to: **Settings** → **API access**
   - Click **Create new service account**

3. **Create Service Account in Google Cloud:**
   - This opens Google Cloud Console
   - Click **Create Service Account**
   - Name it: `github-actions-play-store`
   - Click **Create and Continue**
   - Grant role: **Editor** (or **Owner** for full access)
   - Click **Continue** → **Done**

4. **Grant Play Console Access:**
   - Back in Play Console, find your new service account
   - Click **Grant Access**
   - Select permissions:
     - ✅ **Release apps to production** (or Internal testing)
     - ✅ **Release apps to testing tracks**
     - ✅ **View app information**
   - Click **Invite User**

5. **Download JSON Key:**
   - In Google Cloud Console → IAM & Admin → Service Accounts
   - Click on your service account
   - Go to **Keys** tab → **Add Key** → **Create new key**
   - Choose **JSON** format
   - **Download** the JSON file (keep it secure!)

### **Step 2: Add GitHub Secret** (REQUIRED)

1. **Open the JSON file** you just downloaded
2. **Copy the ENTIRE contents** (it's a single JSON object)

3. **Add to GitHub Secrets:**
   - Go to your GitHub repository
   - Click **Settings** → **Secrets and variables** → **Actions**
   - Click **New repository secret**
   - Name: `GOOGLE_PLAY_SERVICE_ACCOUNT_JSON`
   - Value: **Paste the entire JSON content** (make sure it's on one line or properly formatted)
   - Click **Add secret**

**⚠️ Important:** The JSON should look like this (one line or properly formatted):
```json
{"type":"service_account","project_id":"your-project",...}
```

---

## 🎯 How Play Store CI/CD Works Now

### **Flow Diagram:**
```
Push to main branch
    ↓
Run Tests (lint + unit tests)
    ↓
Build Signed Release APK & AAB
    ↓
Upload Artifacts to GitHub
    ↓
🆕 Deploy AAB to Google Play Store (Internal Track)
    ↓
✅ Done! App is in Play Console
```

### **What Happens on Each Push to `main`:**

1. ✅ **Tests run** - Lint and unit tests
2. ✅ **AAB built** - Signed Android App Bundle created
3. ✅ **Artifacts uploaded** - You can download APK/AAB from GitHub
4. ✅ **Play Store deployment** - Automatically uploads to Internal track
5. ✅ **Status: completed** - App is ready for testing/review

---

## 📝 Configuration Details

### **Current Deployment Settings:**

```yaml
track: internal          # Deploys to Internal testing track
status: completed        # App is ready immediately (no draft)
packageName: com.reactnativecicd
```

### **Available Tracks:**
- `internal` - Internal testing (fastest, up to 100 testers)
- `alpha` - Alpha testing (up to 20,000 testers)
- `beta` - Beta testing (unlimited testers)
- `production` - Public release

### **Available Status:**
- `completed` - Ready immediately (for testing tracks)
- `draft` - Manual review needed (for production)

---

## 🧪 Testing the Setup

### **Test the Deployment:**

1. **Push to main branch:**
   ```bash
   git checkout main
   git push origin main
   ```

2. **Monitor GitHub Actions:**
   - Go to **Actions** tab
   - Watch the workflow run
   - Check for any errors

3. **Verify in Play Console:**
   - Go to Play Console → **Release** → **Testing** → **Internal testing**
   - You should see your new build!

### **If Deployment Fails:**

**Error: "Service account not found"**
- ❌ JSON secret is missing or incorrect
- ✅ Solution: Re-add `GOOGLE_PLAY_SERVICE_ACCOUNT_JSON` secret

**Error: "Package name mismatch"**
- ❌ Package name in workflow doesn't match Play Console
- ✅ Solution: Verify `com.reactnativecicd` matches your app

**Error: "Insufficient permissions"**
- ❌ Service account lacks permissions
- ✅ Solution: Grant proper permissions in Play Console

**Error: "Track not found"**
- ❌ Internal track doesn't exist
- ✅ Solution: Create Internal testing track in Play Console

---

## 🎛️ Customization Options

### **Change Deployment Track:**

Edit `.github/workflows/android-ci-cd.yml`:

```yaml
track: alpha  # or beta, production
```

### **Use Draft Status (Manual Review):**

```yaml
status: draft  # Requires manual review in Play Console
```

### **Deploy to Multiple Tracks:**

Add multiple deployment steps:

```yaml
# Deploy to Internal
- name: Deploy to Internal
  uses: r0adkll/upload-google-play@v1
  with:
    track: internal
    # ... other settings

# Deploy to Alpha
- name: Deploy to Alpha
  uses: r0adkll/upload-google-play@v1
  with:
    track: alpha
    # ... other settings
```

### **Add Release Notes:**

```yaml
- name: Deploy to Play Store
  uses: r0adkll/upload-google-play@v1
  with:
    # ... existing settings
    whatsNewDirectory: android/release-notes  # Path to release notes
```

---

## ✅ Pre-Deployment Checklist

Before your first deployment, ensure:

- [ ] Google Play Service Account created
- [ ] Service Account JSON downloaded
- [ ] `GOOGLE_PLAY_SERVICE_ACCOUNT_JSON` secret added to GitHub
- [ ] Service Account has proper permissions in Play Console
- [ ] Internal testing track exists in Play Console
- [ ] App created in Play Console with package name `com.reactnativecicd`
- [ ] Keystore secrets already configured (`ANDROID_KEYSTORE_BASE64`, etc.)

---

## 🔒 Security Best Practices

1. **Never commit** the service account JSON to the repository
2. **Use GitHub Secrets** for all sensitive data
3. **Limit service account permissions** to only what's needed
4. **Rotate keys annually** for security
5. **Review deployments** before promoting to production

---

## 📊 Monitoring Deployments

### **Check Deployment Status:**

1. **GitHub Actions Logs:**
   - Actions tab → Select workflow run
   - Look for "Deploy to Play Store" step
   - Green checkmark = Success ✅

2. **Play Console:**
   - Release → Testing → Internal testing
   - See build version and status

### **Troubleshooting:**

- Check GitHub Actions logs for detailed errors
- Verify secrets are correctly set
- Ensure app is created in Play Console
- Check service account permissions

---

## 🎉 Success!

Once setup is complete:
- Every push to `main` automatically deploys to Play Store
- No manual uploads needed
- Consistent deployment process
- Fast feedback loop

---

**Need Help?**
- Check GitHub Actions logs for errors
- Review Play Console for app status
- Verify all secrets are set correctly

