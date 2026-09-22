# Crossnumbers — Monthly Pass

**Target:** Unity 2022 LTS+ case-study build  
**Status:** New case-study feature, not part of the reverse-engineered original game  
**Companion files:** `GAME_SPEC.md`, `IMPLEMENTATION_SPEC.md`, `level_data.md`

> For Monthly Pass behavior, this file is the feature authority. It does not change the core Crossnumbers gameplay.

---

# 1. Feature summary

The Monthly Pass is a **30-day, 30-step reward track** layered on top of normal level completion.

Core rules:

- Two lanes: **Free** and **Premium**.
- Progress unit: **Pass Key**.
- Only first-time level completion grants Pass Keys.
- Case-study rate: **5 Pass Keys per completed level**.
- Six levels × 5 keys = full **30/30** completion.
- Rewards are manually claimed; `Claim All` is available.
- Premium can be unlocked at any time and retroactively exposes already-reached Premium rewards.
- Progress, claims, Premium ownership, and season timing persist between sessions.
- Premium unlock is mocked; no real IAP is required.
- Main rewards are **Hints** plus a few lightweight cosmetics.
- The pass uses a rolling 30-day timer beginning on first initialization so the reviewer can always test it.

The pass does **not** add lives, fail states, coins, premium currency, quests, streaks, ads, loot boxes, team systems, paid progress skips, replay farming, or extra paid tiers.

---

# 2. Competitor research and transferable patterns

The closest references are casual puzzle games whose passes progress through ordinary level play.

| Game | Relevant pass pattern | Used in Crossnumbers? |
|---|---|---|
| **Royal Match** | 30-step monthly pass, keys from level wins, Free/Premium lanes, post-track bank | 30 steps, level-win progress, Free/Premium adopted; Bonus Bank rejected |
| **Candy Crush Saga** | Free/Gold tracks, level-win progress, late premium purchase gives access to earlier reached premium rewards | Retroactive Premium adopted |
| **Gardenscapes** | Season points from normal levels, premium track, temporary pass bonuses | Core-loop integration adopted; lives/social bonuses rejected |
| **Homescapes** | 30 stages, progress from regular level completion, Premium remains separate from progression | 30-stage structure and separation of progress/Premium adopted |
| **Match Factory** | Stars from wins, Free/special rewards, manual claiming after unlock | Explicit `Locked → Claimable → Claimed` state adopted |
| **Toon Blast** | Level-win badges, multiple reward tracks, post-track rewards | Simple win-based progression adopted; multiple paid tiers rejected |
| **Clash of Clans** | Recent pass redesign emphasizes simpler progression and catch-up friendliness | Principle adopted: no daily obligations or missed-day punishment |

## Design conclusion

The common transferable structure is:

```text
Time-limited season
+ normal level completion
+ linear progress
+ Free/Premium lanes
+ claimable milestone rewards
+ persistent season state
```

Crossnumbers should use that structure without importing systems it does not already support.

### Features deliberately not transferred

- Lives or life-cap bonuses: Crossnumbers has no lives.
- Extra moves: there is no move limit.
- Team gifts: there is no team system.
- Daily/weekly pass tasks: they create a second gameplay loop unnecessarily.
- Coins/premium currency: there is no existing economy.
- Bonus Bank/post-track rewards: there is no content after Level 6 and no replay loop.
- Multiple paid tiers: unnecessary for a small case-study implementation.

---

# 3. Season structure

## 3.1 Duration

```text
30 days
```

For the case-study build, the season starts when the pass is first initialized:

```text
seasonStartUtc = DateTime.UtcNow
seasonEndUtc   = seasonStartUtc + 30 days
```

Use UTC and persist both timestamps.

A production game would normally receive fixed season dates from live configuration, but a rolling demo season prevents the submitted build from being expired when reviewed.

## 3.2 Season ID

```text
crossnumbers_monthly_pass_demo_v1
```

Premium ownership and claim state belong only to this season ID.

---

# 4. Pass progression

## 4.1 Pass Keys

A Pass Key is a non-spendable progress unit.

It cannot be lost or used anywhere else.

## 4.2 Progress source

Only first-time level completion grants progress.

```text
Level completed for first time → +5 Pass Keys
```

Restarting, continuing, opening, or partially solving a level grants nothing.

Use `rewardedLevelIds` to prevent duplicate grants.

## 4.3 Six-level progression

| Level completed | Keys gained | Total keys | Newly reached steps |
|---|---:|---:|---|
| Level 1 | +5 | 5 | 1–5 |
| Level 2 | +5 | 10 | 6–10 |
| Level 3 | +5 | 15 | 11–15 |
| Level 4 | +5 | 20 | 16–20 |
| Level 5 | +5 | 25 | 21–25 |
| Level 6 | +5 | 30 | 26–30 |

This compressed rate exists only so the complete pass can be demonstrated in the six-level build.

## 4.4 Step thresholds

There are exactly 30 steps:

```text
Step 1  = 1 key
Step 2  = 2 keys
...
Step 30 = 30 keys
```

Therefore each completed level unlocks five pass steps.

---

# 5. Rewards

Crossnumbers has no general economy, so the pass uses only:

1. **Hint charges**
2. **Cosmetics**

## 5.1 Hint balance

For this case-study feature, Hint charges are a persistent player-level balance shared across levels and sessions.

Recommended starting value:

```text
hintBalance = 5
```

Using a Hint:

```text
hintBalance -= 1
```

A pass Hint reward adds to that same balance.

Restarting a level does not refund consumed Hints.

This is a Monthly Pass design decision needed to make Hint rewards meaningful; it is not claimed as observed original-game behavior.

## 5.2 Cosmetics

Keep cosmetics lightweight and visual only:

```text
premium_bank_frame_gold
premium_confetti_stars
premium_board_frame_sapphire
free_season_emblem_silver
premium_season_emblem_gold
```

Cosmetics must not modify semantic gameplay colors:

- locked cells stay blue;
- player cells stay white;
- candidates stay orange;
- valid placement stays green;
- contradiction stays red/pink.

Newly unlocked cosmetics may auto-equip to avoid building a cosmetic inventory screen.

---

# 6. Reward track

`—` means no reward on that lane for that step.

| Step | Free | Premium |
|---:|---|---|
| 1 | +1 Hint | — |
| 2 | — | +1 Hint |
| 3 | +1 Hint | — |
| 4 | — | +1 Hint |
| 5 | +1 Hint | +1 Hint + Gold Bank Frame |
| 6 | — | +1 Hint |
| 7 | +1 Hint | — |
| 8 | — | +1 Hint |
| 9 | +1 Hint | — |
| 10 | — | +1 Hint + Star Confetti |
| 11 | +1 Hint | — |
| 12 | — | +1 Hint |
| 13 | +1 Hint | — |
| 14 | — | +1 Hint |
| 15 | +1 Hint | +1 Hint |
| 16 | — | +1 Hint |
| 17 | +1 Hint | — |
| 18 | — | +1 Hint |
| 19 | +1 Hint | — |
| 20 | — | +1 Hint + Sapphire Board Frame |
| 21 | +1 Hint | — |
| 22 | — | +1 Hint |
| 23 | +1 Hint | — |
| 24 | — | +1 Hint |
| 25 | +1 Hint | +1 Hint |
| 26 | — | +1 Hint |
| 27 | +1 Hint | — |
| 28 | — | +1 Hint |
| 29 | +1 Hint | — |
| 30 | +2 Hints + Silver Emblem | +2 Hints + Gold Emblem |

### Full-track totals

Free player:

```text
17 Hint charges
Silver completion emblem
```

Premium player receives Free + Premium rewards:

```text
34 total Hint charges
Gold bank frame
Star confetti
Sapphire board frame
Silver completion emblem
Gold completion emblem
```

---

# 7. Premium behavior

## 7.1 Premium ownership

Default:

```text
premiumUnlocked = false
```

Premium is season-specific and does not carry into a future `seasonId`.

## 7.2 Mock purchase

Use a button such as:

```text
UNLOCK PREMIUM
Mock Purchase
```

A presentation-only `$4.99` label is acceptable if clearly marked as a mock.

Do not implement Unity IAP, receipts, store validation, price localization, or restore purchase.

## 7.3 Late Premium unlock

Premium may be activated at any point during the active season.

Example:

```text
Player has 20/30 keys
→ unlock Premium
→ Premium rewards from Steps 1–20 become claimable
→ Steps 21–30 remain locked
```

Premium never grants progress or unlocks future steps.

## 7.4 Unlock confirmation

After activation:

```text
PREMIUM UNLOCKED
8 rewards are ready to claim.

[CLAIM ALL]
[VIEW TRACK]
```

Do not automatically claim the rewards on purchase; let the player see the unlock moment and claim them with one tap.

---

# 8. Reward claiming

Each reward cell has one of four states:

```text
Locked
Claimable
Claimed
PremiumLocked
```

### Locked

Required key threshold not reached.

### Claimable

Threshold reached and entitlement condition satisfied.

### Claimed

Reward already applied.

### PremiumLocked

Step reached, but Premium is not active.

## 8.1 Individual claim

Claim flow:

```text
verify threshold
→ verify entitlement
→ verify not already claimed
→ apply reward
→ record claimed reward ID
→ save immediately
→ refresh UI
```

## 8.2 Claim All

Show `CLAIM ALL` when at least two rewards are currently claimable.

It claims:

- all reached unclaimed Free rewards;
- all reached unclaimed Premium rewards if Premium is active.

It never claims future rewards.

---

# 9. Expiration

Pass progression is active only while:

```text
DateTime.UtcNow < seasonEndUtc
```

On initialization, focus regain, Main entry, or Pass opening, call:

```text
CheckExpirationAndSettle()
```

When the season has expired, automatically grant all **reached but unclaimed** rewards the player was entitled to:

- reached Free rewards always;
- reached Premium rewards only if Premium had been unlocked.

Then:

```text
seasonSettled = true
```

and save.

After expiration, the track remains viewable in read-only `Ended` state.

Do not automatically create another season in the case-study build.

---

# 10. Main UI

Add a Monthly Pass entry to the existing Main screen without redesigning the base UI.

Recommended card:

```text
MONTHLY PASS
12 / 30
24d 13h left
```

If rewards are claimable, show a small badge containing the number of claimable rewards, capped visually at `9+`.

---

# 11. Monthly Pass panel

Use a full-screen panel in the Main scene.

Portrait layout:

```text
┌─────────────────────────────┐
│ ←   MONTHLY PASS     24d    │
│     Number Trail            │
│                             │
│ Keys  ███████      12 / 30  │
│                             │
│ FREE       STEP      PREMIUM│
│ reward       1       reward │
│ reward       2       reward │
│ reward       3       reward │
│ ... vertical ScrollRect ... │
│ reward      30       reward │
│                             │
│ [CLAIM ALL] [UNLOCK PREMIUM]│
└─────────────────────────────┘
```

Use a vertical `ScrollRect` with Free rewards on the left, numbered step nodes in the center, and Premium rewards on the right.

Reached nodes are filled; future nodes are dimmed.

On first opening after progress, scroll near the newest reached step.

Premium uses restrained gold accents. Avoid fake discounts or unrelated purchase pressure.

---

# 12. Level-completion integration

Monthly Pass progress is granted only after the puzzle is confirmed solved.

```text
Puzzle solved
→ save puzzle completion
→ mark level completed
→ GrantLevelCompletion(levelId)
→ save
→ show Congratulations
→ show pass progress result
```

Add a compact strip to the win overlay:

```text
MONTHLY PASS
+5 Keys
10 → 15 / 30
5 steps unlocked
[VIEW PASS]
```

`VIEW PASS` may return to Main and automatically open the pass panel. No cross-scene UI framework is required.

---

# 13. First-time introduction

Introduce the pass only after Level 1 has taught the player the core game.

Suggested one-time popup:

```text
MONTHLY PASS

Complete levels to earn Pass Keys.
Unlock rewards on the Free track.
Activate Premium for extra rewards.

[VIEW PASS]
[LATER]
```

Persist:

```text
passIntroSeen = true
```

---

# 14. Data model

## 14.1 Reward types

```csharp
public enum PassRewardType
{
    None,
    Hint,
    Cosmetic
}
```

## 14.2 Reward definition

```csharp
[Serializable]
public class PassRewardDefinition
{
    public string rewardId;
    public PassRewardType type;
    public int amount;
    public string cosmeticId;
    public string displayName;
    public Sprite icon;
}
```

## 14.3 Step definition

```csharp
[Serializable]
public class PassStepDefinition
{
    public string stepId;
    public int stepNumber;
    public int requiredKeys;
    public PassRewardDefinition freeReward;
    public PassRewardDefinition premiumReward;
}
```

## 14.4 Config

```csharp
[CreateAssetMenu]
public class MonthlyPassConfig : ScriptableObject
{
    public string seasonId = "crossnumbers_monthly_pass_demo_v1";
    public int durationDays = 30;
    public int keysPerCompletedLevel = 5;

    public string title = "Monthly Pass";
    public string subtitle = "Number Trail";
    public string mockPriceLabel = "$4.99";

    public List<PassStepDefinition> steps;
}
```

Author exactly 30 steps.

---

# 15. Save format

Extend the root save with shared player meta and pass state.

```text
SaveDataV1
  int schemaVersion
  int currentLevelIndex
  LevelProgressSave currentLevelProgress
  SettingsSave settings
  PlayerMetaSave playerMeta
  MonthlyPassSave monthlyPass
```

## PlayerMetaSave

```text
int hintBalance = 5
List<string> unlockedCosmeticIds
```

## MonthlyPassSave

```text
string seasonId
long seasonStartUtcTicks
long seasonEndUtcTicks

int progressKeys
bool premiumUnlocked
bool seasonSettled
bool passIntroSeen

List<string> claimedFreeRewardIds
List<string> claimedPremiumRewardIds
List<string> rewardedLevelIds
```

Do not save `currentStep`; derive it from `progressKeys`.

Save immediately after:

- initializing the pass;
- granting level-completion keys;
- claiming rewards;
- unlocking Premium;
- consuming a Hint;
- settling an expired season.

---

# 16. MonthlyPassService

Recommended public API:

```text
InitializeIfNeeded()

bool IsActive()
bool IsExpired()
TimeSpan GetTimeRemaining()

int GetProgressKeys()
int GetReachedStep()
int GetClaimableCount()

bool IsPremiumUnlocked()

PassProgressResult GrantLevelCompletion(string levelId)

bool ClaimFree(string rewardId)
bool ClaimPremium(string rewardId)
int ClaimAll()

void MockUnlockPremium()
void CheckExpirationAndSettle()
```

## GrantLevelCompletion

```text
if pass inactive:
    return no change

if rewardedLevelIds contains levelId:
    return no change

rewardedLevelIds.Add(levelId)
progressKeys = min(progressKeys + 5, 30)
save
```

Return enough data for the win animation:

```text
oldKeys
newKeys
newlyReachedSteps
newClaimableRewardCount
```

## Claim

```text
if already claimed:
    fail

if step threshold not reached:
    fail

if Premium reward and Premium is locked:
    fail

apply reward
record reward ID
save
```

## ApplyReward

```text
Hint:
    hintBalance += amount

Cosmetic:
    add cosmeticId if not already unlocked
```

All mutations must be idempotent.

---

# 17. Recommended prefab/UI hierarchy

```text
Main
└── Canvas
    └── SafeAreaRoot
        ├── MainPanel
        │   ├── Logo
        │   ├── SettingsButton
        │   ├── MonthlyPassButton
        │   │   ├── ProgressText
        │   │   ├── TimeRemainingText
        │   │   └── ClaimBadge
        │   └── NewGameButton
        │
        ├── SettingsPanel
        ├── ContinueRestartPanel
        ├── MonthlyPassPanel
        │   ├── Header
        │   ├── ProgressHeader
        │   ├── ScrollView
        │   │   └── Content
        │   │       └── PassMilestone x30
        │   ├── ClaimAllButton
        │   └── PremiumButton
        ├── PremiumUnlockModal
        └── PassIntroModal
```

`PassMilestone.prefab`:

```text
PassMilestone
├── FreeReward
├── StepNode
└── PremiumReward
```

Gameplay addition:

```text
WinOverlay
└── PassProgressStrip
```

---

# 18. Animation

Keep animation minimal:

- `+5 Keys` appears on level completion.
- Progress bar fills from old to new value.
- Newly reached nodes pop briefly.
- Claimed reward icon pops and receives a checkmark.
- Premium unlock uses one short gold sweep.

Do not use long or looping attention animations.

---

# 19. WebGL considerations

Use only local/offline systems:

- ScriptableObject config;
- JSON save;
- `PlayerPrefs`/browser-backed persistence;
- `DateTime.UtcNow`;
- TextMeshPro;
- ScrollRect.

Do not add billing SDKs, backend services, remote config, accounts, push notifications, or web-time APIs.

The client clock is authoritative. Clock manipulation is acceptable for this case study.

On browser focus regain:

```text
CheckExpirationAndSettle()
RefreshPassUI()
```

Do not rely only on `OnApplicationQuit` for saving.

---

# 20. Critical edge cases

1. **Duplicate level-complete callback**  
   `rewardedLevelIds` prevents duplicate keys.

2. **Premium unlocked late**  
   Earlier reached Premium rewards become claimable; future rewards remain locked.

3. **App closes after key grant**  
   Grant is saved immediately before UI animation.

4. **Reward clicked twice**  
   Claimed reward ID prevents duplicate application.

5. **Season expires with unclaimed rewards**  
   Entitled reached rewards are automatically settled once.

6. **Level completed after expiration**  
   Grants no Pass Keys.

7. **Browser refresh**  
   Pass state, Hint balance, Premium state, and claims must restore correctly.

---

# 21. Validation

`MonthlyPassConfig` should validate:

```text
durationDays == 30
keysPerCompletedLevel == 5
steps.Count == 30
step numbers == 1..30
requiredKeys == 1..30
step IDs unique
reward IDs unique
Hint amount > 0
Cosmetic reward has cosmeticId
```

Also verify:

```text
6 playable levels × 5 keys = 30 total keys
```

---

# 22. Minimum test checklist

## Progress

- [ ] New save starts at 0/30.
- [ ] Each first-time level completion grants exactly +5.
- [ ] Duplicate callbacks grant nothing.
- [ ] Six completed levels reach exactly 30/30.

## Claims

- [ ] Reached Free reward is claimable.
- [ ] Future reward is not claimable.
- [ ] Reward applies once only.
- [ ] `Claim All` claims only eligible rewards.

## Premium

- [ ] Premium starts locked.
- [ ] Mock unlock persists.
- [ ] Already-reached Premium rewards become claimable.
- [ ] Future Premium rewards remain locked.
- [ ] Premium does not change key gain.

## Persistence

- [ ] Key progress survives reload.
- [ ] Hint balance survives reload.
- [ ] Claimed rewards survive reload.
- [ ] Premium state survives reload.

## Expiration

- [ ] Expired pass stops granting keys.
- [ ] Reached unclaimed rewards settle once.
- [ ] Ended pass becomes read-only.

## WebGL

- [ ] Browser refresh preserves state.
- [ ] Focus changes do not duplicate rewards.
- [ ] No network/IAP dependency exists.

---

# 23. Implementation order

1. Extend save data with `PlayerMetaSave` and `MonthlyPassSave`.
2. Create reward/step/config data classes.
3. Author the 30-step `MonthlyPassConfig.asset`.
4. Implement `MonthlyPassService` and first-time level grant protection.
5. Implement Hint and cosmetic reward application.
6. Implement claims and `Claim All`.
7. Implement mock Premium unlock.
8. Implement expiration settlement.
9. Build the pass panel and milestone prefab.
10. Add Main-screen entry and claim badge.
11. Add the win-overlay `+5 Keys` strip.
12. Test persistence in WebGL.

---

# 24. Design rationale

The Monthly Pass follows the strongest repeated pattern across comparable casual games: **normal level completion advances a time-limited Free/Premium reward track**.

For Crossnumbers, the design intentionally stops there.

The feature adds:

- a longer-term progression goal;
- a clear Premium value proposition;
- meaningful persistence and claim states for the technical case study;
- a use for the existing Hint mechanic;
- very limited cosmetic rewards.

It avoids systems that would conflict with the original game or inflate project scope.

The six-level build compresses progression to +5 keys per level solely so the reviewer can experience the entire 30-step pass end-to-end.

---

# 25. Research sources

Consulted 2026-09-21.

1. **Royal Match — Royal Pass**, Dream Games Help Center  
   https://dreamgames.helpshift.com/hc/az/3-royal-match/faq/112-royal-pass/

2. **Candy Crush Saga — Season Pass**, King Help Center  
   https://candycrush.zendesk.com/hc/en-us/articles/7897323489693--Season-Pass

3. **Gardenscapes — Garden Pass**, Playrix Help Center  
   https://playrix.helpshift.com/hc/en/5-gardenscapes/faq/15118-garden-pass/

4. **Homescapes — Home Pass**, Playrix Help Center  
   https://playrix.helpshift.com/hc/en/14-homescapes/faq/15526-home-pass-1755869542/

5. **Match Factory — Factory Pass**, Peak Help Center  
   https://peakgames.helpshift.com/hc/en/16-match-factory/faq/581-what-is-factory-pass/

6. **Toon Blast — Toon Pass**, Peak Help Center  
   https://peakgames.helpshift.com/hc/en/4-toon-blast/faq/322-what-is-toon-pass/

7. **Clash of Clans — Gold Pass changes**, Supercell, 2026-02-16  
   https://supercell.com/en/games/clashofclans/blog/news/big-changes-are-coming-to-gold-pass/

8. **Casual-game season pass patterns**, GameRefinery  
   https://www.gamerefinery.com/2021-casual-games-trends/

9. **Battle pass design variants**, GameRefinery  
   https://www.gamerefinery.com/12-ways-to-take-battle-passes-to-the-next-level-in-mobile-games/

---

# 26. Coding-agent handoff

For the Monthly Pass:

1. Use this file as the feature authority.
2. Keep puzzle rules from `GAME_SPEC.md` unchanged.
3. Keep level content from `level_data.md` unchanged.
4. Reuse the simple architecture in `IMPLEMENTATION_SPEC.md`.
5. Do not add real IAP.
6. Do not add currencies, quests, lives, replay farming, or other pass systems not specified here.

The feature is complete when a fresh save can progress from `0/30` to `30/30` through the six first-time level completions, rewards can be claimed and persisted, Premium can be unlocked at any point with retroactive reached rewards, and the full state survives a WebGL refresh.
