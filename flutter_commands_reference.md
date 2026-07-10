# Flutter Commands Reference — VS Code + Production

Quick reference for daily Flutter development, emulator management, and production release.

---

## 1. Setup & Health Check

| Command | Purpose |
|---|---|
| `flutter doctor` | Check SDK, Android toolchain, VS Code, connected devices |
| `flutter doctor -v` | Verbose output — shows exact paths and versions |
| `flutter --version` | Current Flutter/Dart version |
| `flutter upgrade` | Upgrade Flutter SDK to latest stable |
| `flutter channel stable` | Switch to stable channel |
| `flutter config --list` | Show all Flutter config values |

---

## 2. Emulator / Device Commands

### List & Launch

| Command | Purpose |
|---|---|
| `flutter devices` | List all **connected** devices (physical + running emulators) |
| `flutter emulators` | List all **available** emulators (installed AVDs) |
| `flutter emulators --launch <emulator_id>` | Launch a specific emulator, e.g. `flutter emulators --launch Pixel_7_API_34` |
| `flutter emulators --create --name my_avd` | Create a new emulator with default config |

### Android SDK level (advanced)

| Command | Purpose |
|---|---|
| `emulator -list-avds` | List AVDs directly via Android SDK |
| `emulator -avd Pixel_7_API_34` | Launch AVD directly (faster, more flags available) |
| `emulator -avd Pixel_7_API_34 -no-snapshot-load` | Cold boot (fixes stuck/frozen emulator) |
| `adb devices` | List devices visible to ADB |
| `adb kill-server && adb start-server` | Restart ADB when device shows `offline`/`unauthorized` |
| `adb reverse tcp:8000 tcp:8000` | Forward emulator → localhost API (useful for local Laravel/FastAPI backend) |

> **Tip:** Emulator can't reach `localhost` of your machine directly — use `10.0.2.2` instead of `127.0.0.1` in API base URLs, or use `adb reverse`.

---

## 3. Running the App (Debug / Profile / Release)

| Command | Mode | Use Case |
|---|---|---|
| `flutter run` | Debug | Default. Hot reload enabled, assertions on, slow perf |
| `flutter run -d <device_id>` | Debug | Run on specific device (`flutter devices` to get ID) |
| `flutter run --profile` | Profile | Performance analysis — DevTools timeline, near-release perf |
| `flutter run --release` | Release | Full optimization, no hot reload — final perf testing |
| `flutter run --flavor prod -t lib/main_prod.dart` | Any | Run a specific flavor + entrypoint |
| `flutter run --dart-define=API_URL=https://api.example.com` | Any | Pass compile-time environment variables |

### While `flutter run` is active (terminal keys)

| Key | Action |
|---|---|
| `r` | Hot reload |
| `R` | Hot restart (resets state) |
| `p` | Toggle debug paint (layout boundaries) |
| `o` | Toggle platform (Android ↔ iOS rendering) |
| `v` | Open DevTools in browser |
| `q` | Quit |

---

## 4. VS Code Specific (Flutter/Dart extension)

### Shortcuts

| Shortcut | Action |
|---|---|
| `F5` | Start debugging (debug mode) |
| `Ctrl+F5` | Run without debugging |
| `Shift+F5` | Stop |
| `Ctrl+Shift+F5` | Hot restart |
| `Ctrl+S` | Save → triggers hot reload automatically (if enabled) |
| `Ctrl+Shift+P` | Command Palette |

### Useful Command Palette commands (`Ctrl+Shift+P`)

- `Flutter: Select Device` — switch target device/emulator
- `Flutter: Launch Emulator` — pick and start an emulator from list
- `Flutter: New Project`
- `Flutter: Hot Reload` / `Flutter: Hot Restart`
- `Flutter: Open DevTools` — widget inspector, memory, network
- `Flutter: Run Flutter Doctor`
- `Flutter: Change SDK` — switch between multiple Flutter SDKs
- `Dart: Restart Analysis Server` — fixes stale analyzer errors

### `launch.json` — run configurations (debug/profile/release + flavors)

```jsonc
// .vscode/launch.json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Debug",
      "request": "launch",
      "type": "dart"
    },
    {
      "name": "Profile",
      "request": "launch",
      "type": "dart",
      "flutterMode": "profile"
    },
    {
      "name": "Release",
      "request": "launch",
      "type": "dart",
      "flutterMode": "release"
    },
    {
      "name": "Prod Flavor",
      "request": "launch",
      "type": "dart",
      "program": "lib/main_prod.dart",
      "args": ["--flavor", "prod", "--dart-define=ENV=production"]
    }
  ]
}
```

---

## 5. Dependencies & Codegen

| Command | Purpose |
|---|---|
| `flutter pub get` | Install dependencies from `pubspec.yaml` |
| `flutter pub upgrade` | Upgrade deps within constraints |
| `flutter pub upgrade --major-versions` | Upgrade including breaking versions |
| `flutter pub outdated` | Show which packages have newer versions |
| `flutter pub add <package>` | Add a dependency |
| `flutter pub remove <package>` | Remove a dependency |
| `dart run build_runner build --delete-conflicting-outputs` | Run code generation (json_serializable, freezed, etc.) |
| `dart run build_runner watch --delete-conflicting-outputs` | Codegen in watch mode |

---

## 6. Cleaning & Troubleshooting

| Command | Purpose |
|---|---|
| `flutter clean` | Delete `build/` and `.dart_tool/` — fixes most weird build errors |
| `flutter clean && flutter pub get` | Standard reset combo |
| `cd android && ./gradlew clean` | Deep clean Gradle cache (Windows: `gradlew clean`) |
| `flutter pub cache repair` | Repair corrupted pub cache |
| `adb uninstall com.example.app` | Remove old install when signature conflicts occur |
| `flutter analyze` | Static analysis / lint check |
| `flutter test` | Run unit/widget tests |

---

## 7. Production Builds

### Android — APK

| Command | Output |
|---|---|
| `flutter build apk --release` | Single fat APK (`build/app/outputs/flutter-apk/app-release.apk`) |
| `flutter build apk --release --split-per-abi` | Separate APKs per ABI (arm64-v8a, armeabi-v7a, x86_64) — smaller size |
| `flutter build apk --debug` | Debug APK for internal testing |

### Android — App Bundle (Play Store — required)

```bash
flutter build appbundle --release
# Output: build/app/outputs/bundle/release/app-release.aab
```

With flavor + obfuscation (recommended for production):

```bash
flutter build appbundle --release \
  --flavor prod -t lib/main_prod.dart \
  --obfuscate --split-debug-info=build/symbols \
  --dart-define=ENV=production
```

> **Keep `build/symbols`** — you need it to de-obfuscate crash stack traces (`flutter symbolize`), and upload native debug symbols to Play Console.

### Android — Signing setup (one-time)

```bash
# 1. Generate keystore
keytool -genkey -v -keystore upload-keystore.jks \
  -keyalg RSA -keysize 2048 -validity 10000 -alias upload
```

```properties
# 2. android/key.properties (never commit this)
storePassword=YOUR_PASSWORD
keyPassword=YOUR_PASSWORD
keyAlias=upload
storeFile=../upload-keystore.jks
```

```groovy
// 3. android/app/build.gradle
def keystoreProperties = new Properties()
def keystorePropertiesFile = rootProject.file('key.properties')
if (keystorePropertiesFile.exists()) {
    keystoreProperties.load(new FileInputStream(keystorePropertiesFile))
}

android {
    signingConfigs {
        release {
            keyAlias keystoreProperties['keyAlias']
            keyPassword keystoreProperties['keyPassword']
            storeFile keystoreProperties['storeFile'] ? file(keystoreProperties['storeFile']) : null
            storePassword keystoreProperties['storePassword']
        }
    }
    buildTypes {
        release {
            signingConfig signingConfigs.release
        }
    }
}
```

### iOS (macOS only)

| Command | Purpose |
|---|---|
| `flutter build ios --release` | Build iOS release (requires Xcode + signing) |
| `flutter build ipa --release` | Build `.ipa` for App Store / TestFlight upload |
| `flutter build ipa --release --obfuscate --split-debug-info=build/symbols` | With obfuscation |
| `cd ios && pod install --repo-update` | Fix CocoaPods dependency issues |

### Web / Desktop (if needed)

| Command | Purpose |
|---|---|
| `flutter build web --release` | Web build → `build/web/` |
| `flutter build web --release --web-renderer canvaskit` | Better fidelity, larger bundle |
| `flutter build windows --release` | Windows desktop |

---

## 8. Version Bumping Before Release

```yaml
# pubspec.yaml
version: 1.4.2+18   # versionName + versionCode
```

- Increment `+18` (build number / versionCode) on **every** Play Store upload
- Increment `1.4.2` (versionName) for user-visible releases

Override at build time without editing pubspec:

```bash
flutter build appbundle --release --build-name=1.4.2 --build-number=18
```

---

## 9. Typical Production Release Checklist

```bash
flutter clean
flutter pub get
flutter analyze                      # zero errors
flutter test                         # tests pass
flutter build appbundle --release \
  --obfuscate --split-debug-info=build/symbols
# Upload .aab to Play Console → upload symbols → staged rollout
```
