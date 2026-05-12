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

<!-- TODO: replace the two href="#" below with your Free / Pro APK download URLs (Release asset or Play Store link) -->
<a href="#">
  <img src="https://img.shields.io/badge/Download-Free%20APK-BB9844?style=for-the-badge&logo=android&logoColor=white" alt="Download Free APK" />
</a>
&nbsp;
<a href="#">
  <img src="https://img.shields.io/badge/Download-Pro%20APK-9C6830?style=for-the-badge&logo=android&logoColor=white" alt="Download Pro APK" />
</a>

<br /><br />

<img src="docs/media/lighthome.png" width="320" alt="XpensTracker dashboard" />

</div>

---

## Demo

<video src="https://github.com/Sokhib/XpensTracker-Wiki/raw/main/docs/media/Screen_recording_20260512_084245.webm" controls width="320"></video>

> Safari doesn't play `.webm` inline — [open the video directly](docs/media/Screen_recording_20260512_084245.webm) instead.

---

## Features

### Auto-capture from Google Pay

> Listens to your Google Pay notifications, parses the amount and merchant, and files the expense for you. Zero taps.

A foreground `NotificationListenerService` runs in both flavors, watching for GPay payment notifications. Amounts and merchant names are extracted on-device and saved as expenses — automatically categorized using a UAE-aware rule engine.

### Bank SMS parsing *(Pro)*

> Your bank's transaction SMS becomes an expense the moment it arrives.

Pro ships bundled patterns for **Emirates NBD**, **Emirates Islamic**, **ADCB**, and **FAB**, plus sender-only attribution stubs for **Mashreq**, **DIB**, **RAKBANK**, **CBD**, and **ADIB**. SMS that don't match any bank pattern land in an *unmatched queue* — open one and the app pre-fills the edit form from a best-effort generic parser. Save it, and the app learns the sender's pattern after **3 confirmations**.

### Smart categorization

> Carrefour → Groceries. DEWA → Bills. Talabat → Food. No setup required.

Two layers of UAE-aware classification:
- A canonical **merchant catalog** with exact-match mappings (Carrefour, Lulu, Talabat, Careem, DEWA, Etisalat, etc.)
- A **rule-based classifier** with 100+ keyword rules covering the long tail

If nothing matches, the expense is flagged `Uncategorized` so you can review it — distinct from `Other`, which is a real explicit user choice.

### Dashboard

> Hero balance card, category pie, recent expenses, unmatched-queue badge.

<img src="docs/media/lighthome.png" width="280" alt="Dashboard" />

### Expense ledger with chip filters

> Filter by category, bank, payment method. Swipe to delete. Search by merchant.

<img src="docs/media/lightledger.png" width="280" alt="Expense ledger" />

### Filter sheet

> Chip-based multi-dimensional filtering — combine categories, banks, and payment methods.

<img src="docs/media/lightfilter.png" width="280" alt="Filter sheet" />

### Monthly insights

> Natural-language observations on your spending patterns, trend charts, category breakdowns.

<img src="docs/media/lightinsights.png" width="280" alt="Monthly insights" />

### Settings

> Theme, currency, profile, notification permissions — including separate toggles for the GPay and Bank-SMS listeners.

<img src="docs/media/lightsettings.png" width="280" alt="Settings" />

---

## Layl — Dark mode

XpensTracker ships a hand-tuned dark palette called **Layl** (Arabic: *night*). Deep midnight surfaces, warm gold primary, ivory text — designed to be easy on the eyes without losing the warmth of the light theme.

<table>
  <tr>
    <td align="center"><img src="docs/media/darkhome.png" width="260" alt="Dashboard (dark)" /><br /><sub>Dashboard</sub></td>
    <td align="center"><img src="docs/media/darkledger.png" width="260" alt="Ledger (dark)" /><br /><sub>Ledger</sub></td>
  </tr>
  <tr>
    <td align="center"><img src="docs/media/darkinsights.png" width="260" alt="Insights (dark)" /><br /><sub>Insights</sub></td>
    <td align="center"><img src="docs/media/darksettings.png" width="260" alt="Settings (dark)" /><br /><sub>Settings</sub></td>
  </tr>
</table>

---

## Architecture

Multi-module **MVVM + Clean Architecture**. Pure-Kotlin domain at the center, Android-aware layers wrap around it.

### Module dependency graph

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

    app --> home
    app --> expenses
    app --> analytics
    app --> ui
    app --> data
    app --> domain

    home --> ui
    home --> domain
    expenses --> sms
    expenses --> ui
    expenses --> domain
    analytics --> ui
    analytics --> domain

    sms --> data
    sms --> domain
    data --> domain
    ui --> domain

    classDef pure fill:#48886E,stroke:#1C1E32,color:#FAF7F0
    classDef android fill:#B98441,stroke:#1C1E32,color:#FAF7F0
    class domain pure
    class app,home,expenses,analytics,sms,data,ui android
```

**Dependency rule:** `feature/*` and `app` depend on `core/*`. `core/domain` has zero Android dependencies. `core/data` implements domain interfaces. `core/sms-parsing` depends on `core/domain` + `core/data`.

### SMS → Expense pipeline *(Pro)*

```mermaid
flowchart TD
    A[Bank SMS notification] --> B{Learned pattern<br/>for this sender?}
    B -- yes --> C[Apply learned regex]
    B -- no --> D{Sender matches<br/>bundled bank config?}
    D -- yes --> E[Apply bank regex]
    D -- no --> F[Push to Unmatched queue]

    C --> G{Extracted<br/>amount?}
    E --> G
    G -- yes --> H[Classify category<br/>via LayeredCategoryClassifier]
    G -- no --> F

    H --> I[Insert Expense]
    F --> J[User opens row in Edit screen]
    J --> K[GenericFallbackParser pre-fills fields]
    K --> L[User saves]
    L --> M[markResolved + recordSave]
    M --> N{Save count<br/>≥ 3 for this sender?}
    N -- yes --> O[Derive learned pattern]
    N -- no --> P[Wait for more saves]
```

### Build flavors

```mermaid
graph LR
    subgraph Free["🆓 Free flavor"]
        F1[GPay listener] --> F2[RuleBasedCategoryClassifier]
        F2 --> F3[Expense DB]
    end

    subgraph Pro["⭐ Pro flavor"]
        P1[GPay listener] --> P2[LayeredCategoryClassifier]
        P3[SMS listener] --> P4{Bank pattern<br/>match?}
        P4 -- yes --> P2
        P4 -- no --> P5[Unmatched queue]
        P5 -.user resolves.-> P2
        P2 --> P6[Expense DB]
    end
```

---

## Build variants

`flavorDimensions = ["tier"]` — Free and Pro install **side-by-side** (different `applicationId`s).

| Variant | applicationId | Version suffix | Capabilities |
|---|---|---|---|
| **Free** | `com.example.xpenstracker` | — | GPay notification capture only |
| **Pro** | `com.example.xpenstracker.pro` | `-pro` | GPay capture **+** UAE bank SMS parsing **+** self-learning unmatched queue |

The database schema is shared — Pro-only tables ship empty in Free.

---

## Design system — AquaTheme

A custom Material 3 theme built around **Dubai / MENA** warmth: gold, sage, desert rose, deep midnight. Two palettes:

### Shams (light) — *warm sand paper*

| Token | Hex | |
|---|---|---|
| Primary | `#B98441` | ![](https://placehold.co/20x20/B98441/B98441.png) |
| Primary Dim | `#9C6830` | ![](https://placehold.co/20x20/9C6830/9C6830.png) |
| Secondary (Sage) | `#48886E` | ![](https://placehold.co/20x20/48886E/48886E.png) |
| Tertiary (Desert Rose) | `#C45D46` | ![](https://placehold.co/20x20/C45D46/C45D46.png) |
| Surface | `#FAF7F0` | ![](https://placehold.co/20x20/FAF7F0/FAF7F0.png) |
| On Surface | `#1C1E32` | ![](https://placehold.co/20x20/1C1E32/1C1E32.png) |

### Layl (dark) — *deep midnight*

| Token | Hex | |
|---|---|---|
| Primary | `#E8B765` | ![](https://placehold.co/20x20/E8B765/E8B765.png) |
| Primary Dim | `#C28642` | ![](https://placehold.co/20x20/C28642/C28642.png) |
| Secondary (Sage) | `#9CD9B8` | ![](https://placehold.co/20x20/9CD9B8/9CD9B8.png) |
| Tertiary (Desert Rose) | `#E29785` | ![](https://placehold.co/20x20/E29785/E29785.png) |
| Surface | `#181824` | ![](https://placehold.co/20x20/181824/181824.png) |
| On Surface | `#FAF6EC` | ![](https://placehold.co/20x20/FAF6EC/FAF6EC.png) |

### Category palette

Each category gets its own tuned hue, paired across light/dark.

| Category | Light | | Dark | |
|---|---|---|---|---|
| Food | `#BC6A47` | ![](https://placehold.co/20x20/BC6A47/BC6A47.png) | `#E09878` | ![](https://placehold.co/20x20/E09878/E09878.png) |
| Transport | `#3F6B90` | ![](https://placehold.co/20x20/3F6B90/3F6B90.png) | `#8AADC5` | ![](https://placehold.co/20x20/8AADC5/8AADC5.png) |
| Shopping | `#9B5A85` | ![](https://placehold.co/20x20/9B5A85/9B5A85.png) | `#C79BBE` | ![](https://placehold.co/20x20/C79BBE/C79BBE.png) |
| Bills | `#C25E54` | ![](https://placehold.co/20x20/C25E54/C25E54.png) | `#E39089` | ![](https://placehold.co/20x20/E39089/E39089.png) |
| Entertainment | `#B0983F` | ![](https://placehold.co/20x20/B0983F/B0983F.png) | `#CFC274` | ![](https://placehold.co/20x20/CFC274/CFC274.png) |
| Health | `#4D8C72` | ![](https://placehold.co/20x20/4D8C72/4D8C72.png) | `#9CD9B8` | ![](https://placehold.co/20x20/9CD9B8/9CD9B8.png) |
| Education | `#5976B3` | ![](https://placehold.co/20x20/5976B3/5976B3.png) | `#8AA2D4` | ![](https://placehold.co/20x20/8AA2D4/8AA2D4.png) |
| Grocery | `#3F8C5C` | ![](https://placehold.co/20x20/3F8C5C/3F8C5C.png) | `#7BCC8B` | ![](https://placehold.co/20x20/7BCC8B/7BCC8B.png) |
| Other | `#7F8391` | ![](https://placehold.co/20x20/7F8391/7F8391.png) | `#A4A6B0` | ![](https://placehold.co/20x20/A4A6B0/A4A6B0.png) |

### Typography

| Family | Role | Notable use |
|---|---|---|
| **Fraunces** *(variable serif)* | Display headlines, monetary amounts | `amount` style at 22sp |
| **Figtree** *(variable sans)* | Body & labels | 14–16sp, Regular / Medium / SemiBold / Bold |

### Spacing scale

| Token | dp | | Token | dp |
|---|---|---|---|---|
| none | 0 | | lg | 16 |
| xxs | 2 | | xl | 24 |
| xs | 4 | | xxl | 32 |
| sm | 8 | | xxxl | 48 |
| md | 12 | | xxxxl | 64 |

### Components

Reusable composables in `core/ui/aqua/components/`:

| Layout | Input | Data viz | Feedback |
|---|---|---|---|
| `AquaCard` | `AquaButton` | `AquaPieChart` | `AquaBanner` |
| `AquaTopBar` | `AquaTextField` | `AquaHeroBalanceCard` | `AquaShimmer` |
| `AquaBottomBar` | `AquaSearchField` | `AquaStatPill` | `AquaEmptyState` |
| `AquaDivider` | `AquaChip` | `AquaMiniStat` | `AquaBadge` |
|  |  | `AquaCategoryAvatar` | `AquaObservationCard` |
|  |  | `AquaExpenseRow` |  |

---

## Tech stack

| Layer | Library | Version |
|---|---|---|
| **UI** | Jetpack Compose (BOM) | 2026.03.01 |
| | Material 3 | (BOM) |
| **Architecture** | MVVM + Clean | — |
| **DI** | Hilt | 2.59.2 |
| **Persistence** | Room | 2.8.4 |
| | DataStore (Preferences) | 1.2.1 |
| **Charts** | Vico | 3.0.3 |
| **Icons** | Lucide Compose | 1.1.0 |
| **Language** | Kotlin | 2.3.20 |
| **Build** | KSP | (Kotlin-aligned) |
| **Platform** | Min SDK 28 (Android 9) · Target/Compile SDK 36 | — |

---

## Project structure

```
app/                      Main activity, navigation, global ViewModels
  src/main/               Shared code + GPay notification listener
  src/free/               Free flavor: rule-based classifier
  src/pro/                Pro flavor: SMS listener, layered classifier
core/domain/              Pure Kotlin — entities, repos, use cases (zero Android deps)
core/data/                Room DB, DataStore, classifier rules
core/ui/                  AquaTheme + shared Compose components
core/sms-parsing/         Bank pattern catalog, regex parser, learned-pattern store
feature/home/             Dashboard + unmatched queue (Pro)
feature/expenses/         Expense list, filters, add/edit
feature/analytics/        Monthly insights + trends
```

---

## Privacy

XpensTracker is **on-device only**. There is no telemetry, no analytics SDK, no backend. Notification and SMS content is parsed locally and never leaves your device.

- Notification access is split into two distinct services so you grant each one independently — *"XpensTracker — Google Pay"* and *"XpensTracker — Bank SMS"*.
- No network permission is used for any expense data.
- Your expense database lives in private app storage; CSV export goes through a `FileProvider` share sheet.

---

<div align="center">

Built by [Sokhib](https://github.com/Sokhib) · Made with Compose

</div>
