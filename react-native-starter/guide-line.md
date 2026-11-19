# React Native Setup

A systematic, end-to-end guide for setting up a **bare React Native** project with the
React Native Community CLI, Android Studio, and Xcode.

The versions below target **React Native 0.88.x** (current stable as of November 2026).
If you use a newer release, check the official
[environment setup guide](https://reactnative.dev/docs/set-up-your-environment) for
updated SDK requirements.

---

## Table of Contents

- [1. Overview](#1-overview)
  - [1.1 Requirements](#11-requirements)
  - [1.2 Setup](#12-setup)
    - [1.2.1 Install Node.js](#121-install-nodejs)
    - [1.2.2 Install JDK 17](#122-install-jdk-17)
    - [1.2.3 Set up the Android environment](#123-set-up-the-android-environment)
    - [1.2.4 Set up the iOS environment (macOS only)](#124-set-up-the-ios-environment-macos-only)
    - [1.2.5 Install Watchman (macOS / Linux)](#125-install-watchman-macos--linux)
    - [1.2.6 Create the project](#126-create-the-project)
    - [1.2.7 Install dependencies](#127-install-dependencies)
    - [1.2.8 Verify the toolchain](#128-verify-the-toolchain)
    - [1.2.9 Run the app](#129-run-the-app)
    - [1.2.10 First-run configuration](#1210-first-run-configuration)
    - [1.2.11 Troubleshooting](#1211-troubleshooting)
    - [1.2.12 Clean rebuild](#1212-clean-rebuild)
- [1.3 Next steps](#13-next-steps)

---

## 1. Overview

This document covers the complete local setup for a React Native application that owns
its native projects (`android/` and `ios/`). It is intentionally written for the
**Community CLI workflow** rather than a managed framework, so every native dependency,
signing configuration, and build setting is available to you.

### What you are building

A new React Native app consists of three cooperating pieces:

| Layer | Responsibility |
| --- | --- |
| **JavaScript app** (`App.tsx`, `src/`) | UI, business logic, state, navigation |
| **Metro** | The JavaScript bundler and development server (port `8081`) |
| **Native shells** (`android/`, `ios/`) | App container, permissions, native modules, release builds |

React Native 0.88 uses the **New Architecture** by default and ships with **Hermes** as
the JavaScript engine. You do not need to enable either manually.

### Host and target support

| Host OS | Android target | iOS target |
| --- | --- | --- |
| Windows | Yes | No (macOS required; use a physical device with Expo Go if needed) |
| Linux | Yes | No (macOS required) |
| macOS | Yes | Yes |

### Time and resource expectations

- **Fresh machine:** about 60–90 minutes (downloads dominate).
- **Machine already running Android Studio or Xcode:** 10–15 minutes.
- **Disk:** at least 20 GB free; 40 GB+ recommended with Xcode and emulators.
- **RAM:** 16 GB recommended; 8 GB is the practical minimum.
- **CPU virtualization:** must be enabled in BIOS/UEFI for Android emulators.

### Alternative: Expo

If you do not need to maintain native projects yourself, Expo is a faster path and
removes the need to install Android Studio or Xcode for day-to-day development. This
guide does not cover it; see [Expo's documentation](https://docs.expo.dev/). The rest of
this document assumes the bare Community CLI workflow.

---

## 1.1 Requirements

### Required software

| Tool | Version | Purpose |
| --- | --- | --- |
| **Node.js** | 22.11.0 or newer (LTS) | Metro, Community CLI, npm packages |
| **npm** | Bundled with Node.js | Dependency management |
| **JDK** | 17 | Gradle and Android builds |
| **Android Studio** | Latest stable | Android SDK, AVD Manager, emulator |
| **Android SDK Platform** | API 35 (Android 15, "VanillaIceCream") | Compile SDK for RN 0.88 |
| **Android SDK Build-Tools** | 36.0.0 | Android build pipeline |
| **Android SDK Command-line Tools** | Latest | `sdkmanager`, `avdmanager`, licenses |
| **Android SDK Platform-Tools** | Latest | `adb` |
| **Xcode** | Latest stable (macOS only) | iOS build toolchain and simulator |
| **CocoaPods** | Latest, via the project `Gemfile` (macOS only) | iOS native dependencies |
| **Ruby** | 3.x recommended (macOS only) | CocoaPods runtime |
| **Watchman** | Latest (macOS / Linux) | Efficient file watching |
| **Git** | Latest | Source control |

Additional requirements:

- **Android:** an emulator (AVD) or a physical device with Developer Options and USB
  debugging enabled.
- **iOS:** a Mac, an iOS Simulator runtime installed through Xcode, and (for physical
  devices) an Apple Developer account for code signing.

> **Note on JDK versions:** React Native validates JDK 17. Newer JDKs may work, but they
> are not the supported configuration and commonly break Gradle. If you already have a
> newer JDK, install JDK 17 and point `JAVA_HOME` at it for this project.

> **Note on Node versions:** avoid odd-numbered/current releases. Use the latest Node
> LTS (22.11+). If you switch Node versions often, use `fnm`, `nvm`, or `nvm-windows`.

### Verify the baseline

Run these commands after installation. Every command should report a version:

```bash
node -v        # v22.11.0 or newer
npm -v
java -version  # 17.x
git --version
```

---

## 1.2 Setup

The steps are ordered so that each layer can be verified before the next one depends on
it. Follow them in sequence and do not skip the verification commands.

### 1.2.1 Install Node.js

#### Windows

Use one of the following:

```powershell
# Option A: winget
winget install OpenJS.NodeJS.LTS

# Option B: Chocolatey (run as Administrator)
choco install -y nodejs-lts
```

For multiple Node versions, install
[nvm-windows](https://github.com/coreybutler/nvm-windows):

```powershell
nvm install lts
nvm use lts
```

#### macOS

```bash
brew install node
# or, for version management:
brew install fnm
fnm install --lts
```

#### Linux

Use your distribution's package manager, NodeSource, or a version manager:

```bash
# Example with fnm
curl -fsSL https://fnm.vercel.app/install | bash
fnm install --lts
```

Verify:

```bash
node -v
npm -v
```

### 1.2.2 Install JDK 17

#### Windows

```powershell
winget install EclipseAdoptium.Temurin.17.JDK
# or
choco install -y microsoft-openjdk17
```

Then set `JAVA_HOME` for your user account and open a **new** terminal:

```powershell
$jdk = "C:\Program Files\Eclipse Adoptium\jdk-17.0.13.11-hotspot"  # adjust to your install
[Environment]::SetEnvironmentVariable("JAVA_HOME", $jdk, "User")

$userPath = [Environment]::GetEnvironmentVariable("Path", "User")
if ($userPath -notlike "*$jdk\bin*") {
  [Environment]::SetEnvironmentVariable("Path", "$userPath;$jdk\bin", "User")
}
```

You can also use the JDK bundled with Android Studio, provided it is version 17:

```text
C:\Program Files\Android\Android Studio\jbr
```

Do **not** point `JAVA_HOME` at a JRE. Verify in a new terminal:

```powershell
java -version
echo $env:JAVA_HOME
```

#### macOS

```bash
brew install --cask temurin@17
export JAVA_HOME=$(/usr/libexec/java_home -v 17)
```

Add the `export` line to `~/.zshrc` to make it persistent.

#### Linux

```bash
sudo apt install openjdk-17-jdk
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
export PATH="$JAVA_HOME/bin:$PATH"
```

Add both exports to `~/.bashrc` or `~/.zshrc`.

### 1.2.3 Set up the Android environment

#### 1. Install Android Studio

Download it from [developer.android.com/studio](https://developer.android.com/studio) and
run the installer. Keep these components selected:

- Android SDK
- Android SDK Platform
- Android Virtual Device

#### 2. Install the required SDK packages

Open **Android Studio → More Actions → SDK Manager** (or **Settings → Languages &
Frameworks → Android SDK**).

Under **SDK Platforms**, enable **Show Package Details**, expand
**Android 15 (VanillaIceCream)**, and select:

- `Android SDK Platform 35`
- `Google APIs Intel x86_64 Atom System Image` (Intel/AMD hosts)
- `Google APIs ARM 64 v8a System Image` (Apple Silicon)

Under **SDK Tools**, enable **Show Package Details**, and select:

- `Android SDK Build-Tools 36.0.0`
- `Android SDK Command-line Tools (latest)`
- `Android SDK Platform-Tools`
- `Android Emulator`

Click **Apply** and accept the licenses.

You can also accept licenses from a terminal:

```bash
sdkmanager --licenses
```

#### 3. Configure environment variables

##### Windows

The default SDK location is:

```text
%LOCALAPPDATA%\Android\Sdk
```

Set the variables persistently with PowerShell (then open a new terminal):

```powershell
$sdk = "$env:LOCALAPPDATA\Android\Sdk"
[Environment]::SetEnvironmentVariable("ANDROID_HOME", $sdk, "User")

$userPath = [Environment]::GetEnvironmentVariable("Path", "User")
$additions = @(
  "$sdk\platform-tools",
  "$sdk\emulator",
  "$sdk\cmdline-tools\latest\bin"
) | Where-Object { $userPath -notlike "*$_*" }

if ($additions) {
  [Environment]::SetEnvironmentVariable(
    "Path",
    ($userPath + ";" + ($additions -join ";")),
    "User"
  )
}
```

Alternatively, set them through **Control Panel → User Accounts → Change my
environment variables**.

> Avoid `setx PATH "%PATH%;..."`: `setx` truncates values longer than 1024 characters
> and does not update the current terminal.

Verify in a new terminal:

```powershell
echo $env:ANDROID_HOME
adb --version
emulator -list-avds
```

##### macOS

```bash
# ~/.zshrc
export ANDROID_HOME="$HOME/Library/Android/sdk"
export PATH="$ANDROID_HOME/emulator:$ANDROID_HOME/platform-tools:$ANDROID_HOME/cmdline-tools/latest/bin:$PATH"
```

Reload the shell:

```bash
source ~/.zshrc
```

##### Linux

```bash
# ~/.bashrc or ~/.zshrc — verify the path with Android Studio if it differs
export ANDROID_HOME="$HOME/Android/Sdk"
export PATH="$ANDROID_HOME/emulator:$ANDROID_HOME/platform-tools:$ANDROID_HOME/cmdline-tools/latest/bin:$PATH"
```

#### 4. Create an Android Virtual Device (AVD)

**Using Android Studio:** open **Device Manager → Create Virtual Device**, choose a Pixel
profile, select the **API 35** system image, and finish. Start the emulator with the ▶
button.

**Using the command line:**

```bash
# Choose the image matching your CPU: x86_64 (Intel/AMD) or arm64-v8a (Apple Silicon)
sdkmanager "system-images;android-35;google_apis;x86_64"
avdmanager create avd -n Pixel_API_35 -k "system-images;android-35;google_apis;x86_64" -d pixel_7
emulator -avd Pixel_API_35
```

> On Windows, enable virtualization in BIOS/UEFI and turn on **Windows Hypervisor
> Platform** (or Hyper-V) for acceptable emulator performance. Intel HAXM is deprecated.

#### 5. Prepare a physical Android device (optional)

1. Open **Settings → About phone** and tap **Build number** seven times.
2. Open **Developer options** and enable **USB debugging**.
3. Connect the device by USB and accept the RSA fingerprint prompt.
4. Verify:

```bash
adb devices
```

The device must appear with the status `device` (not `unauthorized`).

### 1.2.4 Set up the iOS environment (macOS only)

Skip this section on Windows or Linux; native iOS builds require macOS.

#### 1. Install Xcode

Install the latest Xcode from the Mac App Store, then launch it once and accept the
license agreement.

#### 2. Install the Xcode Command Line Tools

```bash
sudo xcodebuild -license accept
sudo xcodebuild -runFirstLaunch
xcode-select -p   # should point to /Applications/Xcode.app/Contents/Developer
```

If it points elsewhere, select the full Xcode installation:

```bash
sudo xcode-select --switch /Applications/Xcode.app/Contents/Developer
```

#### 3. Install an iOS Simulator runtime

Open **Xcode → Settings → Platforms**, click **+**, and install the latest iOS simulator
runtime.

#### 4. Install Ruby and CocoaPods

React Native projects include a `Gemfile` so that CocoaPods versions stay reproducible
across machines. Use a modern Ruby rather than the system Ruby:

```bash
brew install rbenv ruby-build
rbenv install 3.3.6        # or any current 3.3.x/3.4.x release
rbenv global 3.3.6
ruby -v                    # must be 3.x
```

You do not need a global CocoaPods install: `bundle exec pod install` uses the version
pinned by the project. If you prefer a global install:

```bash
gem install cocoapods
pod --version
```

Verify Xcode:

```bash
xcodebuild -version
```

CocoaPods is installed per project by `bundle install` in
[1.2.7](#127-install-dependencies); you can confirm the resolved version from the `ios/`
directory with `bundle exec pod --version`.

#### 5. Xcode environment for Node

The template contains `ios/.xcode.env`, which tells Xcode where to find Node. Do not
edit that tracked file; create `ios/.xcode.env.local` instead (it is git-ignored). If
Xcode builds fail with `node: command not found` (common with `nvm`/`fnm`), set the
absolute path explicitly:

```bash
# ios/.xcode.env.local
export NODE_BINARY=/Users/<you>/.nvm/versions/node/v22.11.0/bin/node
```

### 1.2.5 Install Watchman (macOS / Linux)

Watchman improves file watching and avoids stale-bundle issues. It is not needed on
Windows.

```bash
# macOS
brew install watchman

# Linux (package name may vary)
sudo apt install watchman
```

Verify:

```bash
watchman --version
```

### 1.2.6 Create the project

Remove any legacy global CLI first; it conflicts with the current tooling:

```bash
npm uninstall -g react-native-cli @react-native-community/cli
```

Generate a new application:

```bash
npx @react-native-community/cli@latest init AwesomeProject
cd AwesomeProject
```

Useful options:

```bash
# Pin the React Native version
npx @react-native-community/cli@latest init AwesomeProject --version 0.88.0

# Choose a package manager
npx @react-native-community/cli@latest init AwesomeProject --pm npm
```

> If you cloned this starter repository instead of generating a new project, confirm that
> `package.json` exists and skip to [1.2.7](#127-install-dependencies).

The generated project already includes TypeScript, ESLint, Prettier, Jest, the Android
project, and the iOS project.

### 1.2.7 Install dependencies

```bash
npm install
```

Install iOS pods **on macOS only**:

```bash
cd ios
bundle install
bundle exec pod install
cd ..
```

The first pod install can take several minutes. Do not run `pod install` on Windows or
Linux.

### 1.2.8 Verify the toolchain

Run the built-in environment doctor:

```bash
npx @react-native-community/cli doctor
```

It checks Node, JDK, Android SDK, `adb`, emulator images, Xcode, CocoaPods, and Watchman.
Fix every reported error before continuing. Warnings about optional tools are usually
safe to ignore.

Additional checks:

```bash
adb devices                    # Android emulator/device must be listed
emulator -list-avds            # AVDs must exist
xcodebuild -version            # macOS only
```

### 1.2.9 Run the app

Start Metro in the first terminal:

```bash
npm start

# If you changed native dependencies or see a stale bundle:
npm start -- --reset-cache
```

Keep Metro running. In a second terminal, build and install the app:

```bash
# Android emulator or connected device
npm run android

# iOS Simulator (macOS only)
npm run ios

# Target a specific simulator
npm run ios -- --simulator "iPhone 16"
```

The first native build is slow; later builds are incremental.

#### Metro and developer shortcuts

| Action | Shortcut |
| --- | --- |
| Open the developer menu | `Ctrl+M` (Windows/Linux), `Cmd+M` (macOS Android), `Cmd+D` (iOS Simulator) |
| Reload the app | Press `r` in Metro, or `R` twice in the app |
| Open React Native DevTools | Press `j` in Metro |
| Show all Metro commands | Press `?` in Metro |

On a physical Android device, make sure Metro is reachable over USB:

```bash
adb reverse tcp:8081 tcp:8081
```

#### Success criteria

- Metro reports `Bundling 100%` with no red error screen.
- The app displays the React Native welcome screen.
- Editing `App.tsx` triggers Fast Refresh on save.
- `npm run lint` completes without errors.

### 1.2.10 First-run configuration

These steps are recommended before building features or committing the starter.

#### Set the application identity

- **Android package / application ID:** `android/app/build.gradle`
  (`namespace` and `defaultConfig.applicationId`). Use reverse-DNS form, e.g.
  `com.company.appname`.
- **Android display name:** `android/app/src/main/res/values/strings.xml`.
- **iOS bundle identifier and display name:** open `ios/AwesomeProject.xcworkspace` in
  Xcode, select the app target, and edit **General → Identity**.
- After changing the Android package, a clean rebuild is required.

#### Pin the toolchain

Create `.nvmrc` at the project root:

```text
22
```

Add an `engines` block to `package.json`:

```json
{
  "engines": {
    "node": ">=22.11.0"
  }
}
```

#### Configure version control

The template includes a `.gitignore`. Confirm it excludes `node_modules/`, `ios/Pods/`,
`ios/build/`, `android/.gradle/`, `android/app/build/`, and local signing files. Never
commit keystores, `google-services.json`, `GoogleService-Info.plist`, or `.env` files
containing secrets.

#### Run the quality gates

```bash
npm run lint
npm test
```

### 1.2.11 Troubleshooting

| Symptom | Likely cause | Fix |
| --- | --- | --- |
| `'java' is not recognized` or Gradle reports an unsupported Java version | `JAVA_HOME` is unset or points at a JRE/newer JDK | Install JDK 17, set `JAVA_HOME`, open a new terminal, re-run `java -version` |
| `SDK location not found` | `ANDROID_HOME` missing or SDK installed elsewhere | Set `ANDROID_HOME`, restart the terminal, or add `sdk.dir=<path>` to `android/local.properties` |
| `adb: no devices/emulators found` | Emulator not running, USB debugging off, or driver issue | Start the AVD, enable USB debugging, run `adb kill-server && adb start-server`, re-check `adb devices` |
| Emulator fails to start or is extremely slow | Virtualization disabled | Enable VT-x/AMD-V in BIOS/UEFI and Windows Hypervisor Platform; use an x86_64/arm64 image matching the host |
| `Unable to load script` from Metro | Metro is not running or the device cannot reach it | Start `npm start`, verify port `8081`, run `adb reverse tcp:8081 tcp:8081` |
| Port `8081` already in use | Another Metro instance or service | Find the process with `netstat -ano` (Windows) or `lsof -i :8081` (macOS/Linux), stop it, then restart Metro |
| Changes do not appear | Stale Metro or Watchman state | `npm start -- --reset-cache`; macOS: `watchman watch-del-all` |
| Android build fails after an SDK update | Partial SDK install or stale Gradle cache | Reinstall Build-Tools 36.0.0 and Platform 35, then run `cd android && ./gradlew clean` |
| `pod install` fails | Outdated CocoaPods spec repo or wrong Xcode CLT | `bundle exec pod install --repo-update`; verify `xcode-select -p`; delete `Pods/` and `Podfile.lock` and retry |
| Xcode build error containing `node: command not found` | Xcode cannot see the Node binary from `nvm`/`fnm` | Set `NODE_BINARY` in `ios/.xcode.env` to the absolute Node path |
| Old app version still launches after reinstall | Stale app data or duplicate install | `adb uninstall com.company.appname`, then rebuild |
| Windows build fails with "path too long" | Long Metro/Gradle paths | Enable long paths: `git config --system core.longpaths true` and the Windows `LongPathsEnabled` policy; move the project closer to the drive root |
| Gradle runs out of memory | Default heap too small | Set `org.gradle.jvmargs=-Xmx4g -XX:MaxMetaspaceSize=1g` in `android/gradle.properties` |

Logs:

```bash
# Android native logs
adb logcat

# React Native logs and Metro output
npm start
```

### 1.2.12 Clean rebuild

Use this when builds fail for unclear reasons, after switching branches, or after changing
SDK versions or package identifiers. Stop Metro and any running app first.

**Android (Windows PowerShell):**

```powershell
cd android
.\gradlew clean
cd ..
```

**Android (macOS/Linux):**

```bash
cd android
./gradlew clean
cd ..
```

**iOS (macOS only):**

```bash
cd ios
rm -rf Pods Podfile.lock build
bundle install
bundle exec pod install
cd ..
```

**Metro and Watchman caches:**

```bash
npm start -- --reset-cache   # start Metro with a clean transform cache
watchman watch-del-all       # macOS/Linux, when Watchman is installed
```

If the failure persists, delete `node_modules/` and reinstall:

```bash
rm -rf node_modules   # Windows: Remove-Item -Recurse -Force node_modules
npm install
```

---

## 1.3 Next steps

Once the welcome screen runs reliably on your target platform, continue with:

- **Project structure:** move screens and components into `src/` and keep `App.tsx` as the
  root composition point.
- **Navigation:** add [React Navigation](https://reactnavigation.org/) for stacks, tabs,
  and deep links.
- **Data and state:** pair a server-state library such as
  [TanStack Query](https://tanstack.com/query) with a lightweight store such as
  [Zustand](https://zustand.docs.pmnd.rs/) when needed.
- **Native builds:** read
  [Running on Device](https://reactnative.dev/docs/running-on-device) and
  [Publishing to Google Play](https://reactnative.dev/docs/signed-apk-android).
- **CI/CD:** consider [EAS Build](https://docs.expo.dev/build/introduction/) or a
  Gradle/Xcode pipeline for signed artifacts.
- **Debugging and testing:** use React Native DevTools (`j` in Metro),
  [Jest](https://jestjs.io/), and
  [React Native Testing Library](https://callstack.github.io/react-native-testing-library/).

### Reference documentation

- [React Native: Set Up Your Environment](https://reactnative.dev/docs/set-up-your-environment)
- [React Native: Get Started Without a Framework](https://reactnative.dev/docs/getting-started-without-a-framework)
- [React Native: Troubleshooting](https://reactnative.dev/docs/troubleshooting)
- [React Native Community CLI](https://github.com/react-native-community/cli)
- [Android Studio and SDK](https://developer.android.com/studio)
- [CocoaPods](https://guides.cocoapods.org/using/getting-started.html)
- [Watchman](https://facebook.github.io/watchman/docs/install)
