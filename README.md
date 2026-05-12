<div align="center">

<img src="docs/media/icon.png" width="96" alt="XpensTracker icon" />

# XpensTracker

**Effortless expense tracking for the UAE — Google Pay notifications and bank SMS become expenses, automatically.**

**100% offline  ·  Zero data collection  ·  Nothing ever leaves your phone.**

![Android](https://img.shields.io/badge/Android-9%2B-3DDC84?logo=android&logoColor=white)
![Kotlin](https://img.shields.io/badge/Kotlin-2.3.20-7F52FF?logo=kotlin&logoColor=white)
![Compose](https://img.shields.io/badge/Jetpack%20Compose-2026.03-4285F4?logo=jetpackcompose&logoColor=white)
![Hilt](https://img.shields.io/badge/Hilt-2.59.2-34A853)
![Room](https://img.shields.io/badge/Room-2.8.4-FF6F00)

<br />

<!-- TODO: replace href="#" below with your Free / Pro APK download URLs -->
<a href="#"><img src="https://img.shields.io/badge/Download-Free%20APK-BB9844?style=for-the-badge&logo=android&logoColor=white" alt="Download Free APK" /></a>
&nbsp;
<a href="#"><img src="https://img.shields.io/badge/Download-Pro%20APK-9C6830?style=for-the-badge&logo=android&logoColor=white" alt="Download Pro APK" /></a>

<br /><br />

<video src="https://github.com/user-attachments/assets/bcc0c3a7-0ec0-4576-ad05-e4bdef923317" controls width="320" muted></video>

</div>

---

## Features

- **Auto-capture from Google Pay** — notifications are parsed on-device into expenses. Zero taps.
- **Bank SMS parsing *(Pro)*** — Emirates NBD, Emirates Islamic, ADCB, FAB out of the box; Mashreq, DIB, RAKBANK, CBD, ADIB attributed by sender.
- **Self-learning unmatched queue** — unparseable SMS land in a queue; resolve 3 from the same sender and the app learns its pattern.
- **Smart categorization** — UAE merchant catalog (Carrefour, Lulu, Talabat, Careem, DEWA, Etisalat…) + 100+ keyword rules.
- **Insights** — monthly trends, observation cards, category breakdowns.
- **Filters** — chip-based filtering by category, bank, and payment method.
- **Dual theme** — hand-tuned light (*Shams*) and dark (*Layl*) palettes.
- **On-device only** — no telemetry, no network upload of SMS or notifications.

---

## Screens — Shams (light)

<table>
  <tr>
    <td align="center"><img src="docs/media/lighthome.png" width="240" alt="Dashboard" /><br /><sub><b>Dashboard</b><br />Balance, pie, recent</sub></td>
    <td align="center"><img src="docs/media/lightledger.png" width="240" alt="Ledger" /><br /><sub><b>Ledger</b><br />Chip filters, search</sub></td>
    <td align="center"><img src="docs/media/lightfilter.png" width="240" alt="Filter sheet" /><br /><sub><b>Filter</b><br />Multi-dimensional filtering</sub></td>
  </tr>
  <tr>
    <td align="center"><img src="docs/media/lightinsights.png" width="240" alt="Insights" /><br /><sub><b>Insights</b><br />Trends & observations</sub></td>
    <td align="center"><img src="docs/media/lightsettings.png" width="240" alt="Settings" /><br /><sub><b>Settings</b><br />Theme, profile, perms</sub></td>
    <td></td>
  </tr>
</table>

## Screens — Layl (dark)

<table>
  <tr>
    <td align="center"><img src="docs/media/darkhome.png" width="240" alt="Dashboard (dark)" /><br /><sub><b>Dashboard</b></sub></td>
    <td align="center"><img src="docs/media/darkledger.png" width="240" alt="Ledger (dark)" /><br /><sub><b>Ledger</b></sub></td>
    <td align="center"><img src="docs/media/darkfilter.png" width="240" alt="Filter sheet (dark)" /><br /><sub><b>Filter</b></sub></td>
  </tr>
  <tr>
    <td align="center"><img src="docs/media/darkinsights.png" width="240" alt="Insights (dark)" /><br /><sub><b>Insights</b></sub></td>
    <td align="center"><img src="docs/media/darksettings.png" width="240" alt="Settings (dark)" /><br /><sub><b>Settings</b></sub></td>
    <td></td>
  </tr>
</table>

---

## Architecture

Multi-module **MVVM + Clean Architecture**. Pure-Kotlin domain at the center.

```mermaid
graph TD
    app[app]
    home[feature/home]
    expenses[feature/expenses]
    analytics[feature/analytics]
    sms[core/sms-parsing]
    data[core/data]
    domain[core/domain]
    ui[core/ui]

    app --> home & expenses & analytics & ui & data & domain
    home --> ui & domain
    expenses --> sms & ui & domain
    analytics --> ui & domain
    sms --> data & domain
    data --> domain
    ui --> domain

    classDef pure fill:#48886E,stroke:#1C1E32,color:#FAF7F0
    classDef android fill:#B98441,stroke:#1C1E32,color:#FAF7F0
    class domain pure
    class app,home,expenses,analytics,sms,data,ui android
```

---

## Build variants

`flavorDimensions = ["tier"]` — Free and Pro install **side-by-side** (different `applicationId`s).

| Variant | applicationId | Capabilities |
|---|---|---|
| **Free** | `com.example.xpenstracker` | GPay notification capture |
| **Pro** | `com.example.xpenstracker.pro` | GPay **+** UAE bank SMS **+** self-learning queue |

---

## Tech stack

Jetpack Compose (BOM 2026.03.01) · Material 3 · Hilt 2.59.2 · Room 2.8.4 · DataStore 1.2.1 · Vico 3.0.3 · Kotlin 2.3.20 · KSP · Min SDK 28 · Target/Compile SDK 36

<details>
<summary><b>Project structure</b></summary>

```
app/                  Main activity, navigation, global ViewModels
  src/main/           Shared code + GPay notification listener
  src/free/           Rule-based classifier binding
  src/pro/            SMS listener + layered classifier
core/domain/          Pure Kotlin: entities, repos, use cases
core/data/            Room DB, DataStore, classifier rules
core/ui/              Theme + shared Compose components
core/sms-parsing/     Bank pattern catalog, regex parser, learned-pattern store
feature/home/         Dashboard + unmatched queue (Pro)
feature/expenses/     Expense list, filters, add/edit
feature/analytics/    Monthly insights + trends
```

</details>

---

## Privacy

XpensTracker is **on-device only**. No telemetry, no analytics, no backend. Notification and SMS content is parsed locally and never leaves your device. Notification access is split into two distinct services — *"XpensTracker — Google Pay"* and *"XpensTracker — Bank SMS"* — so you grant each independently.

---

<div align="center">

Built by [Sokhib](https://github.com/Sokhib) · Made with Compose

</div>
