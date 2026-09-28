# Quiz Game Template — Documentation

A complete, production-ready quiz game engine for Unity. The template ships with a football-themed
sample dataset, but the engine is topic-agnostic: replace the JSON data and images and you have a
movie quiz, a history quiz, a geography quiz, or anything else built on "guess the answer from
visual clues."

### Contents

| | |
|---|---|
| [1. Requirements](#1-requirements) | Unity version, modules, first launch, input handling |
| [2. Project structure](#2-project-structure) | Folder tree, the setup menu, naming and language, script index |
| [3. The data system](#3-the-data-system--read-this-first) | How questions, images and levels fit together — **read this first** |
| [4. Backend setup — LootLocker](#4-backend-setup--lootlocker-leaderboards) | Leaderboards, friends, name moderation |
| [5. Ads — Google AdMob](#5-ads--google-admob) | Installing the ad plugin, unit IDs, how often ads appear |
| [6. In-app purchases](#6-in-app-purchases) | Product IDs, restore purchases |
| [7. Audio](#7-audio--bring-your-own) | Empty by design — where to drop your clips |
| [8. Localization](#8-localization-english--turkish) | The EN/TR string system |
| [9. Branding and building](#9-branding-package-name-and-building) | Rename the app, Android and iOS builds |
| [10. Troubleshooting](#10-troubleshooting) | Expected console warnings, common problems |
| [11. What's included](#11-whats-included) | Feature list, and what is deliberately not included |
| **[12. Customization reference](#12-customization-reference--what-you-can-change-and-where)** | **Every file and line you are meant to edit — and the few you should not** |

**If you only read one section, read [12](#12-customization-reference--what-you-can-change-and-where).**
It lists, file by file and line by line, what to change to make this template your own — the keys and
IDs you must replace before shipping, the data and artwork that define your topic, the gameplay
numbers, and the handful of names that will break the project silently if you rename them carelessly.

---

## 1. Requirements

| | |
|---|---|
| Unity version | **6000.0.62f1** (Unity 6 LTS) |
| Modules | Android Build Support (+ SDK & NDK Tools, OpenJDK). iOS Build Support if you target iOS. |
| Platforms | **Android** — verified; developed and shipped on it. **iOS** — code paths written, never built or tested. See [iOS build](#ios-build). |

**Open the project with the exact Unity version listed above.** Opening it with a newer version
triggers an asset upgrade that can break serialized scene references.

### First launch

The product ships as a single package. It carries everything under `Assets/QuizGameTemplate/`, but
the Unity package format cannot carry `Packages/manifest.json`, the render pipeline assignment,
Active Input Handling, the screen orientation or the Build Settings scene list, and TextMesh Pro's
Essential Resources come from Unity itself. One menu item supplies all six.

1. Create a new **2D (URP)** project in Unity 6000.0.62f1, or open the project you want to add the
   template to.
2. Import the template. From the Asset Store, use **Window → Package Manager → My Assets**; from a
   downloaded file, use **Assets → Import Package → Custom Package…** and import everything.
3. What happens next depends on how you got it. The Asset Store copy declares the nine packages it
   needs as dependencies, so Unity installs them during the import and everything compiles straight
   away. A plain `.unitypackage` cannot carry that list, so there you will see compiler errors at
   this point — **expected**, and a console warning says why.
4. Run **Tools ▸ Quiz Game Template ▸ Apply Project Setup**. It adds any missing packages to your
   manifest, assigns the render pipeline, sets Active Input Handling and the screen orientation,
   fills in Build Settings and imports TextMesh Pro's Essential Resources from Unity's own uGUI
   package (into `Assets/TextMesh Pro/`, where Unity always puts them). If packages had to be added,
   Unity restores them, recompiles, and the errors clear.
   The TMP Essential Resources are in fact imported automatically the moment the template's editor
   script first loads, so TextMesh Pro's own "TMP Importer" window never needs to appear; the menu
   simply repeats that step if they are ever missing.
5. Open `Assets/QuizGameTemplate/Scenes/SplashScene.unity` and press **Play**.

Step 4 needs an internet connection — Unity downloads the packages from its own registry.
**Tools ▸ Quiz Game Template ▸ Verify Project Setup** prints an OK/FAIL line for each of the six
items and changes nothing, so you can re-check the state at any time.

**Ads are optional and not included.** The Google Mobile Ads plugin is Google's own product, so it
is not part of this package. Without it the game is fully playable: ads run in *simulation mode* —
rewarded ads grant their reward at once, banners and interstitials are skipped. Install the plugin
when you want real ads; [section 5](#5-ads--google-admob) walks through it.

> ### Input handling — why *Active Input Handling* ships as `Both`
>
> This template uses the **old** input API. There are four calls in total, all `Input.GetKeyDown`:
> the Android back button in `GameManager.cs`, `MenuManager.cs` and `LevelSelectManager.cs`, and
> the screenshot key in `ScreenshotTaker.cs`. Nothing in the project uses the new Input System — no
> scene or prefab uses `InputSystemUIInputModule`, which is why the template ships no
> `.inputactions` asset at all.
>
> The Input System package is still listed in `manifest.json`, so *Active Input Handling*
> (Project Settings → Player → Other Settings) ships set to **Both**. That keeps the existing code
> working and leaves the new Input System available if you want to build on it. It also stops Unity
> from asking, on first open, whether to enable the new backends.
>
> **Do not set it to *Input System Package (New)* on its own.** The four scripts above still
> compile and the game still launches, but they quietly stop receiving input — the back button and
> the screenshot key simply stop responding. It is a silent break with no error in the console.
>
> If you have no use for the new Input System at all, you can remove `"com.unity.inputsystem"` from
> `Packages/manifest.json` and set *Active Input Handling* to **Input Manager (Old)**. Nothing in
> the template depends on it.

---

## 2. Project structure

Everything this product owns lives under a single root folder, `Assets/QuizGameTemplate/`, so it
merges cleanly into a project that already has assets of its own.

```
Assets/
├── QuizGameTemplate/            ← everything this template owns
│   ├── Scenes/
│   │   ├── SplashScene.unity      Splash screen (build index 0 — entry point)
│   │   ├── ProfileScene.unity     First-run profile / avatar setup
│   │   ├── MainMenu.unity         Main menu
│   │   ├── LevelSelect.unity      Level map
│   │   ├── ChallengeMode.unity    Challenge mode menu
│   │   └── GameScene.unity        Gameplay screen (all modes play here)
│   ├── Scripts/                   53 C# files (GameManager is split across 6 of them)
│   ├── Resources/
│   │   ├── quiz_data.json         Question database
│   │   ├── team_colors.json       Team / category database
│   │   ├── IAPProductCatalog.json Unity IAP product catalog (section 6)
│   │   ├── Flags/                 Nationality images    (237 files, T_Flag_*)
│   │   ├── Jerseys/               Jersey number images   (99 files, T_Jersey_*)
│   │   ├── Trophies/              Position icons          (4 files, T_Position_* — see 3.3)
│   │   ├── Characters/            Player avatars         (20 files, T_Char_*)
│   │   └── Achievements/          Achievement icons      (13 files, T_Achievement_*)
│   ├── Prefabs/                   12 prefabs             (P_*)
│   ├── Textures/                  UI sprites and backgrounds (46 files, T_*)
│   ├── Fonts/                     TextMeshPro font assets (F_*)
│   ├── Animations/                clips (A_*) and animator controllers (AC_*)
│   ├── Audio/                     empty — drop your own clips here (section 7)
│   ├── Settings/                  URP pipeline asset, 2D renderer, volume profile
│   ├── Editor/                    one-click project setup — safe to delete, see below
│   ├── Documentation/             this file
│   └── Plugins/                   open-source SDKs, as their publishers ship them
│       ├── LootLockerSDK/            the LootLocker SDK (v8.1.1, MIT) and its config asset
│       └── NativeShare/              yasirkula's Native Share (MIT) — the share button
└── TextMesh Pro/                  created by Apply Project Setup — Unity's own TMP resources
```

The package installs nothing outside `Assets/QuizGameTemplate/`. `TextMesh Pro/` is written by
Unity's own TMP importer, which always uses that path. If you install the Google Mobile Ads plugin
(section 5), its importer adds `GoogleMobileAds/`, `ExternalDependencyManager/` and
`Plugins/Android` / `Plugins/iOS` next to it — those locations are Google's, not the template's.

**Asset naming.** Every asset carries a prefix for its type: `T_` textures, `P_` prefabs, `A_`
animation clips, `AC_` animator controllers, `F_` fonts. Textures also carry a category —
`T_Flag_Brazil`, `T_Jersey_9`, `T_Char_0`, `T_Achievement_FirstCorrect`, `T_Position_Forward` —
because those names are what the data is matched against at runtime. Read
[3.3](#33-images-are-matched-by-filename--the-most-important-rule) before renaming anything
under `Resources/`.

`Assets/TextMesh Pro/` is the exception to the prefixes. Those are Unity's own **TMP Essential
Resources** — `Fonts & Materials/`, `TMP Settings.asset`, the SDF shaders and the rest. TextMesh
Pro looks several of them up by exact string (`TMP_Settings.defaultFontAssetPath` is literally
`"Fonts & Materials/"`), so renaming them breaks text rendering. Leave that folder alone.

> **Scene names are hardcoded** in `SceneManager.LoadScene("...")` calls across the scripts, plus one
> `scene.name` comparison in `AdsManager.cs`. There are 23 such references. If you rename a scene,
> update every one of them **and** the build settings, or the game will break at runtime.

**About `Editor/`.** `Editor/QuizTemplateSetup.cs` is the setup step from
[section 1](#first-launch). Unity's package format can only carry the contents of `Assets/`, so it
cannot bring `Packages/manifest.json`, the render pipeline assignment, Active Input Handling, the
screen orientation or the Build Settings scene list. This script supplies all five from a menu, and
imports TextMesh Pro's Essential Resources as the sixth:

- **Tools ▸ Quiz Game Template ▸ Apply Project Setup** — adds the missing packages to your manifest
  (it only *adds*; a package you already have at another version is left alone), assigns
  `Settings/UniversalRP.asset`, sets Active Input Handling to Both and the orientation to Portrait,
  puts the six scenes in Build Settings with `SplashScene` first, and imports the TMP Essential
  Resources if they are missing.
- **Tools ▸ Quiz Game Template ▸ Verify Project Setup** — prints an OK/FAIL line for each of those
  six, plus an INFO line saying whether ads are real or simulated, and changes nothing.

It also switches the ads on by itself. Whenever the editor loads, it checks whether the Google Mobile
Ads plugin is in the project and adds or removes the `QGT_ADMOB` scripting define to match
(Android, iOS and Standalone). `AdsManager.cs` compiles its AdMob code only when that define is set.

It sits in its own assembly definition so that it still compiles while the gameplay scripts cannot —
which is exactly the state a freshly imported `.unitypackage` leaves the project in.

Once setup has run and the ad plugin is in the state you want, the folder has done its job, and you
can delete `Editor/` before you ship your own game. The `QGT_ADMOB` define stays in your player
settings; if you later add or remove the ad plugin, set or clear it by hand in
*Project Settings → Player → Scripting Define Symbols*.

### 2.1 Naming and language

The whole project is English: scene names, script filenames, class and enum names, method
names, serialized field names — the labels you see in the Inspector — local variables,
localization keys, the JSON data keys, and every comment in the code.

Two things are Turkish on purpose, and both should stay that way:

* **The Turkish half of the localization table.** `LocalizationManager.cs` stores every string
  twice, as `AddItem(key, tr, en)`. The middle argument is the Turkish translation the game
  shows when the player picks TR. It is data, not code. Section 8 explains how to replace or
  remove the Turkish column.
* **Nothing else.** The sample question data, the image filenames and the console messages are
  all English. File names are plain ASCII, so a country whose real name carries an accent is
  stored folded — `T_Flag_Curacao.png`, `T_Flag_SaoTomeAndPrincipe.png` — while the JSON keeps the
  proper spelling. `AssetNames.ToFileName()` does the conversion; see
  [3.3](#33-images-are-matched-by-filename--the-most-important-rule).

**Renaming a serialized field is not a free operation.** Unity binds Inspector assignments to
scene and prefab files **by field name**, so renaming a `public` field in a script silently
unassigns whatever you dragged into it — with no error and no warning; you find out when
something is null at runtime. If you rename one, either update the scene and prefab YAML in the
same edit, or add `[FormerlySerializedAs("oldName")]` from `UnityEngine.Serialization` above the
field so the existing assignment carries over. The same applies to method names bound to a
button's OnClick list, which Unity stores as plain text.

### 2.2 Script index

All 53 runtime scripts live in `Assets/QuizGameTemplate/Scripts/`. The ones you are most likely to touch:

| File | What it does |
|---|---|
| `GameManager.cs` | Core state, Inspector settings, shared fields. Split into six partial files: |
| `GameManager.Question.cs` | Question setup — builds the hints and the letter pool |
| `GameManager.Answer.cs` | Answer checking, jokers, win/lose flow |
| `GameManager.Daily.cs` | Daily Quiz mode |
| `GameManager.Ads.cs` | Ad calls (banner, interstitial, rewarded) |
| `GameManager.UI.cs` | UI updates, panels, animations |
| `LevelManager.cs` | Loads both JSON files, owns the level list and the data model classes |
| `LocalizationManager.cs` | The whole EN/TR dictionary — every string lives here |
| `AdsManager.cs` | AdMob wiring — simulated until the Google Mobile Ads plugin is installed (section 5) |
| `ShopManager.cs` | Shop, coin packs, IAP |
| `LootLockerManager.cs` | Backend calls — leaderboard, friends, player accounts |
| `AchievementManager.cs` | Achievement definitions and unlock checks |
| `AudioManager.cs` | Music / SFX / vibration, six clip slots |
| `NotificationManager.cs` | Local notification scheduling |
| `SmartGrid.cs` | Sizes the answer/letter grid to fit the screen |
| `AnswerBox.cs` | One answer slot — a single letter box |
| `LetterButton.cs` | One key on the on-screen keyboard |
| `MapPawn.cs` | The marker that walks the level map |
| `ChallengeGate.cs` | Unlock condition for Challenge mode |
| `WeeklyCountdown.cs` | Weekly countdown for the Challenge leaderboard reset |
| `LeaderboardUI.cs` | Leaderboard screen |
| `MainMenuProfile.cs` | Profile widget on the main menu |
| `ProfileManager.cs` | Profile creation, name filter |
| `ProfileDetailManager.cs` | Profile detail panel |
| `ProfileOpener.cs` | Opens the profile panel |
| `AvatarLoader.cs` | Loads the chosen avatar sprite wherever it is displayed |
| `SettingsMenu.cs` | Settings menu (sound, music, vibration, language, privacy link) |
| `LanguageButton.cs` | Language toggle button |
| `IAPButtonLanguage.cs` | Localizes the in-app purchase button labels |
| `ShopTabs.cs` | Shop tab switching |
| `ScreenAdjuster.cs` | Screen / safe-area adjuster |
| `SceneTransition.cs` | Scene transition fade |
| `TutorialManager.cs` | Tutorial flow |
| `ErrorDisplay.cs` | On-screen error / warning display (developer tool) |
| `LinkOpener.cs` | Opens external URLs (privacy policy) |
| `ScreenshotTaker.cs` | Editor screenshot tool (see §9) |

**Inspector headers are in English**, and so are the field names underneath them
(`HINT SLOT 1: TEAM / CATEGORY`, `PLAY AREA`, `AD SETTINGS`, `DEVELOPER`…). Nothing in the
Inspector needs translating.

### Game modes

**Normal** — Sequential levels in the order defined by your data file, grouped into sections of 20.
Progress is saved. When an entry has several awards listed, the one shown is picked using the
priority list `awardPriorityList` in `GameManager.cs` (first match wins; if the list is empty the
first award in the entry is used).

The level map builds itself from your data: it grows upward one button per entry, splits into
sections every `levelsPerSection` levels (default 20, on `LevelManager`), and walks the player's
chosen avatar up the path as a marker. Locked levels show a padlock. Add 500 entries to the JSON
and the map is 500 levels tall — there is nothing to lay out by hand.

**Challenge** — Timed survival mode feeding a global leaderboard. 30 seconds per question. Below 20
points the pool is limited to the first 40 entries of your data file as a warm-up; above that the
whole set is in play. Questions already asked in the current run are not repeated until the pool is
exhausted. Two things are deliberately harder than Normal mode: the team hint is drawn at random
from `allTeams` instead of showing the current team, and the award shown is picked at random
rather than by priority.

Access to Challenge mode is rationed:

| Setting | Component | Default | Effect |
|---|---|---|---|
| `dailyAttempts` | `ChallengeGate` | 3 | Free runs per day. Refills on a daily timer. |
| `leaderboardRowCount` | `LootLockerManager` | 10 | Leaderboard rows fetched. The player's own rank is shown underneath, so 10 means 11 rows. |

Players can buy extra attempts with coins or earn them by watching a rewarded ad. The leaderboard
screen has two tabs — **Global** and **Friends** — and shows each entry's avatar, name and score.

When the 30 seconds run out, the run is not over: a **Time's Up** panel offers *Watch Ad and
Continue*, which resumes the same run with the score intact, or *Main Menu* to bank the score. This
is the main rewarded-ad placement in the game, and it also feeds the ad-free counter described in
section 5.

**Daily Quiz** — One question per day, identical for every player worldwide, with no server
required. The question is derived deterministically from the calendar date, so two players opening
the app on the same day see the same question and the same hints. How it works:

- Questions are split into an "easy" pool (the first *N%* of your data file) and a "hard" pool.
  *N* is the `dailyEasyQuestionPercent` field on `GameManager` (default 20%), so the split scales
  with the size of your dataset.
- Each date has a 20% chance of being an "easy" day, decided by a date-seeded random number.
- The engine counts how many easy and hard days have occurred since 1 Jan 2024 and uses that
  count as an index into a seed-shuffled pool, so questions cycle without repeating.
- Which team and which award appear as hints are also date-seeded.

If one pool ends up empty (for example a small dataset where no entry falls into the hard pool),
the engine falls back to the other pool automatically.

---

## 3. The data system — read this first

This is the core of the template. Everything else is presentation.

### 3.1 Question database

`Assets/QuizGameTemplate/Resources/quiz_data.json` — a flat JSON array. Each object is one level:

```json
{
    "id": 1,
    "answer": "JOHN SAMPLE",
    "league": "Demo League",
    "position": "Forward",
    "nationality": "Brazil",
    "currentTeam": "Sample FC",
    "allTeams": ["Sample FC", "Placeholder United"],
    "trophies": [],
    "jerseyNumber": "9"
}
```

| Field | Type | Purpose |
|---|---|---|
| `id` | int | Unique identifier |
| `answer` | string | **The answer the player must spell.** |
| `league` | string | Category/group label. Free-form; not used for filtering. |
| `position` | string | Shown as a fallback icon when `trophies` is empty. Loads `Trophies/T_Position_<value>.png`. |
| `nationality` | string | Loads `Flags/T_Flag_<value>.png`. |
| `currentTeam` | string | Must match a `TeamName` in the team database — drives colors and city hint. |
| `allTeams` | string[] | Extra hint list (shown in the answer reveal) |
| `trophies` | string[] | Award names. Each loads `Trophies/T_Trophy_<value>.png`. Empty list is valid. |
| `jerseyNumber` | string | Loads `Jerseys/T_Jersey_<value>.png` |

Any key your JSON contains that is not declared in the `QuizEntry` class in `LevelManager.cs` is
simply ignored by JsonUtility, so extra columns in your data file are harmless.

**A leading `#` in `answer` is tolerated.** The engine strips it before the answer is used, a leftover
from the original data pipeline. You do not need it — write plain names.

> ### How answers are normalised
>
> Before an answer is shown it goes through `EditName()` in `GameManager.Question.cs`, which strips
> the leading `#`, uppercases the text, and then folds accented Latin letters to their base form:
>
> | Input | Becomes |
> |---|---|
> | `Â À Á Ä Ã Å` | `A` |
> | `Ê È É Ë` | `E` |
> | `Î Ì Í Ï` | `I` |
> | `Ô Ò Ó Õ Ø` | `O` |
> | `Û Ù Ú Ü` | `U` |
> | `Ñ` → `N`, `ß` → `S` | |
>
> So `JOSÉ` is spelled `JOSE` and `MÜLLER` is spelled `MULLER` — the player never has to find an
> accented key. **Characters the table does not recognise are passed through unchanged**, so an
> answer in Cyrillic, Greek or any other script survives intact, and its letters still appear on the
> keyboard because the answer's own characters are always added to the key pool.
>
> Two things worth knowing:
>
> - Entries whose `nationality` is `Turkey` keep the Turkish letters `Ç Ğ İ Ö Ş Ü` instead of folding
>   them, because they use a Turkish alphabet (see `customAlphabetCountry` below).
> - If your language needs different folding, extend `NormalizeForeignCharacters()` — it is a short list
>   of `if` statements right below `EditName()`.
>
> **Keyboard settings** (on `GameManager`, Inspector):
>
> | Field | Meaning |
> |---|---|
> | `keyboardLetterCount` | Number of keys shown (default 18). The answer's own letters are always included; the rest are decoys. |
> | `keyboardAlphabet` | Alphabet the **decoy** letters are drawn from. The answer's own letters are added regardless, so a non-Latin answer is always typeable even if this stays as A–Z. |
> | `customKeyboardAlphabet` | Optional second alphabet. |
> | `customAlphabetCountry` | The `nationality` value that switches to the second alphabet. **Leave this empty to disable the feature entirely** — recommended unless you need a second script. |
>
> The template ships with this set to `Turkey` because the original game needed Turkish letters for
> Turkish entries. If that does not apply to your data, clear the field.

### 3.2 Team / category database

`Assets/QuizGameTemplate/Resources/team_colors.json`:

```json
{
    "TeamName": "Sample FC",
    "Color1": "#0033CC",
    "Color2": "#FFFFFF",
    "City": "Sample City"
}
```

`TeamName` must match the `currentTeam` value in the question database exactly (the lookup is a
dictionary keyed on the trimmed name). `Color1`/`Color2` paint the two color bars shown as a hint,
and `City` is displayed as a text clue.

### 3.3 Images are matched by filename — the most important rule

The engine loads images by turning a JSON value into a file name and concatenating it onto a folder
path:

```csharp
Resources.Load<Sprite>("Flags/T_Flag_"     + AssetNames.ToFileName(nationality));
Resources.Load<Sprite>("Jerseys/T_Jersey_" + jerseyNumber);
Resources.Load<Sprite>("Trophies/T_Trophy_" + AssetNames.ToFileName(awardName));
```

| JSON value | File it looks for |
|---|---|
| `"nationality": "Brazil"` | `Flags/T_Flag_Brazil.png` |
| `"nationality": "Bosnia and Herzegovina"` | `Flags/T_Flag_BosniaAndHerzegovina.png` |
| `"nationality": "Curaçao"` | `Flags/T_Flag_Curacao.png` |
| `"jerseyNumber": "9"` | `Jerseys/T_Jersey_9.png` |
| `"trophies": ["Demo Cup"]` | `Trophies/T_Trophy_DemoCup.png` |
| `"position": "Forward"` | `Trophies/T_Position_Forward.png` |

`AssetNames.ToFileName()` (`Scripts/AssetNames.cs`) is the whole rule: it folds accents, treats
anything that is not a letter or a digit as a word break, and capitalises the first letter of each
word. Words that are already capitalised keep their capitals, so `DR Congo` becomes `DRCongo`. It
exists so file names can stay plain ASCII while your data keeps proper spelling, spaces and accents.

**If the file name does not match, the image silently disappears** — the code null-checks and hides
the element rather than throwing an error. When adding your own data:

1. Add the image to the matching folder under `Assets/QuizGameTemplate/Resources/`.
2. Name it `T_<Category>_<Name>`, where `<Name>` is what `ToFileName()` makes of your JSON value.
   In practice: drop the spaces, capitalise each word, strip the accents.

`Trophies/` serves double duty: alongside award images (`T_Trophy_*`) it holds four **position
icons** — `T_Position_Forward.png`, `T_Position_Defender.png`, `T_Position_Midfielder.png`,
`T_Position_GoalKeeper.png`. When an entry's `trophies` array is empty, the engine falls back to the
icon named after the `position` value. **Do not delete these four files** unless you also supply
awards for every entry.

### 3.4 Level ordering

**Array order = level order.** The engine does not sort or shuffle. Level 1 is the first object in
the array, level 2 the second, and so on. You control the difficulty curve entirely by ordering the
file — put your easiest questions first.

The level map groups levels into sections of `levelsPerSection` (default 20), configurable on the
`LevelManager` component.

### 3.5 Worked example — converting to a movie quiz

| Football concept | Movie quiz equivalent |
|---|---|
| `answer` — player name | Film title |
| `nationality` + `Flags/` | Country of production + flag images |
| `currentTeam` + team DB | Studio + brand colors, `City` → release year |
| `jerseyNumber` + `Jerseys/` | Genre code + genre icons |
| `trophies` + `Trophies/` | Awards won + award icons |
| `position` | Fallback category icon |

You are not required to use every hint. Leave `trophies` empty and the position-icon fallback covers
it; UI elements bound to missing images hide themselves automatically.

---

## 4. Backend setup — LootLocker (leaderboards)

The global leaderboard and friend features run on [LootLocker](https://lootlocker.com) (free tier
available). The template ships with **empty credentials** — the leaderboard will not function until
you connect your own project.

1. Create a free account and a new game at [lootlocker.com](https://lootlocker.com).
2. In the LootLocker dashboard, copy your **Game Key** (API key) and **Domain Key**.
3. In Unity, select `Assets/QuizGameTemplate/Plugins/LootLockerSDK/Resources/Config/LootLockerConfig.asset`.
4. Paste your keys into `apiKey` and `domainKey` in the Inspector.
5. In the dashboard, create a leaderboard with the key **`global_highscore`**. This is the single
   key the template uses — it is defined at `Assets/QuizGameTemplate/Scripts/LootLockerManager.cs:10`. Either name
   your leaderboard to match, or change that line to your own key.

The SDK itself (v8.1.1, MIT licence) ships inside this package, at
`Assets/QuizGameTemplate/Plugins/LootLockerSDK/`. It is included rather than referenced because
LootLocker distributes it as a Git URL, which a `.unitypackage` cannot declare as a dependency.
If you would rather track the upstream package, delete that folder first — keeping both copies
would give you the same classes twice and the project would not compile — and then add
`https://github.com/LootLocker/unity-sdk.git` through **Window ▸ Package Manager ▸ + ▸ Add package
from git URL**. Only the SDK's `Runtime` folder and its `package.json` are included; its samples
and tests are not.

If you don't want online features, you can leave the keys empty; the rest of the game works offline.

> ### ⚠ The weekly reset is not automatic — you must configure it
>
> The leaderboard screen shows a countdown reading **"Resets in 4 Days 23 Hours"**. That counter is
> **display only**: `WeeklyCountdown.cs` simply counts down to the next Monday 00:00 UTC. It does not
> clear any scores.
>
> The actual reset is a **LootLocker dashboard setting**. When you create the leaderboard, set its
> schedule to reset weekly on Monday so the countdown matches reality. If you skip this, the timer
> will keep hitting zero while the scores never change, and your players will notice.
>
> To use a different cadence (daily, monthly, never), change the schedule in LootLocker **and**
> adjust the target-day calculation in `WeeklyCountdown.cs` so the two agree.

Player accounts are created automatically as **guest sessions** — no sign-up, no email. Each install
gets its own LootLocker player ID, which is shown on the profile screen and is what the friends
system uses to add people. Until the keys are set, that ID reads `Unknown`.

### Moderating player names

Names are filtered twice, and the second layer is the one that matters after launch.

**1. On the device, before the name is accepted.** `ProfileManager.cs` holds two arrays:
`cannotContain` (substrings that may not appear anywhere in the name — role impersonation like
*admin* or *moderator*, plus profanity) and `cannotMatchExactly` (names rejected only on an exact
match). `NormalizeNameForFilter()` normalises the input first, so simple letter-swapping does not
get past it. **The shipped word list is weighted towards Turkish and English** — if you launch in
another market, extend those arrays. Add your own game's name too, so nobody impersonates staff.

**2. From the LootLocker dashboard, at any time.** This is the important one. On every launch the
game calls `GetPlayerName()` and, if the server returns a name, **overwrites the local one**
(`LootLockerManager.cs`). So renaming an abusive player in the dashboard changes their name inside
the game as well, on their next launch — you are not stuck with whatever the device stored, and you
do not need to ship an update to clean up the leaderboard.

The local name is only sent up when the server has no name for that player yet, which is what keeps
the dashboard authoritative rather than the device.

### There is also an offline score list — unused

`ScoreManager.cs` sits on the Challenge scene and records every run into a local top-score table in
`PlayerPrefs`, seeding it with the placeholder names in its `botNames` array so the table is
never empty. **Nothing displays it** — the leaderboard screen reads from LootLocker.

Treat it as a choice rather than a bug: delete it (remove the component from `ChallengeMode` and
the single `AddScore()` call in `GameManager.cs`), or wire it into your UI as an offline fallback
for players with no connection, or for shipping without LootLocker at all. If you keep the
placeholder names, edit them in the Inspector to suit your theme.

---

## 5. Ads — Google AdMob

The ad code is complete, but **the Google Mobile Ads Unity plugin is not included** — it is
Google's product, and you install it yourself. Until you do, `AdsManager` runs in
**simulation mode**: rewarded ads grant their reward at once, banners and interstitials are skipped,
and the console shows one warning saying so. Every ad-driven feature (hints, the daily reward, the
`0/5` counter) can be tested that way.

### Installing the plugin

1. Download the Google Mobile Ads Unity plugin `.unitypackage` from its official releases page,
   <https://github.com/googleads/googleads-mobile-unity/releases>. The template was built and tested
   with **v10.7.0**.
2. Import it with **Assets → Import Package → Custom Package…** and import everything. It brings
   Google's External Dependency Manager (EDM4U) with it, which resolves the Android and iOS native
   libraries. If EDM4U asks to enable Android auto-resolution or custom Gradle templates, answer
   **Yes**.
3. That is all the wiring. When Unity finishes compiling, `Editor/QuizTemplateSetup.cs` sees the
   plugin and adds the `QGT_ADMOB` scripting define, and the console says so. `AdsManager` then
   loads real ads — Google's test ads, until you change the IDs below.
   **Tools ▸ Quiz Game Template ▸ Verify Project Setup** confirms which mode is active.

If you use a different ad network instead, leave the plugin out and replace the bodies of
`ShowRewardedAd`, `ShowInterstitial` and the banner methods in `AdsManager.cs` — every other script
only calls those.

### Your own IDs

The template ships with **Google's official test ad unit IDs**. You must replace them before
publishing, or you will show test ads in production.

1. Open `Assets/QuizGameTemplate/Scripts/AdsManager.cs` and replace the three IDs:
   ```csharp
   private string bannerID       = "ca-app-pub-3940256099942544/6300978111";
   private string interstitialID = "ca-app-pub-3940256099942544/1033173712";
   private string rewardedID     = "ca-app-pub-3940256099942544/5224354917";
   ```
2. Set your AdMob **App ID** in **Assets → Google Mobile Ads → Settings**.

> **Never ship someone else's ad unit IDs.** Doing so sends revenue to another account and can get
> your AdMob account suspended for invalid traffic.

### How often ads appear

The template does not simply show an interstitial whenever it can. Two mechanics control this, and
both are tunable from the Inspector:

| Setting | Component | Default | Effect |
|---|---|---|---|
| `adFrequency` | `GameManager` | 3 | An interstitial is shown after every 3rd completed level. |
| `forcedAdSkipThreshold` | `GameManager` | 2 | Once the player has voluntarily watched this many rewarded ads in the current run, the forced interstitial is skipped. |
| `dailyTarget` | `AdsManager` | 5 | How many **rewarded** ads fill the counter shown on the main menu (`0/5`). |
| `adFreeRewardHours` | `AdsManager` | 24 | Hours of ad-free play granted when that counter fills. |
| `dailyRewardLimit` | `GameManager` | 3 | How many times a day the **Daily Reward** button pays out. Drives the `0/3` label on the button. |
| `dailyRewardAmount` | `GameManager` | 100 | Coins granted per Daily Reward watch. |

**The rewarded-ad counter is the interesting part.** Any rewarded ad the player chooses to watch —
in any scene, for coins, for a hint, for a double reward — counts toward it. Fill it and the game
turns off interstitials entirely for the next 24 hours. Nothing is ever forced: the player opts in,
and is rewarded for it with a quieter game.

It is a retention mechanic rather than a revenue-maximising one. If you would rather monetise
harder, lower `adFreeRewardHours` or raise `dailyTarget`. Players who buy the `remove_ads` IAP
skip all of this permanently.

---

## 6. In-app purchases

Unity IAP (`com.unity.purchasing`) is integrated. Products are defined in
`Assets/QuizGameTemplate/Resources/IAPProductCatalog.json`:

> Unity IAP finds this file through `Resources.Load`, so it works from any `Resources` folder — it
> does not have to sit at `Assets/Resources/`. You will see a second file there, `BillingMode.json`,
> that this package does not ship: Unity IAP writes it into your project by itself and keeps it
> up to date. Leave it alone.

| Product ID | Type | Purpose |
|---|---|---|
| `coins_1000` … `coins_10000` | Consumable | Coin packs |
| `remove_ads` | Non-consumable | Remove ads |

Create products with matching IDs in Google Play Console / App Store Connect, or rename them in the
catalog to match your own. Purchase callbacks live in `GameManager.cs`
(`PurchaseSucceeded_Gold`, `PurchaseSucceeded_NoAds`).

### Restore Purchases — already implemented

The settings menu carries a **Restore Purchases** button (localization key `Btn_Restore`). You will
not find a `RestoreTransactions()` call anywhere in the scripts, and that is not an omission: the
button uses Unity's **Codeless IAP** component (`CodelessIAPButton` with *Button Type = Restore*,
empty *Product ID*), which performs the restore itself.

This matters for review. **Apple rejects apps that sell non-consumables without a restore
mechanism**, and `remove_ads` is a non-consumable. Google Play re-queries owned products
automatically on reinstall, so the button is belt-and-braces there, but reviewers still expect to
see it. Keep it.

If you add non-consumables of your own, they are covered by the same button — nothing to wire up.
If you delete the settings menu, move the button somewhere else rather than dropping it.

---

## 7. Audio — bring your own

The audio system is fully wired but **ships without sound files**, so you can drop in audio you
hold the rights to rather than inherit licences you cannot verify.

`AudioManager` (`Assets/QuizGameTemplate/Scripts/AudioManager.cs`) exposes six labelled slots in the Inspector:

| Slot | Plays when |
|---|---|
| `backgroundMusic` | Background music (looped, volume 0.4) |
| `clickSound` | Any button tap |
| `winSound` | Correct answer / level won |
| `wrongLetterSound` | Wrong letter (also triggers vibration) |
| `gameOverSound` | Game over (also triggers vibration) |
| `coinSound` | Coins earned or spent |

To add sound: drop your audio files anywhere under `Assets/`, select the `AudioManager` object in
the scene, and drag each clip into its slot. Nothing else to configure — music/SFX toggles,
volume, mute persistence and vibration are already implemented.

Empty slots are handled safely: every playback call is null-checked, so the game runs silently
without errors until you add files.

**Where to find freely licensed audio:** [Kenney.nl](https://kenney.nl) publishes game audio packs
under CC0 (public domain, no attribution required) — the safest option if you intend to resell your
game. [Freesound](https://freesound.org) also works if you filter for CC0.

## 8. Localization (English / Turkish)

`Assets/QuizGameTemplate/Scripts/LocalizationManager.cs` holds the full string table as `AddItem(key, tr, en)` calls
inside `LoadHardcodedDictionary()`. To add or change a string, edit that method.

- Language is auto-detected on first launch: Turkish device → `TR`, everything else → `EN`.
  The choice is saved to `PlayerPrefs` and can be changed in-game from the settings menu.
- `GetText(key)` **returns the key itself when no translation exists**, so untranslated strings
  degrade gracefully instead of showing blanks — translating your data is optional.
- **Values from your data files are translated too.** The `league`, `nationality` and `City` fields are all
  passed through `GetText()` before display. Add an entry per value you want localized; see the
  "SAMPLE DATA LABELS" block for the pattern.
- **69 country names ship pre-translated (EN/TR)**, matching the flag images in `Resources/Flags/`.
  If your quiz involves nationalities or geography, this is ready to use as-is.

To add a third language, add a field to the `LocalizationItem` class and extend `GetText()`.

> ### ⚠ Unity Analytics starts itself — decide whether you want that
>
> `LocalizationManager.Start()` calls `UnityServices.InitializeAsync()` and then
> `AnalyticsService.Instance.StartDataCollection()`. Out of the box this fails harmlessly, because
> the project is not linked to a Unity organization — that is the `Unity Analytics could not start`
> line in [section 10](#10-troubleshooting).
>
> **The moment you link your own Unity project, it succeeds** and your game begins collecting
> analytics on launch, with no consent prompt of any kind. If you ship in the EU, the UK or any
> other jurisdiction with consent requirements, that is your obligation to handle, not something
> this template does for you.
>
> If you do not want analytics, delete those two lines from `LocalizationManager.Start()` and
> remove `com.unity.services.analytics` from `Packages/manifest.json`. Nothing else in the template
> uses them. If you do want analytics, gate `StartDataCollection()` behind your own consent flow —
> AdMob's UMP consent form, which comes with the Google Mobile Ads plugin (section 5), is the usual place to hang it.

> ### ⚠ A second place can define strings — and it wins
>
> `BuildDictionary()` loads the entries from `LoadHardcodedDictionary()` **first**, then overwrites
> them with the `Dictionary List` on the **`Assets/QuizGameTemplate/Prefabs/LocalizationManager` prefab**.
>
> **That list ships empty**, so the script is the single source of truth out of the box. But if you
> add entries there, they silently win. It is a convenient way to override one string without
> touching code — just remember it, because if you edit a string in the script and nothing changes
> in game, an Inspector entry with the same key is the reason.
>
> ### ⚠ Keys are case-sensitive
>
> `btn_yes` and `Btn_Yes` are two different keys. Since a missing key renders as its own name, a
> capitalisation slip shows up in game as raw text like `btn_sec` on a button — not as a blank or
> an error. If you see a key name on screen, that is what happened.

---

## 9. Branding, package name and building

### Rename the app

1. **Edit → Project Settings → Player**
   - `Product Name` — your app's display name
   - `Company Name`
   - **Other Settings → Identification → Package Name** (e.g. `com.yourcompany.yourgame`)
2. Replace the app icons under **Player → Icon**.
3. Search the scripts for leftover template strings you want to change — notably the notification
   channel ID in `Assets/QuizGameTemplate/Scripts/NotificationManager.cs` and the website URL in
   `Assets/QuizGameTemplate/Scripts/SettingsMenu.cs` and `LinkOpener.cs`.

### Android build

1. **File → Build Profiles** → select **Android** → **Switch Platform**.
2. **Player Settings → Publishing Settings → Keystore Manager** → create a new keystore, set the
   passwords, and select it. Keep this file safe — Google Play requires the same key for every
   future update.
3. Set **Minimum API Level** as required by your target audience.
4. To publish, tick **Build App Bundle (Google Play)** and build an `.aab`.

### iOS build

1. Switch platform to **iOS**.
2. Set your **Bundle Identifier** and **Signing Team ID** in Player Settings.
3. Build to generate an Xcode project, then open it and archive from Xcode.

> The project must be built on macOS with Xcode to produce an iOS binary.

> **iOS support is written, but never verified.** This template was developed, shipped and
> maintained as an Android game.
>
> What is in place: `NotificationManager` has complete iOS code paths, including the
> `AuthorizationRequest` permission flow, scheduling and cancellation. `AudioManager.TriggerHaptics()` is
> guarded for both platforms. The Google Mobile Ads plugin (section 5) brings its iOS native plugin, its
> `SKAdNetworkItems` list and the CocoaPods resolver. Unity IAP and LootLocker both
> support iOS. There is no Android-only branch left without an iOS counterpart.
>
> What has never happened: an actual iOS build. No Xcode archive, no device run, no App Store IAP
> product tested, no iOS ad mediation tested. The steps above are the standard Unity iOS flow, not
> a recipe verified on this project — budget time for the usual first-build work (signing,
> CocoaPods, the ATT prompt, and AdMob's `GADApplicationIdentifier` entry in `Info.plist`).

---

## 10. Troubleshooting

### Console warnings on a fresh install — this is expected

The template ships with **no credentials of any kind**, by design. Until you connect your own
services (sections 4–6) and install the ad plugin (section 5), the console will show warnings on
first play. **Expect warnings of these kinds, some of them repeated, and zero errors** — that is
the normal, verified state of a fresh import. All of them are configuration messages, not defects:

| Message | Goes away when you… |
|---|---|
| `Unity Services could not be initialized` | Link the project to your own Unity organization: **Edit → Project Settings → Services** |
| `Unity Analytics could not start` | Same — Analytics rides on the Unity Services link above |
| `InAppPurchasing: IStoreService.Connect called without a callback…` | Link Unity Services and create the matching IAP products (section 6) |
| `LootLocker is not configured…` | Enter your LootLocker API key (section 4) |
| `Leaderboard could not be fetched…`, `Score submission failed…`, `No player exists with that ID` | Same — they are the LootLocker calls that the missing key turns off |
| `[AdsManager] Google Mobile Ads is not installed, so ads are simulated…` | Install the Google Mobile Ads plugin (section 5) |

If you see an *error* rather than a warning, something really is wrong — that is worth reporting.

The core game — all three modes, levels, hints, achievements, localization — runs fine without any
of this configured.

### The game requires an internet connection — by design

`ConnectionManager` checks `Application.internetReachability` on startup. With no connection it
shows a full-screen blocking panel **and sets `Time.timeScale = 0`**, so the game cannot be played
offline until the player taps Retry.

The reason is the Daily Quiz: `TimeManager` fetches the real date from a time server instead of
trusting the device clock, so a player cannot change their phone's date to farm daily rewards.

**Decide whether you want this.** If your quiz has no daily mode and no leaderboard, locking out
offline players costs you sessions for no benefit. Two ways to relax it:

- **Advisory only** — delete the `Time.timeScale = isOn ? 0 : 1;` line in `ConnectionManager.SetOfflinePanel()`.
  The warning still appears, but the game stays playable.
- **Fully offline** — remove the `ConnectionManager` component from the scene. `TimeManager` then
  falls back to `DateTime.Now`, and the daily quiz works off the device clock (and becomes
  farmable — an acceptable trade for many quiz games).

### Other issues

| Symptom | Cause / fix |
|---|---|
| Shop buttons read `1.000 GOLD - ...` | Expected. The `...` is a placeholder for the price, which only arrives once the app is connected to a real store. See section 6. |
| Player ID shows `ID: #Unknown` | Expected until LootLocker connects — see section 4. |
| A flag, jersey or award image doesn't appear | The filename in `Resources/` doesn't exactly match the JSON value. The engine hides missing sprites silently. |
| Team colors are gray and the city shows `???` | The `currentTeam` value has no matching `TeamName` in `team_colors.json`. |
| Leaderboard is empty or errors | LootLocker `apiKey`/`domainKey` not set — see section 4. |
| Package Manager can't resolve a package on import | No internet on first import — the nine required Unity packages are downloaded from the Unity registry. Reconnect and run **Tools ▸ Quiz Game Template ▸ Apply Project Setup**. |
| **Back button and screenshot key do nothing** | *Active Input Handling* has been changed to **Input System Package (New)** on its own. Set Project Settings → Player → **Active Input Handling** back to **Both** (the value this template ships with) and restart the editor. See section 1. |
| Only a few levels exist | The sample database ships with a handful of placeholder entries. Add your own — one array entry per level. |

---

## 11. What's included

- 6 scenes, 54 C# scripts, 12 prefabs, complete UI
- 3 game modes (Normal, Challenge, Daily)
- Global leaderboard + friend system (LootLocker)
- Player-name moderation: on-device profanity filter plus dashboard-side renaming that overrides
  the local name on next launch
- Achievements system
- Player profiles with unlockable pixel-art avatars
- In-app purchases (coin packs, remove ads)
- AdMob-ready ad system with a rewarded-ad hint system (plugin installed separately, simulated until then)
- Local notifications with weekly reminders
- Full EN/TR localization system
- Tutorial flow
- JSON-driven content — no code changes needed to swap the entire question set

**Not included:** audio files and the original question database. The audio slots are named and
null-safe, so the game runs silently until you add your own clips — see [section 7](#7-audio--bring-your-own)
for CC0 sources. A small sample dataset ships with the package so the game is playable on first
launch; [section 3](#3-the-data-system--read-this-first) explains the format.

---

## 12. Customization reference — what you can change, and where

This section is the answer to "what am I allowed to edit, and what happens if I do".
Everything listed in 12.1–12.5 is meant to be changed. Section 12.6 lists the few things that will
silently break the project if you rename them, and how to change them safely anyway.

**One rule applies to the whole reference below.** Most tunables are `public` fields on a component
that already exists in a scene or prefab. Unity stores the *serialized* value, so the number you see
in the Inspector wins and editing the C# line changes nothing for that object. Use the line
reference to find the field, then change the value **in the Inspector**. Fields marked *code-only*
have no Inspector entry, so there the line is the value. In the package as shipped, every serialized
value matches the C# default listed here, so the two agree until you change one.

### 12.1 Change these before you ship

| What | Where | Ships as |
|---|---|---|
| LootLocker API key + domain key | `Assets/QuizGameTemplate/Plugins/LootLockerSDK/Resources/Config/LootLockerConfig.asset`, or the LootLocker settings window | **empty** — leaderboards and friends stay offline until set (section 4) |
| AdMob app ID | *Assets → Google Mobile Ads → Settings*, after installing the plugin (section 5) | Google's public **test** app ID, which the plugin sets on install |
| AdMob banner / interstitial / rewarded unit IDs | `Scripts/AdsManager.cs` lines **37, 38, 39** — *code-only* | Google's public **test** unit IDs |
| IAP product IDs | `Assets/QuizGameTemplate/Resources/IAPProductCatalog.json` | `coins_1000`, `coins_2500`, `coins_5000`, `coins_10000`, `remove_ads` — these must match the products you create in Google Play / App Store (section 6) |
| Share / rate link | `Scripts/GameManager.cs` line **126** (`shareLink`, on **GameManager** in `GameScene` and `MainMenu`) | a `com.yourcompany.quiztemplate` placeholder URL |
| Package name, app name, icon, signing | *Project Settings → Player* | placeholder values (section 9) |

### 12.2 Content — the questions and the artwork that goes with them

| What | Where | Notes |
|---|---|---|
| Question database | `Resources/quiz_data.json` | 4 placeholder entries. One entry per level, in order. Field list in section 3.1 |
| Category / team database | `Resources/team_colors.json` | 3 placeholder entries — colours and city per category. Section 3.2 |
| Answer-clue images | `Resources/Jerseys/` (99 files) | matched **by filename**, not by reference — read 3.3 before renaming any of them |
| Country / origin flags | `Resources/Flags/` (237 files) | same filename rule |
| Unlockable avatars | `Resources/Characters/` (20 files) | the count must match the price list in 12.3 |
| Achievement medals | `Resources/Achievements/` (13 files) | file name is `T_Achievement_` plus the PascalCase form of the achievement `id` in `Scripts/AchievementManager.cs`, lines 74–92 |
| Category icons | `Resources/Trophies/` (4 files) | `T_Position_Forward`, `T_Position_Defender`, `T_Position_Midfielder`, `T_Position_GoalKeeper` — the four positions the sample data refers to |

Swapping the whole topic — football to films, history, geography — is a data-and-images job with no
code changes. Section 3.5 walks through one conversion end to end.

### 12.3 Gameplay balance — the exact lines

All of these are `public`, so change them in the Inspector on the component named in the last
column; the line number tells you which field you are looking at.

| What it controls | File | Line | Ships as | Component lives on |
|---|---|---|---|---|
| Interstitial ad every N levels | `Scripts/GameManager.cs` | 77 | `3` | **GameManager** — `GameScene`, `MainMenu` |
| Share of easy questions in the daily quiz (%) | `Scripts/GameManager.cs` | 113 | `20` | same |
| Cost of one revealed letter, in gold | `Scripts/GameManager.cs` | 105 | `50` | *code-only* — `private`, no Inspector field |
| Rewarded ads a player may watch per day | `Scripts/GameManager.Ads.cs` | 16 | `3` | **GameManager** |
| Gold paid per rewarded ad | `Scripts/GameManager.Ads.cs` | 17 | `100` | same |
| Failed ad attempts before a forced ad is skipped | `Scripts/GameManager.Ads.cs` | 20 | `2` | same |
| Fallback avatar price, used when the list below is short | `Scripts/ShopManager.cs` | 14 | `500` | **ShopManager** — `MainMenu` |
| Per-avatar price overrides | `Scripts/ShopManager.cs` | 17 | Inspector list, 20 entries (500 … 5000) | same — one entry per avatar, in folder order |
| Ads to watch for the daily bonus | `Scripts/AdsManager.cs` | 36 | `5` | **AdsManager** — `MainMenu` |
| Length of the ad-free reward, in hours | `Scripts/AdsManager.cs` | 39 | `24` | same |
| Leaderboard rows fetched per request | `Scripts/LootLockerManager.cs` | 125 | `10` | **LootLockerManager** — `MainMenu` |
| Levels per map section | `Scripts/LevelManager.cs` | 30 | `20` | **LevelManager** — `LevelManager.prefab` |
| Level the player starts on | `Scripts/LevelManager.cs` | 40 | `1` | same |
| Gold for one extra life in Challenge mode | `Scripts/ChallengeGate.cs` | 11 | `250` | **ChallengeGate** — `ChallengeMode` |

### 12.4 Look and feel

| What | Where |
|---|---|
| All UI artwork | `Textures/` — 46 PNGs, `T_` prefixed and PascalCase (`T_CloseIcon`, `T_ArrowLeft`, `T_MusicOn`, `T_LevelLocked`, …). Replace a file, keep its name, and every reference survives |
| Font | `Fonts/F_Jersey10Regular_SDF.asset`, with the `.ttf` and its OFL licence beside it. Replace it, or add your own TMP font asset and reassign it on the text objects |
| Button animation | `Animations/A_ButtonStartBackground.anim` + `AC_ButtonStartBackground.controller`, and `A_ButtonBlink.anim` + `AC_ButtonStart.controller` |
| Screen layout | the 6 scenes in `Scenes/` — ordinary Unity UI, edit freely |
| Reusable UI pieces | the 12 prefabs in `Prefabs/` |
| Render settings | `Settings/UniversalRP.asset`, `Settings/Renderer2D.asset` |

### 12.5 Text and language

Every visible string lives in one method — `LoadHardcodedDictionary()` in
`Scripts/LocalizationManager.cs`, **lines 147–452** — as 220 calls of this shape:

```csharp
AddItem("Btn_Play", "OYNA", "PLAY");   // AddItem(key, Turkish, English)
```

- **To change wording:** edit the second (Turkish) or third (English) argument. Nothing else refers
  to them, so this is always safe.
- **To rename a key:** the key is also stored on the `LocalizeUI` component of each text object
  (as `key:` in the scene file, 58 of them) and in 45 `GetText("…")` calls. Change all three, or the UI falls
  back to printing the key itself.
- **To add a language:** extend the `AddItem()` signature and add a case to `GetText()`. The file's
  header comment, lines 9–34, documents the whole mechanism.
- Country names, positions and category names are deliberately **not** all in this list: they are
  read from your JSON and looked up as keys, so add an entry only for the ones you want translated.

The language selector is in-game — Settings → language button — and the choice is saved to
`PlayerPrefs`.

### 12.6 Change these carefully, or not at all

| Thing | Why it is fragile | The safe way |
|---|---|---|
| **Serialized field names** — the 257 `public` names such as `backgroundMusic` or `defaultPrice` | Unity matches saved data to fields **by name**. Renaming one silently empties every Inspector slot that used it: no compiler error, no console warning | Put `[FormerlySerializedAs("oldName")]` (from `UnityEngine.Serialization`) above the field, rename it, open every scene and prefab that uses it once so Unity rewrites the data, then delete the attribute. See section 2.1 |
| **Method names wired to buttons** | An `OnClick` entry stores the method **name as text**. Renaming the method leaves the button pointing at nothing — again with no error | Rename in code, then reassign that button's `OnClick` entry in the Inspector |
| **Image filenames under `Resources/`** | They are loaded by name at runtime, never by reference. A typo gives you a blank image and a console warning, not an error | Section 3.3 |
| **`.meta` files** | They carry the GUIDs that hold every reference in the project together | Never delete or hand-edit them. Move and rename assets **from inside Unity**, which keeps each `.meta` with its file |
| `QuizGameTemplate/Plugins/` (LootLockerSDK, NativeShare) and `Assets/TextMesh Pro/` | Third-party SDKs, sitting at the paths their own importers require | Update them through their own importer or package, and leave the folder locations alone |

---

## Support

For questions about this template, contact us through our profile page on the marketplace where you
purchased it.
