<div align="center">

<img src="docs/media/icon.png" width="96" alt="XpensTracker icon" />

# XpensTracker

**Effortless expense tracking for the UAE — Google Pay notifications and bank SMS become expenses, automatically.**

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
  </tr>
  <tr>
    <td align="center"><img src="docs/media/lightinsights.png" width="240" alt="Insights" /><br /><sub><b>Insights</b><br />Trends & observations</sub></td>
    <td align="center"><img src="docs/media/lightsettings.png" width="240" alt="Settings" /><br /><sub><b>Settings</b><br />Theme, profile, perms</sub></td>
  </tr>
</table>

## Screens — Layl (dark)

<table>
  <tr>
    <td align="center"><img src="docs/media/darkhome.png" width="240" alt="Dashboard (dark)" /><br /><sub><b>Dashboard</b></sub></td>
    <td align="center"><img src="docs/media/darkledger.png" width="240" alt="Ledger (dark)" /><br /><sub><b>Ledger</b></sub></td>
  </tr>
  <tr>
    <td align="center"><img src="docs/media/darkinsights.png" width="240" alt="Insights (dark)" /><br /><sub><b>Insights</b></sub></td>
    <td align="center"><img src="docs/media/darksettings.png" width="240" alt="Settings (dark)" /><br /><sub><b>Settings</b></sub></td>
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

<details>
<summary><b>SMS → Expense pipeline (Pro)</b></summary>

```mermaid
flowchart TD
    A[Bank SMS] --> B{Learned pattern?}
    B -- yes --> C[Apply learned regex]
    B -- no --> D{Bundled bank config?}
    D -- yes --> E[Apply bank regex]
    D -- no --> F[Unmatched queue]
    C & E --> G{Amount extracted?}
    G -- yes --> H[Classify + Insert]
    G -- no --> F
    F --> J[User opens row] --> K[GenericFallbackParser pre-fills] --> L[User saves]
    L --> M{≥ 3 saves for sender?}
    M -- yes --> N[Derive learned pattern]
```

</details>

---

## Build variants

`flavorDimensions = ["tier"]` — Free and Pro install **side-by-side** (different `applicationId`s).

| Variant | applicationId | Capabilities |
|---|---|---|
| **Free** | `com.example.xpenstracker` | GPay notification capture |
| **Pro** | `com.example.xpenstracker.pro` | GPay **+** UAE bank SMS **+** self-learning queue |

---

## Design system — AquaTheme

Custom Material 3 theme inspired by Dubai / MENA warmth: gold, sage, desert rose.

**Shams (light)**

| Primary | Secondary | Tertiary | Surface | On Surface |
|---|---|---|---|---|
| ![](https://placehold.co/20x20/B98441/B98441.png) `#B98441` | ![](https://placehold.co/20x20/48886E/48886E.png) `#48886E` | ![](https://placehold.co/20x20/C45D46/C45D46.png) `#C45D46` | ![](https://placehold.co/20x20/FAF7F0/FAF7F0.png) `#FAF7F0` | ![](https://placehold.co/20x20/1C1E32/1C1E32.png) `#1C1E32` |

**Layl (dark)**

| Primary | Secondary | Tertiary | Surface | On Surface |
|---|---|---|---|---|
| ![](https://placehold.co/20x20/E8B765/E8B765.png) `#E8B765` | ![](https://placehold.co/20x20/9CD9B8/9CD9B8.png) `#9CD9B8` | ![](https://placehold.co/20x20/E29785/E29785.png) `#E29785` | ![](https://placehold.co/20x20/181824/181824.png) `#181824` | ![](https://placehold.co/20x20/FAF6EC/FAF6EC.png) `#FAF6EC` |

<details>
<summary><b>Category palette · typography · spacing · components</b></summary>

### Category colors

| | Food | Transport | Shopping | Bills | Entertainment | Health | Education | Grocery |
|---|---|---|---|---|---|---|---|---|
| Light | ![](https://placehold.co/16x16/BC6A47/BC6A47.png) | ![](https://placehold.co/16x16/3F6B90/3F6B90.png) | ![](https://placehold.co/16x16/9B5A85/9B5A85.png) | ![](https://placehold.co/16x16/C25E54/C25E54.png) | ![](https://placehold.co/16x16/B0983F/B0983F.png) | ![](https://placehold.co/16x16/4D8C72/4D8C72.png) | ![](https://placehold.co/16x16/5976B3/5976B3.png) | ![](https://placehold.co/16x16/3F8C5C/3F8C5C.png) |
| Dark | ![](https://placehold.co/16x16/E09878/E09878.png) | ![](https://placehold.co/16x16/8AADC5/8AADC5.png) | ![](https://placehold.co/16x16/C79BBE/C79BBE.png) | ![](https://placehold.co/16x16/E39089/E39089.png) | ![](https://placehold.co/16x16/CFC274/CFC274.png) | ![](https://placehold.co/16x16/9CD9B8/9CD9B8.png) | ![](https://placehold.co/16x16/8AA2D4/8AA2D4.png) | ![](https://placehold.co/16x16/7BCC8B/7BCC8B.png) |

### Typography

- **Fraunces** *(variable serif)* — display headlines + monetary amounts (`amount` style at 22sp)
- **Figtree** *(variable sans)* — body & labels, weights 400–700

### Spacing scale

`xxs 2` · `xs 4` · `sm 8` · `md 12` · `lg 16` · `xl 24` · `xxl 32` · `xxxl 48` · `xxxxl 64` (dp)

### Components

`AquaCard` · `AquaTopBar` · `AquaBottomBar` · `AquaButton` · `AquaTextField` · `AquaSearchField` · `AquaChip` · `AquaPieChart` · `AquaHeroBalanceCard` · `AquaStatPill` · `AquaMiniStat` · `AquaCategoryAvatar` · `AquaExpenseRow` · `AquaBadge` · `AquaBanner` · `AquaShimmer` · `AquaEmptyState` · `AquaObservationCard` · `AquaDivider`

</details>

---

## Tech stack

Jetpack Compose (BOM 2026.03.01) · Material 3 · Hilt 2.59.2 · Room 2.8.4 · DataStore 1.2.1 · Vico 3.0.3 · Lucide Compose 1.1.0 · Kotlin 2.3.20 · KSP · Min SDK 28 · Target/Compile SDK 36

<details>
<summary><b>Project structure</b></summary>

```
app/                  Main activity, navigation, global ViewModels
  src/main/           Shared code + GPay notification listener
  src/free/           Rule-based classifier binding
  src/pro/            SMS listener + layered classifier
core/domain/          Pure Kotlin: entities, repos, use cases
core/data/            Room DB, DataStore, classifier rules
core/ui/              AquaTheme + shared Compose components
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
