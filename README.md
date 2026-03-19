<div align="center">

# check-d

**turn completions into momentum.**

*An iCloud-powered iOS productivity app focused on logging what you finished,*
*reinforcing consistency with AI motivation, and visualising progress over time.*

<br />

![Swift](https://img.shields.io/badge/Swift-5.9-F05138?style=flat-square&logo=swift&logoColor=white)
![iOS](https://img.shields.io/badge/iOS-17%2B-000000?style=flat-square&logo=apple&logoColor=white)
![CloudKit](https://img.shields.io/badge/CloudKit-Private_DB-3478F6?style=flat-square&logo=icloud&logoColor=white)
![SwiftUI](https://img.shields.io/badge/SwiftUI-Native-0A84FF?style=flat-square&logo=swift&logoColor=white)
![Status](https://img.shields.io/badge/Status-Coming_Soon-6b8fff?style=flat-square)

</div>

---

## what is check-d?

Most productivity apps make you feel behind. check-d flips that. Instead of tracking what's left to do, you log what you've already finished — your wins, your completions, your momentum.

Every task you log feeds into a streak, earns XP, triggers an AI motivational response, and adds to an analytics dashboard that shows you exactly how consistent you've been. The goal is simple: finish things, log them, build the habit.

---

## features

### done-task logging
Tap the centre `+` button and record something you completed. No categories, no priorities, no friction. Just wins.

### AI motivation
Every logged win triggers a short, focused motivational response from an AI layer built directly into the task completion flow — not a chatbot, a reinforcement system.

### personalised tone
Choose how you want to be pushed. Three modes:
- **encouraging** — warm, positive reinforcement
- **strict** — direct, no-fluff accountability
- **playful** — light and energetic

### streak & XP tracking
Current streaks and daily completion counts update in real time. XP accumulates across every task, feeding a level-up system that rewards consistency over time.

### progress dashboard
Week, month, and year views give you the full picture. Tap any active day in the heatmap to inspect exactly what you logged.

### guided daily check-in
Three structured journal prompts each day — joyful moments, lessons learned, emotional check-in. Select from curated pill tags or write your own. Entries persist and are editable from the history view.

---

## architecture

check-d is built entirely on Apple's stack. No external auth, no third-party database.

```
checkdApp
├── RootView                  — iCloud account gate + coordinated launch
├── MainContentView           — TabView shell + BlobTabBar
│   ├── HomeCloudTab          — Done-task feed, AI responses, streak count
│   ├── AnalyticsCloudTab     — Heatmap, streaks, XP, stats dashboard
│   ├── PetCloudTab           — Daily check-in, journal prompts, history
│   └── ProfileCloudTab       — Display name, tone, accent, dark mode
│
├── Managers
│   ├── CloudKitManager       — All CKRecord read/write operations
│   ├── TaskManager           — Task CRUD, ownerRecordName scoping
│   ├── UserProfileManager    — Profile fetch, update, sign-out
│   ├── PetManager            — XP, level, expression state
│   ├── CheckInManager        — Check-in save/fetch, today/history
│   └── AIService             — Motivation request → Cloudflare Worker → Gemini
│
└── Supporting
    ├── ThemeManager          — Accent colour, dark/light mode, glow opacities
    ├── SnowfallView          — Particle background system
    └── AppSecrets            — Worker URL + secret (gitignored, CI-generated)
```

### data model

| Record Type | Key Fields |
|---|---|
| `Task` | `title`, `createdAt`, `ownerRecordName` |
| `UserProfile` | `displayName`, `preferredTone`, `totalCompletedTasks`, `currentStreak` |
| `Pet` | `xp`, `level`, `totalXp`, `ownerRecordName` |
| `CheckIn` | `date`, `joyfulMoment`, `lessonLearned`, `emotionalCheckIn`, `ownerRecordName` |

All records are scoped to the authenticated iCloud user via `ownerRecordName`. The app uses CloudKit's **private database** — data never leaves the user's iCloud account.

---

## AI integration

Motivation is triggered automatically after every task is logged. The flow:

```
Task saved → AIService.getMotivationalMessage()
          → POST to Cloudflare Worker (authenticated via APP_SECRET)
          → Worker → Gemini 2.5 Flash API
          → Short motivational response (max 15 words)
          → Displayed on HomeCloudTab with fade-in animation
```

The worker validates every request against a shared secret. If the AI call fails for any reason, the app degrades gracefully with fallback text — the core experience is never blocked.

---

## design system

check-d follows Apple's Human Interface Guidelines closely.

- **Floating tab bar** with 5 controls and a centre FAB-style `+` action
- **Transparent cards** with hairline borders — no heavy fills
- **Accent colour system** — 4 user-selectable colours applied app-wide including glow effects
- **Light / dark mode** — light default, user-toggled in profile settings, persisted across sessions
- **Particle background** — accent-tinted dots falling gently behind all content
- **Typography** — custom Inter font family throughout
- **Animations** — spring-based transitions, `.contentTransition(.numericText())` counters, smooth expand/collapse for check-in card

---

## privacy

| What | How |
|---|---|
| Identity | Derived from iCloud account — no email, no password |
| Storage | CloudKit private database — Apple's infrastructure |
| Data isolation | Every record scoped by `ownerRecordName` to the signed-in user |
| AI requests | Routed through a Cloudflare Worker — no raw API keys on device |
| Secrets | `AppSecrets.swift` is gitignored — generated at build time via Xcode Cloud CI |

No external auth database. No password handling. No ads. No tracking.

---

## CI / CD

Built and distributed via **Xcode Cloud**.

- `ci_scripts/ci_pre_xcodebuild.sh` generates `AppSecrets.swift` from Xcode Cloud environment variables before every build
- CloudKit schema is deployed to Production via the CloudKit Dashboard before each TestFlight release
- Distribution profile includes the iCloud / CloudKit entitlement

---

## status

> check-d is currently in development and TestFlight testing.
> App Store release coming soon.

---

## tech stack

| Layer | Technology |
|---|---|
| Language | Swift 5.9 |
| UI | SwiftUI |
| Backend | CloudKit (Apple private database) |
| Auth | iCloud account (CKAccountStatus) |
| AI | Gemini 2.5 Flash via Cloudflare Worker |
| CI/CD | Xcode Cloud |
| Distribution | TestFlight → App Store |

---

<div align="center">

*built with SwiftUI + iCloud*

</div>
