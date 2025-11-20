# Quran Android - Modernization Changes

## Summary

This document outlines all the changes made to modernize the Quran Android codebase in preparation for adding Firebase Authentication and Cloud Sync features.

## Changes Made

### 1. Gradle & Build System Updates

#### gradle-wrapper.properties
- **Updated Gradle:** 4.5 → 8.2

#### build.gradle (root)
- **Android Gradle Plugin:** 3.2.0-alpha07 → 8.2.0
- **Kotlin:** 1.2.21 → 1.9.20
- **Removed:** jcenter() repository (deprecated)
- **Added:** mavenCentral() repository
- **Removed:** Fabric Crashlytics plugin (deprecated)
- **Updated:** error-prone plugin 0.0.13 → 3.1.0

#### SDK Versions
- **minSdkVersion:** 14 → 21 (Android 5.0+)
- **targetSdkVersion:** 27 → 34 (Android 14)
- **compileSdkVersion:** 27 → 34
- **buildToolsVersion:** 27.0.3 → 34.0.0

### 2. AndroidX Migration

#### app/build.gradle
- **Added:** namespace declaration (required for AGP 8.0+)
- **Removed:** package attribute from AndroidManifest.xml (moved to namespace)
- **Removed:** Fabric plugin and dependencies
- **Updated test runner:** android.support.test → androidx.test

#### Dependency Updates

| Old (Support Library) | New (AndroidX) | Version |
|----------------------|----------------|---------|
| com.android.support:support-v4 | androidx.core:core-ktx | 1.6.1 |
| com.android.support:appcompat-v7 | androidx.appcompat:appcompat | 1.6.1 |
| com.android.support:recyclerview-v7 | androidx.recyclerview:recyclerview | 1.3.2 |
| com.android.support:design | com.google.android.material:material | 1.11.0 |
| com.android.support.test.espresso | androidx.test.espresso | 3.5.1 |

#### Other Dependency Updates
- **Dagger:** 2.13 → 2.48.1
- **OkHttp:** 3.9.0 → 4.12.0
- **RxJava:** 2.1.9 → 2.2.21
- **RxAndroid:** 2.0.2 → 2.1.1
- **Moshi:** 1.5.0 → 1.15.0
- **Timber:** 4.6.1 → 5.0.1
- **JUnit:** 4.12 → 4.13.2
- **Truth:** 0.36 → 1.1.5
- **Mockito:** 2.15.0 → 5.7.0
- **LeakCanary:** 1.5.1 → 2.12
- **Stetho:** 1.5.0 → 1.6.0

### 3. Module Updates

#### common/data/build.gradle
- **Updated:** `compile` → `implementation` (deprecated API)
- **Added:** Java 17 compatibility settings

#### pages/madani/build.gradle
- **Added:** kotlin-kapt plugin
- **Updated:** Dagger 2.13 → 2.48.1
- **Added:** Java 17 compatibility settings

### 4. Configuration Files

#### gradle.properties (NEW)
```properties
android.useAndroidX=true
android.enableJetifier=true
kotlin.code.style=official
org.gradle.caching=true
org.gradle.parallel=true
org.gradle.jvmargs=-Xmx2048m -XX:MaxMetaspaceSize=512m
org.gradle.configuration-cache=true
```

#### AndroidManifest.xml
- **Removed:** package attribute (moved to namespace in build.gradle)
- **Removed:** deprecated uses-sdk element

## Breaking Changes to Address

### 1. Java 17 Requirement
The project now requires JDK 17 to build. Ensure your `JAVA_HOME` points to JDK 17+.

### 2. Minimum SDK Change
The minimum SDK was bumped from 14 (Android 4.0) to 21 (Android 5.0). This:
- Drops support for devices running Android 4.x
- Affects <5% of active Android devices globally
- Enables use of modern APIs and libraries

### 3. Import Statements
All Android Support Library imports need to be migrated to AndroidX. The Jetifier tool will handle third-party libraries automatically.

**Example changes needed:**
```java
// OLD
import android.support.v7.app.AppCompatActivity;
import android.support.v4.content.ContextCompat;
import android.support.design.widget.Snackbar;

// NEW
import androidx.appcompat.app.AppCompatActivity;
import androidx.core.content.ContextCompat;
import com.google.android.material.snackbar.Snackbar;
```

### 4. Crashlytics Migration
Fabric Crashlytics has been removed. Will need to migrate to Firebase Crashlytics:
```gradle
// To be added later
implementation 'com.google.firebase:firebase-crashlytics-ktx'
```

## Next Steps

### Step 1: Build & Test Locally

1. **Sync project with Gradle files**
   ```bash
   ./gradlew clean
   ```

2. **Fix import statements**
   - Android Studio → Refactor → Migrate to AndroidX
   - Or manually update imports in code

3. **Build the project**
   ```bash
   ./gradlew assembleDebug
   ```

4. **Fix any compilation errors**
   - Most common: import statement changes
   - API changes in updated libraries
   - Deprecated API usage

5. **Run unit tests**
   ```bash
   ./gradlew test
   ```

6. **Run instrumentation tests**
   ```bash
   ./gradlew connectedAndroidTest
   ```

### Step 2: Code Migration

Expected compilation errors and fixes:

#### Error: Cannot resolve symbol 'android.support.*'
**Fix:** Use AndroidX imports instead
```java
// Change this
import android.support.annotation.NonNull;
// To this
import androidx.annotation.NonNull;
```

#### Error: OkHttp 4.x API changes
**Fix:** Update OkHttp usage
```java
// OkHttp 3.x
Request request = new Request.Builder().url(url).build();

// OkHttp 4.x (same, but Kotlin)
// Java usage remains compatible
```

#### Error: LeakCanary 2.x setup changes
**Fix:** LeakCanary 2.x auto-installs, remove manual setup
```java
// DELETE any LeakCanary.install() calls
// It now works automatically in debug builds
```

### Step 3: Firebase Integration (After Build Success)

Once the build succeeds:

1. **Add Firebase**
   - Follow FIREBASE_SETUP.md guide
   - Add google-services.json
   - Add Firebase dependencies

2. **Implement Authentication**
   - Create AuthManager
   - Add login/register UI
   - Implement Google Sign-In

3. **Implement Cloud Sync**
   - Create SyncManager
   - Update database schema
   - Implement Firestore sync logic

4. **Add Notes Feature**
   - Create notes table
   - Add notes UI
   - Include in sync

## Rollback Plan

If issues occur, you can rollback by checking out the previous commit:

```bash
git log --oneline  # Find commit before modernization
git checkout <commit-hash>
```

Or create a branch from the current state before modernization:
```bash
git branch pre-modernization <commit-hash>
```

## Estimated Build Fixes

Based on the changes:
- **Import fixes:** ~50-100 files
- **API updates:** ~10-20 locations
- **Build configuration:** Already done ✓
- **Time estimate:** 2-4 hours

## Testing Checklist

After successful build:

- [ ] App launches successfully
- [ ] Bookmarks can be created
- [ ] Bookmarks can be viewed
- [ ] Bookmarks can be deleted
- [ ] Tags can be created and assigned
- [ ] Recent pages are tracked
- [ ] Quran pages display correctly
- [ ] Translations work
- [ ] Search functions correctly
- [ ] Settings can be changed
- [ ] Import/export backup works

## Support

If you encounter issues:

1. Check Android Studio Build Output for specific errors
2. Search for "AndroidX migration [specific error]"
3. Common issues: https://developer.android.com/jetpack/androidx/migrate/class-mappings

## Ready for Firebase

Once all tests pass, the codebase will be ready for Firebase integration according to the plan in:
- FIREBASE_SETUP.md (configuration guide)
- IMPLEMENTATION_PLAN.md (detailed implementation steps)
