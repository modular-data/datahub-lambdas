# Next Steps - Quick Reference

## 🎯 What's Been Fixed

✅ **datahub-lambdas** - OWASP upgraded to 10.0.4, cache keys updated
✅ **dataworks-orb** - Gradle cache cleanup added to ALL Gradle jobs (run_with_cache + gradle_owasp_check)

---

## 📋 What You Need to Do (5 Steps)

### Step 1: Get NVD API Key (2 minutes)
```bash
# Visit this URL and enter your email:
https://nvd.nist.gov/developers/request-an-api-key

# Check your email for the API key (arrives immediately)
```

### Step 2: Add API Key to CircleCI (1 minute)
1. Go to: https://app.circleci.com/
2. Navigate to your project: **datahub-lambdas**
3. Click: **Project Settings** → **Environment Variables**
4. Click: **Add Environment Variable**
5. Enter:
   - Name: `NVD_API_KEY`
   - Value: `<paste-your-api-key>`

### Step 3: Commit datahub-lambdas Changes
```bash
cd /Users/hari.chintala/modular-data/datahub-lambdas

# Review what changed
git status
git diff build.gradle .circleci/config.yml

# Commit
git add build.gradle .circleci/config.yml
git commit -F COMMIT_MESSAGE.txt

# Push
git push
```

### Step 4: Commit dataworks-orb Changes
```bash
cd /Users/hari.chintala/repos/dataworks-orb

# Review what changed
git status
git diff src/commands/run_with_cache.yml src/jobs/gradle_owasp_check.yml

# Commit
git add src/commands/run_with_cache.yml src/jobs/gradle_owasp_check.yml
git commit -F ORB_COMMIT_MESSAGE.txt

# Push
git push
```

### Step 5: Publish New Orb Version & Update datahub-lambdas
```bash
# Publish new orb version (follow your standard process)
# Then update datahub-lambdas to use the new version

cd /Users/hari.chintala/modular-data/datahub-lambdas

# Edit .circleci/config.yml and update orb version:
# orbs:
#   dataworks: modular-data/dataworks-orb@0.0.9  # <-- increment version

git add .circleci/config.yml
git commit -m "chore: Update dataworks-orb to latest version"
git push
```

---

## 🎉 Done!

After completing these steps:
- ✅ OWASP 403 errors will be fixed
- ✅ Gradle cache corruption errors will be fixed
- ✅ CircleCI builds will pass
- ✅ Security vulnerability scanning will work

---

## 📊 Expected Timeline

| Step | Time | Status |
|------|------|--------|
| Get NVD API key | 2 min | ⏳ Pending |
| Add to CircleCI | 1 min | ⏳ Pending |
| Commit datahub-lambdas | 2 min | ⏳ Pending |
| Commit dataworks-orb | 2 min | ⏳ Pending |
| Publish orb & update | 5 min | ⏳ Pending |
| **Total** | **~12 min** | |

---

## 📚 Documentation

For detailed information, see:
- **COMPLETE_FIX_SUMMARY.md** - Comprehensive overview of all fixes
- **OWASP_FIX_README.md** - Detailed OWASP setup and troubleshooting
- **COMMIT_MESSAGE.txt** - Commit message for datahub-lambdas
- **../dataworks-orb/ORB_COMMIT_MESSAGE.txt** - Commit message for orb

---

## 🆘 Need Help?

If builds still fail after these steps:
1. Check CircleCI logs for the specific error
2. Verify `NVD_API_KEY` is set correctly
3. Verify new orb version is being used
4. See troubleshooting section in COMPLETE_FIX_SUMMARY.md

