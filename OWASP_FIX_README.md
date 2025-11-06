# OWASP Dependency Check Fix - Quick Guide

## 🚨 Problem Fixed

Your CircleCI pipeline was failing with:
```
Error retrieving https://nvd.nist.gov/feeds/json/cve/1.1/nvdcve-1.1-modified.meta; 
received response code 403; Forbidden
```

**Root Cause**: NVD (National Vulnerability Database) shut down the legacy JSON 1.1 data feeds in December 2023. The old OWASP plugin version (8.2.1) is no longer supported.

## ✅ Solution Applied

**Upgraded OWASP Dependency Check**: `8.2.1` → `10.0.4`

This version supports the new NVD API 2.0 instead of the deprecated data feeds.

## 🔑 CRITICAL: Action Required

**You MUST add an NVD API key to CircleCI** for this to work properly.

### Step 1: Get Your NVD API Key (2 minutes)

1. Visit: **https://nvd.nist.gov/developers/request-an-api-key**
2. Enter your email address
3. Check your email - the API key arrives immediately (usually within seconds)

### Step 2: Add to CircleCI (1 minute)

1. Go to your CircleCI project: https://app.circleci.com/
2. Navigate to: **Project Settings** → **Environment Variables**
3. Click **Add Environment Variable**
4. Enter:
   - **Name**: `NVD_API_KEY`
   - **Value**: `<paste-your-api-key-here>`
5. Click **Add Variable**

**Important**: Make sure the `dataworks-common` context has access to this environment variable, or add it directly to the project.

### Step 3: Verify the Fix

1. Commit and push these changes
2. Watch the CircleCI build
3. The OWASP check should now complete successfully

## 📊 What Changed in build.gradle

### Before:
```gradle
id 'org.owasp.dependencycheck'  version '8.2.1'
```

### After:
```gradle
id 'org.owasp.dependencycheck'  version '10.0.4'

// ... (at the end of the file)

dependencyCheck {
    nvd {
        apiKey = System.getenv('NVD_API_KEY') ?: project.findProperty('nvdApiKey') ?: ''
    }
    failBuildOnCVSS = 7
}
```

## ⏱️ Performance Expectations

| Scenario | Duration |
|----------|----------|
| **First run WITH API key** | 10-15 minutes |
| **First run WITHOUT API key** | 30+ minutes (may timeout ⚠️) |
| **Subsequent runs** | 2-5 minutes (uses cache) |

**Without the API key, the check will be extremely slow and may fail due to rate limiting.**

## 🧪 Test Locally (Optional)

Before pushing to CI, you can test locally:

```bash
# Set your API key
export NVD_API_KEY="your-api-key-here"

# Update the NVD database
./gradlew dependencyCheckUpdate

# Run the security analysis
./gradlew dependencyCheckAnalyze

# View the report
open build/reports/dependency-check-report.html
```

## 🔧 Configuration Details

### CVSS Threshold
The build is configured to **fail on CVSS score ≥ 7** (HIGH and CRITICAL vulnerabilities).

To adjust this threshold, edit `build.gradle`:

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
| `11` | Disabled (no build failures) |

### Suppressing False Positives

If you encounter false positives:

1. Create a file: `dependency-check-suppressions.xml`
2. Uncomment the suppression line in `build.gradle`:
   ```gradle
   suppressionFile = 'dependency-check-suppressions.xml'
   ```
3. Add suppressions following the [official documentation](https://jeremylong.github.io/DependencyCheck/general/suppression.html)

Example suppression file:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<suppressions xmlns="https://jeremylong.github.io/DependencyCheck/dependency-suppression.1.3.xsd">
    <suppress>
        <notes><![CDATA[
        False positive - this CVE doesn't apply to our usage
        ]]></notes>
        <cve>CVE-2023-12345</cve>
    </suppress>
</suppressions>
```

## 🔍 Understanding the CircleCI Workflow

Your `.circleci/config.yml` uses the `modular-data/dataworks-orb@0.0.8` which includes:

1. **`dataworks/gradle_owasp_check`** job:
   - Runs on every commit (line 10)
   - Uses cache key: `dwh-lambdas-build-cache-v2`
   - Context: `dataworks-common`

2. **Scheduled OWASP check** (line 38-52):
   - Runs daily at 7:30 AM UK time
   - Only on `main` branch
   - Uses cache key: `dwh-lambdas-owasp-build-cache-v1`

The orb automatically:
- ✅ Runs `./gradlew dependencyCheckUpdate`
- ✅ Runs `./gradlew dependencyCheckAnalyze`
- ✅ Caches the NVD database
- ✅ Picks up the `NVD_API_KEY` environment variable

## 🐛 Troubleshooting

### Issue: Still getting 403 errors
**Solution**: Make sure you've committed and pushed the `build.gradle` changes with version 10.0.4.

### Issue: Very slow updates (30+ minutes)
**Solution**: Add the NVD API key to CircleCI environment variables.

### Issue: Build fails with vulnerabilities found
**Solution**: 
1. Review the dependency-check report
2. Update vulnerable dependencies
3. Add suppressions for false positives
4. Or adjust the CVSS threshold

### Issue: "Database is corrupt" or similar errors
**Solution**: The first run after upgrade creates a new database. If you see errors, you may need to clear the cache:
- In CircleCI, you can clear the cache by changing the `cache_key` parameter
- Or run locally: `./gradlew dependencyCheckPurge`

## 📚 Additional Resources

- [OWASP Dependency Check Documentation](https://jeremylong.github.io/DependencyCheck/)
- [NVD API Documentation](https://nvd.nist.gov/developers)
- [Gradle Plugin Documentation](https://jeremylong.github.io/DependencyCheck/dependency-check-gradle/)
- [NVD API Key Request](https://nvd.nist.gov/developers/request-an-api-key)

## ✅ Checklist

- [x] Upgraded OWASP Dependency Check to 10.0.4
- [x] Added NVD API configuration to build.gradle
- [ ] **ACTION REQUIRED**: Request NVD API key
- [ ] **ACTION REQUIRED**: Add `NVD_API_KEY` to CircleCI
- [ ] **ACTION REQUIRED**: Commit and push changes
- [ ] **ACTION REQUIRED**: Verify CI pipeline passes
- [ ] Review any vulnerabilities found in the report

## 🎯 Summary

1. **Get API Key**: https://nvd.nist.gov/developers/request-an-api-key
2. **Add to CircleCI**: Project Settings → Environment Variables → `NVD_API_KEY`
3. **Commit & Push**: The changes are ready to go
4. **Monitor**: Watch the CI build complete successfully

That's it! The fix is complete - you just need to add the API key. 🚀

