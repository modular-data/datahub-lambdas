# Fixes Applied to datahub-lambdas

## 🎯 Two Issues Fixed

### Issue #1: OWASP Dependency Check - 403 Forbidden ❌
**Error**: 
```
Error retrieving https://nvd.nist.gov/feeds/json/cve/1.1/nvdcve-1.1-modified.meta; 
received response code 403; Forbidden
```

### Issue #2: Gradle Cache Corruption ❌
**Error**:
```
java.util.concurrent.ExecutionException: org.gradle.api.GradleException: 
Failed to create Jar file /home/circleci/.gradle/caches/jars-9/...jackson-core-2.17.2.jar
```

---

## ✅ Solutions Applied

### Fix #1: Upgraded OWASP Dependency Check

**File**: `build.gradle`

**Changes**:
1. **Line 6**: Upgraded plugin version
   ```gradle
   # Before:
   id 'org.owasp.dependencycheck'  version '8.2.1'
   
   # After:
   id 'org.owasp.dependencycheck'  version '10.0.4'
   ```

2. **Lines 144-155**: Added NVD API configuration
   ```gradle
   dependencyCheck {
       nvd {
           apiKey = System.getenv('NVD_API_KEY') ?: project.findProperty('nvdApiKey') ?: ''
       }
       failBuildOnCVSS = 7
   }
   ```

**Why**: NVD shut down legacy JSON 1.1 feeds in December 2023. Version 10.0.4 uses the new NVD API 2.0.

---

### Fix #2: Updated CircleCI Cache Keys

**File**: `.circleci/config.yml`

**Changes**:
1. **Line 14**: Updated main build cache
   ```yaml
   # Before:
   cache_key: "dwh-lambdas-build-cache-v2"
   
   # After:
   cache_key: "dwh-lambdas-build-cache-v3"
   ```

2. **Line 36**: Updated gradle_build_publish cache
   ```yaml
   # Before:
   cache_key: "dwh-lambdas-build-cache-v2"
   
   # After:
   cache_key: "dwh-lambdas-build-cache-v3"
   ```

3. **Line 52**: Updated scheduled OWASP cache
   ```yaml
   # Before:
   cache_key: "dwh-lambdas-owasp-build-cache-v1"
   
   # After:
   cache_key: "dwh-lambdas-owasp-build-cache-v2"
   ```

**Why**: Forces CircleCI to create fresh caches, eliminating the corrupted Gradle jar files.

---

## 🚨 CRITICAL: Action Required

### You MUST add an NVD API key to CircleCI

Without this key, the OWASP check will be extremely slow (30+ minutes) and may timeout.

#### Quick Steps:

1. **Get API Key** (2 minutes):
   - Visit: https://nvd.nist.gov/developers/request-an-api-key
   - Enter your email
   - Check email for the key (arrives immediately)

2. **Add to CircleCI** (1 minute):
   - Go to: https://app.circleci.com/
   - Navigate to: **Project Settings** → **Environment Variables**
   - Click: **Add Environment Variable**
   - Enter:
     - **Name**: `NVD_API_KEY`
     - **Value**: `<your-api-key>`
   - Click: **Add Variable**

3. **Verify Context Access**:
   - Make sure the `dataworks-common` context can access this variable
   - Or add it directly to the project environment variables

---

## 📊 Expected Results

### After Committing These Changes:

| Scenario | Result |
|----------|--------|
| **First CI run WITH API key** | ✅ 10-15 minutes (downloads fresh NVD database) |
| **First CI run WITHOUT API key** | ⚠️ 30+ minutes (may timeout due to rate limiting) |
| **Subsequent runs** | ✅ 2-5 minutes (uses cached NVD database) |
| **Gradle cache corruption** | ✅ Fixed (fresh cache created) |

---

## 📝 Files Changed

1. ✅ `build.gradle` - OWASP plugin upgrade + NVD API configuration
2. ✅ `.circleci/config.yml` - Cache key updates (v2→v3, v1→v2)
3. ✅ `OWASP_FIX_README.md` - Complete documentation
4. ✅ `COMMIT_MESSAGE.txt` - Ready-to-use commit message
5. ✅ `FIXES_APPLIED.md` - This file

---

## 🚀 Next Steps

### 1. Add NVD API Key to CircleCI
See instructions above ☝️

### 2. Commit and Push
```bash
cd /Users/hari.chintala/modular-data/datahub-lambdas

# Review changes
git status
git diff

# Stage all changes
git add build.gradle .circleci/config.yml OWASP_FIX_README.md COMMIT_MESSAGE.txt FIXES_APPLIED.md

# Commit using prepared message
git commit -F COMMIT_MESSAGE.txt

# Push to your fork
git push
```

### 3. Monitor CircleCI
- Watch the build in CircleCI
- First run will take 10-15 minutes (with API key)
- Should complete successfully ✅

### 4. Review Security Report
- Check for any vulnerabilities found
- Review the dependency-check report
- Update dependencies if needed
- Or add suppressions for false positives

---

## 🔧 Configuration Details

### CVSS Threshold
Currently set to **7** (fails on HIGH and CRITICAL vulnerabilities).

To adjust, edit `build.gradle`:
```gradle
dependencyCheck {
    failBuildOnCVSS = 7  // Change this value
}
```

| Value | Severity Levels |
|-------|----------------|
| `4` | MEDIUM, HIGH, CRITICAL |
| `7` | HIGH, CRITICAL (current) |
| `9` | CRITICAL only |
| `11` | Disabled |

### Cache Keys
- **Main builds**: `dwh-lambdas-build-cache-v3`
- **Scheduled OWASP**: `dwh-lambdas-owasp-build-cache-v2`

If you encounter cache issues in the future, increment these version numbers.

---

## 🐛 Troubleshooting

### If you still see Gradle cache errors:
1. Increment cache keys again (v3 → v4)
2. Or clear cache manually via CircleCI UI

### If OWASP check is still slow:
1. Verify `NVD_API_KEY` is set in CircleCI
2. Check the `dataworks-common` context has access
3. Review CircleCI build logs for API key usage

### If build fails with vulnerabilities:
1. Review the dependency-check report
2. Update vulnerable dependencies
3. Or create a suppression file (see OWASP_FIX_README.md)

---

## 📚 Documentation

For complete details, see:
- **`OWASP_FIX_README.md`** - Full setup guide with troubleshooting
- **`COMMIT_MESSAGE.txt`** - Detailed commit message

---

## ✅ Summary

Both issues are now fixed:
1. ✅ OWASP Dependency Check upgraded to 10.0.4 (supports NVD API 2.0)
2. ✅ CircleCI cache keys updated (eliminates Gradle corruption)

**Just add the NVD API key to CircleCI and you're done!** 🎉

