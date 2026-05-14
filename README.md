# 🌱 Rooted — Plant Tracker

Rooted is a **local-first mobile app** for tracking your plants over time.
Log events, take photos, monitor growth stages, and keep a visual journal of your garden — all stored privately on your device.

---

## ✨ Features

- **Plant journal** — create plants with name, variety, location, and a reference photo
- **Event timeline** — log watering, fertilising, repotting, problems, and custom notes
- **Photo gallery** — attach photos to events and set a cover photo per plant
- **Growth stages** — profile-aware stage stepper tailored to each plant type
- **Quick water** — one-tap 💧 action from both the list and the detail page
- **AI analysis** *(optional)* — photo-based stage assessment, health check, and growing guide powered by OpenAI
- **Plant lookup** — identify an unknown plant from a photo or by name
- **Backup & restore** — full snapshot export and safe atomic restore via the OS share sheet
- **Fully offline** — no account, no cloud, no tracking

---

## 🗂 Project Structure

```
Rooted/
├── src/
│   ├── Rooted.Domain/          # Core entities, enums, business rules
│   ├── Rooted.Application/     # Use cases, interfaces, DTOs
│   ├── Rooted.Infrastructure/  # SQLite, file storage, backup service
│   └── Rooted.Mobile/          # .NET MAUI Android app (views, viewmodels, DI)
├── AGENTS.md                   # Architecture & conventions reference
├── Rooted.slnx                 # Solution file
└── README.md
```

### Architecture

Rooted follows **Clean Architecture** with strict layer separation:

| Layer | Responsibility |
|---|---|
| `Domain` | Entities (`Plant`, `PlantEvent`, `PlantPhoto`), enums |
| `Application` | Use cases, repository interfaces, service interfaces |
| `Infrastructure` | SQLite via sqlite-net, local file storage, backup/restore |
| `Mobile` | MAUI XAML views, MVVM viewmodels, dependency injection wiring |

---

## 🛠 Prerequisites

| Tool | Version |
|---|---|
| [.NET SDK](https://dotnet.microsoft.com/download) | 10.0 or later |
| [.NET MAUI workload](https://learn.microsoft.com/en-us/dotnet/maui/get-started/installation) | Bundled with .NET 10 |
| Android SDK | API 21+ (Android 5.0 Lollipop) |
| Visual Studio 2022 | 17.12+ with **Mobile development with .NET** workload, **or** VS Code with C# Dev Kit |
| Java / JDK | 11 or 17 (bundled with Android SDK) |

### Install the MAUI workload

```bash
dotnet workload install maui-android
```

---

## 🚀 Running the App

### Option A — Visual Studio

1. Open `Rooted.slnx`
2. Set **Rooted.Mobile** as the startup project
3. Select an Android emulator or connected device from the toolbar
4. Press **F5** (Debug) or **Ctrl+F5** (Run without debugging)

### Option B — CLI (debug on connected device / emulator)

```bash
# Restore packages
dotnet restore src/Rooted.Mobile/Rooted.Mobile.csproj

# Run on the first connected Android device / running emulator
dotnet build src/Rooted.Mobile/Rooted.Mobile.csproj \
  -f net10.0-android \
  -c Debug \
  -t:Run
```

> **Tip:** Start an emulator first with Android Studio's AVD Manager, or connect a physical device with USB debugging enabled.

---

## 📦 Building a Debug APK

```bash
dotnet build src/Rooted.Mobile/Rooted.Mobile.csproj \
  -f net10.0-android \
  -c Debug
```

The APK is written to:

```
src/Rooted.Mobile/bin/Debug/net10.0-android/com.rooted.planttracker.apk
```

Install manually with `adb`:

```bash
adb install -r "src/Rooted.Mobile/bin/Debug/net10.0-android/com.rooted.planttracker.apk"
```

---

## 🚢 Release Build & Signed APK

### 1. Publish (auto-signed with debug key for side-loading)

```bash
dotnet restore src/Rooted.Mobile/Rooted.Mobile.csproj

dotnet publish src/Rooted.Mobile/Rooted.Mobile.csproj \
  -f net10.0-android \
  -c Release
```

The signed APK is written to:

```
src/Rooted.Mobile/bin/Release/net10.0-android/publish/com.rooted.planttracker-Signed.apk
```

### 2. Sign with your own keystore (Play Store / production)

Create a keystore once:

```bash
keytool -genkey -v \
  -keystore rooted.keystore \
  -alias rooted \
  -keyalg RSA -keysize 2048 \
  -validity 10000
```

Then publish with signing properties:

```bash
dotnet publish src/Rooted.Mobile/Rooted.Mobile.csproj \
  -f net10.0-android \
  -c Release \
  -p:AndroidKeyStore=true \
  -p:AndroidSigningKeyStore=rooted.keystore \
  -p:AndroidSigningKeyAlias=rooted \
  -p:AndroidSigningKeyPass=<key-password> \
  -p:AndroidSigningStorePass=<store-password>
```

> ⚠️ Never commit your `.keystore` file or passwords to source control.

### 3. Install on device

```bash
adb install -r "src/Rooted.Mobile/bin/Release/net10.0-android/publish/com.rooted.planttracker-Signed.apk"
```

---

## 🤖 AI Features (Optional)

Rooted can optionally use **OpenAI GPT-4o mini** for:

- Photo-based plant analysis (stage, health, recommended actions)
- Growing guide generation
- Plant identification from a photo or name

### Setup

1. Obtain an [OpenAI API key](https://platform.openai.com/api-keys)
2. Open Rooted → **Settings** → **AI Settings**
3. Paste your API key

The key is stored securely using the device keychain — it is never included in backups or sent anywhere other than OpenAI.

AI features are fully optional. The app works completely offline without a key.

---

## 💾 Backup & Restore

### Export

Settings → **Export Backup** — creates a `.zip` snapshot containing the database and all plant photos. The OS share sheet opens so you can save it to Files, email it, or send it anywhere.

> **Note on backup size:** Photos are resized and compressed before storage, keeping backups small and portable. In practice this reduced backup size from ~55 MB to ~2 MB with no meaningful loss of quality for in-app use.

### Restore

Settings → **Import Backup** — pick a `.zip` backup file.

- If the app has existing data, you are shown a warning with the backup's metadata (date, plant count, photo count) and offered the option to **back up your current data first**
- If the app is empty, restore proceeds immediately with no prompt
- Restore is a **full overwrite** — your device data is replaced completely with the backup
- If anything goes wrong, the original data is kept intact

---

## 📐 Conventions & Architecture Notes

See [`AGENTS.md`](AGENTS.md) for the full developer reference, including:

- Entity schemas and field definitions
- Database schema versioning and migration rules
- Backup/restore rules (authoritative spec)
- Plant profile registry and stage catalog definitions
- AI prompt strategy IDs
- UI/UX guidelines and screen inventory

---

## 🧩 Tech Stack

| | |
|---|---|
| **Framework** | .NET 10, .NET MAUI (Android) |
| **UI pattern** | MVVM — CommunityToolkit.Mvvm |
| **Database** | SQLite via sqlite-net-pcl |
| **AI** | OpenAI API (`gpt-4o-mini`) |
| **Min Android** | API 21 (Android 5.0) |
| **App ID** | `com.rooted.planttracker` |
| **Version** | 4.1 (build 6) |

---

## 📄 License

Personal project — not licensed for redistribution.
