# github/workflows/build.yml
# Power Plant Checklist - builds an Android APK automatically.
# Save this file in your GitHub repository as: .github/workflows/build.yml
name: Build APK

on:
  push:
  workflow_dispatch:

permissions:
  contents: write

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '17'

      - uses: gradle/actions/setup-gradle@v4
        with:
          gradle-version: '8.7'

      - name: Create app source
        run: |
          mkdir -p app/src/main/java/com/plant/checklist app/src/main/res/drawable app/src/main/assets
          cat > settings.gradle <<'APPEOF'
          pluginManagement {
              repositories { google(); mavenCentral(); gradlePluginPortal() }
          }
          dependencyResolutionManagement {
              repositories { google(); mavenCentral() }
          }
          rootProject.name = "PowerPlantChecklist"
          include ':app'
          APPEOF
          cat > build.gradle <<'APPEOF'
          plugins {
              id 'com.android.application' version '8.5.2' apply false
          }
          APPEOF
          cat > app/build.gradle <<'APPEOF'
          plugins {
              id 'com.android.application'
          }
          android {
              namespace = 'com.plant.checklist'
              compileSdk = 34
              defaultConfig {
                  applicationId = 'com.plant.checklist'
                  minSdk = 24
                  targetSdk = 34
                  versionCode = 1
                  versionName = '1.0'
              }
              compileOptions {
                  sourceCompatibility = JavaVersion.VERSION_17
                  targetCompatibility = JavaVersion.VERSION_17
              }
          }
          APPEOF
          cat > app/src/main/AndroidManifest.xml <<'APPEOF'
          <?xml version="1.0" encoding="utf-8"?>
          <manifest xmlns:android="http://schemas.android.com/apk/res/android">
              <application
                  android:label="Plant Checklist"
                  android:icon="@drawable/ic_launcher"
                  android:theme="@android:style/Theme.Material.Light.NoActionBar">
                  <activity android:name=".MainActivity" android:exported="true"
                      android:windowSoftInputMode="adjustResize">
                      <intent-filter>
                          <action android:name="android.intent.action.MAIN" />
                          <category android:name="android.intent.category.LAUNCHER" />
                      </intent-filter>
                  </activity>
              </application>
          </manifest>
          APPEOF
          cat > app/src/main/java/com/plant/checklist/MainActivity.java <<'APPEOF'
          package com.plant.checklist;

          import android.app.Activity;
          import android.content.Intent;
          import android.os.Bundle;
          import android.webkit.JavascriptInterface;
          import android.webkit.WebChromeClient;
          import android.webkit.WebSettings;
          import android.webkit.WebView;
          import android.webkit.WebViewClient;

          public class MainAc