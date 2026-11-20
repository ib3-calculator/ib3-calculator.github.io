# Infinity Blade III — Complete Gem Forge (Gem Cooker) Engine & Recipes

Reverse-engineered directly from the UE3 v868 binary (`playerpawn`: `SwordPlayer` & `SwordInventoryConsumable` bytecode and `Default__SwordPlayer` CDO at `0x342181`) alongside `DefaultGems.ini`, `DefaultGame.ini`, and `DefaultAbilities.ini`.

## 1. Verified Binary Constants (`Default__SwordPlayer` & `DefaultGame.ini`)

| Property | Binary Value | Role in Gem Forge | 
| ----- | ----- | ----- | 
| `GemCookedMinValueScale` | `1.1` | Lower bound of `FInterpEaseInOut(1.1, 1.55, FRand(), 0.5)` | 
| `GemCookedMaxValueScale` | `1.55` | Upper bound of `FInterpEaseInOut(1.1, 1.55, FRand(), 0.5)` | 
| `ItemMinGoldToValueScale` | `1.0` | `TargetMin = ScaledValue * 1.0` | 
| `ItemMaxGoldToValueScale` | `3.333` | Scale divisor & `TargetMax = ScaledValue * 3.333` | 
| `GemCookerScaleMinOfTotal` | `0.65` | Floor check: $\max(\text{BestInput},\; \lfloor 0.65 \times \text{TotalValue} \rfloor)$ | 
| `GemCookerScaleBetterThanBestGemValue` | `1.1` | Bump multiplier when $\text{ScaledValue} \le \text{MaxGemValue} \times \text{Scale}$ | 
| `GenericGemCookBoostScale` | `1.75` | `ModVars[83]` (*Isa Attack 150*) multiplier on `MaxGemValue` | 
| `NormalSameGemBoostChance` | **`1` (`IntProperty`)** | Configured as `1.1` in `DefaultGame.ini`, truncated to `1` in cooked binary | 
| `GemCookerTotalGemValuePerHour` | `30000.0` | Cook time divisor ($\text{MaxSingleCost} / 30{,}000$ hours) | 
| `GemCookerMinCookTimeHour` | `0.5` | Minimum cook time (30 minutes) | 
| `GemCookerMaxCookTimeHour` | `72.0` | Maximum cook time cap (72 hours) | 
| `TimeSpeedUpChipsPerHour` | `10.0` | Speed-up cost: $\max(\lfloor \text{Hours} \times 10 \rfloor,\; 1)$ Chips | 
| `CurrentPlayerSellGemScale` | `0.5` | Sell price = `50%` of clamped display cost | 
| `CurrentPlayerRecoverGemScale` | `0.25` | Unsocket cost = `25%` of cost (capped at `50,000` Gold) | 

## 2. Socket Shapes & Gem Pool (`DefaultGems.ini`)

There are **152** `SIT_Gem` definitions in `DefaultGems.ini` across **8** socket shapes (`ItemSubType = 1..8`).

| `ItemSubType` | Socket Shape | Icon (`SwordGemIcons`) | Total Gems | Normal Pool (`ItemRare=0`) | Rare / Uber (`ItemRare=100`) | 
| ----- | ----- | ----- | ----- | ----- | ----- | 
| **1** | Circle | `GemCircle` | 16 | 12 | 4 | 
| **2** | Square | `GemSquare` | 41 | 29 | 12 | 
| **3** | Diamond | `GemDiamond` | 18 | 13 | 5 | 
| **4** | Upside-Down Triangle | `GemUpsideTriange` | 32 | 32 | 0 | 
| **5** | Hexagon | `GemHex` | 17 | 14 | 3 | 
| **6** | Light Weapon (Triangle Up) | `GemTriangle` | 8 | 8 | 0 | 
| **7** | Heavy Weapon (Hourglass) | `GemHourglass` | 10 | 10 | 0 | 
| **8** | Dual Weapon (Star) | `GemStar` | 10 | 10 | 0 | 
| **Total** |  |  | **152** | **128** | **24** | 

> **Early Bloodline Shape Cap:** On unlocked (mixed-shape) cooks during early Bloodlines, `GetRandomGem` caps random shape selection (`maxType`): BL 1 $\rightarrow$ Shapes `1–2`, BL 2–3 $\rightarrow$ `1–3`, BL 4–5 $\rightarrow$ `1–4`, BL 6–8 $\rightarrow$ `1–5`, and BL 9+ $\rightarrow$ all `1–8`.

## 3. Core Gem Math (`SwordInventoryConsumable`)

### Unclamped Cook Value vs. Display / Sell Cost (`GetCost`)

Every gem tier scales its base cost by $5^{\text{Tier}}$ (except improvable Uber gems with `RecipeBoostAmount > 0`, which add $+64$ per upgrade level):

$$\text{TierCost}(\text{Tier}) = \begin{cases} \text{Cost} + 64 \times \text{Tier} & \text{if } \text{RecipeBoostAmount} > 0 \\ \lfloor \text{Cost} \times 5^{\text{Tier}} \rfloor & \text{otherwise} \end{cases}$$

$$\text{GetCost}(\text{Unclamped}=\text{true}) = \lfloor \text{TierCost}(\text{Tier}) \times (1 + 0.5 \times \text{RandPct}) \rfloor$$

$$\text{GetCost}(\text{Unclamped}=\text{false}) = \min\!\big(\text{GetCost}(\text{true}),\; 1{,}000{,}000\big)$$

$$\text{GetSellCost}() = \lfloor 0.5 \times \text{GetCost}(\text{false}) \rfloor$$

### Maximum Random Add & `GetMaxCost()` Quirk

If `MaxRandomAddPct` is omitted (`0`), UE3 defaults `GetMaxRandomAdd()` to `0.25`. If negative (`-1.0`, e.g., `BossBoostGem`), `GetMaxRandomAdd()` returns `0.0`.

```
// SwordInventoryConsumable::GetMaxCost()
// Even bUniqueItem=true gems (which can never roll RandPct > 0) multiply by (1 + 0.25 * 0.5) = 1.125!
MaxCost = int(TierCost(UpgradeTier.Length) * (1.0 + GetMaxRandomAdd() * 0.5));

```

Because of this `1.125×` phantom multiplier, the highest `GetMaxCost()` in `ItemCaches[1]` is `UberGoldGem` at $22{,}500{,}000$ ($20{,}000{,}000 \times 1.125$).

### Stat Display & Max-Gem Checks

* **Displayed Stat (`GetScaledValue`):**

  * Improvable Ubers: $\text{BaseVal} + \text{Tier} \times \text{RecipeBoostAmount}$.

  * Standard gems: $\lfloor \text{TierBase} \times (1 + \text{RandPct}) \rfloor$ (multiplied by $100$ for percentage gems, and $100 \times (\text{Scaled} - 1)$ for `BE_World_DropRatio` gems).

* **`IsMaxGem(bMaxTierOnlyNeeded)`:** Returns `true` for all `bUniqueItem` gems; otherwise requires `Tier >= UpgradeTier.Length` and (unless `bMaxTierOnlyNeeded == true`) `RandPct >= GetMaxRandomAdd()`.

* **`IsImprovableUberGem()`:** `bUniqueItem && RecipeBoostAmount > 0 && Tier < 255`.

* **`GetRecipeBoostedGemTier()`:** Indoor (`Restrict=1`) and Outdoor (`Restrict=2`) gems add `+4` to their effective tier (`4 + Tier`) when used in recipe comparisons.

## 4. Cook Time, Chip Speed-Up & Perks (`CalculateGemCookCost`)

```
function CalculateGemCookCost(array<SwordInventoryConsumable> Gems, out int RealSeconds, out int CurrencyRequired)
{
    local float Hours;
    local int MaxSingleCost;

    // Uses the SINGLE most expensive gem (unclamped), NOT the sum of all 3!
    MaxSingleCost = Max(Gems[0].GetCost(true), Gems[1].GetCost(true), Gems[2].GetCost(true));

    Hours = float(MaxSingleCost) / GemCookerTotalGemValuePerHour;   // / 30,000.0
    Hours = FMax(Hours, GemCookerMinCookTimeHour);                  // min 0.5h (30m)
    Hours *= (1.0 - FClamp(ModVars[57], 0.0, 0.75));                // BE_Var_GemCraftTimeMod (cap 75%)
    Hours = FMin(Hours, GemCookerMaxCookTimeHour);                  // max 72.0h

    CurrencyRequired = Max(int(Hours * TimeSpeedUpChipsPerHour), 1); // 10 Chips/hr, min 1
    RealSeconds = int(Hours * 3600.0);
}

```

### Relevant Character Perks (`DefaultAbilities.ini` & `DefaultGame.ini`)

* **Isa Attack 50 (`Isa_Attack_Perk_5`):** `BE_VarAdd_GemCraftSlot = +1` (adds an extra Gem Cooker slot; additional slots cost `500`, `1,000`, `1,500` Chips).

* **Isa Attack 150 (`Isa_Attack_Perk_6`):** `BE_VarAdd_GemCookBoost = 1` (`ModVars[83]`):

  * Multiplies `MaxGemValue` by **`1.75×`** (`GenericGemCookBoostScale`) on random fusions.

  * Guarantees **`HighestTier + 1` at maximum `RandPct`** on Same-Gem Upgrade cooks.

* **Isa Shield 25 (`Isa_Shield_Perk_4`):** `BE_Var_GemCraftTimeMod = 0.5` (`ModVars[57]`, -50% cook time).

## 5. Invalid Combination Rules (`InvalidCookCombo`)

Before a cook is allowed to start, `InvalidCookCombo()` checks whether **any** of the 3 input gems passes `IsMaxGem(false)`. If none are maxed, the cook is always valid. If at least one is maxed **and** the combo does not match an Uber Creation (`CookVar = 10`) or Uber Upgrade (`CookVar = 50`) recipe, the forge rejects the combination if **any** of the following holds:

1. **Rare + Maxed Conflict:** Any of the 3 gems has `ItemRare == 100`.

2. **Maxed Same-Name / RecipeMatch:** All 3 gems share the same template ID or `RecipeMatch` string (without satisfying an Uber Creation recipe).

3. **Global Cost Ceiling:** The highest maxed input gem has `GetCost(true) > 10,000,000` **and** `>= MaxGlobalGemCost` (`22,500,000` Gold).

## 6. Deterministic Recipes (`CheckForGemRecipes`)

`CheckForGemRecipes()` runs before the RNG loop and evaluates recipes in strict priority order:

### A. Uber Creation — Normal + Indoor + Outdoor Triplets (12 Recipes, `CookVar = 10`)

Requires **3 distinct** gems sharing the same `RecipeMatch` and `MPParent`, all **100% maxed** (`IsMaxGem(false)` — max tier **and** max `RandomAddPct`):

| Output Uber Gem (`MPParent`) | Base Stat | Required 3 Maxed Inputs | 
| ----- | ----- | ----- | 
| `UberAttackGem` | `+250 Attack` | `AttackGem` (T5 MAX) + `AttackGemIn` (T1 MAX) + `AttackGemOut` (T1 MAX) | 
| `UberHealthGem` | `+60 Health` | `HealthGem` (T5 MAX) + `HealthGemIn` (T1 MAX) + `HealthGemOut` (T1 MAX) | 
| `UberShieldGem` | `+75 Shield` | `ShieldGem` (T5 MAX) + `ShieldGemIn` (T1 MAX) + `ShieldGemOut` (T1 MAX) | 
| `UberMagicGem` | `+250 Magic` | `MagicGem` (T5 MAX) + `MagicGemIn` (T1 MAX) + `MagicGemOut` (T1 MAX) | 
| `UberFireGem` | `+500 Fire` | `FireGem` (T5 MAX) + `FireGemIn` (T1 MAX) + `FireGemOut` (T1 MAX) | 
| `UberIceGem` | `+500 Ice` | `IceGem` (T5 MAX) + `IceGemIn` (T1 MAX) + `IceGemOut` (T1 MAX) | 
| `UberElecGem` | `+500 Shock` | `ElecGem` (T5 MAX) + `ElecGemIn` (T1 MAX) + `ElecGemOut` (T1 MAX) | 
| `UberPoisonGem` | `+500 Poison` | `PoisonGem` (T5 MAX) + `PoisonGemIn` (T1 MAX) + `PoisonGemOut` (T1 MAX) | 
| `UberLightGem` | `+500 Light` | `LightGem` (T5 MAX) + `LightGemIn` (T1 MAX) + `LightGemOut` (T1 MAX) | 
| `UberDarkGem` | `+500 Dark` | `DarkGem` (T5 MAX) + `DarkGemIn` (T1 MAX) + `DarkGemOut` (T1 MAX) | 
| `UberWaterGem` | `+500 Water` | `WaterGem` (T5 MAX) + `WaterGemIn` (T1 MAX) + `WaterGemOut` (T1 MAX) | 
| `UberWindGem` | `+500 Wind` | `WindGem` (T5 MAX) + `WindGemIn` (T1 MAX) + `WindGemOut` (T1 MAX) | 

### B. Uber Creation — Matching Triplets (`SameForUber = 1`, `CookVar = 10`)

Requires 3 maxed gems with the same `RecipeMatch` / ID where `SameForUber > 0`:

| Output Gem | Effect | Required 3 Maxed Inputs | 
| ----- | ----- | ----- | 
| `UberElementalGem2` | `10%` All-Element Defense | `3×` `UberElementalGem1` (`5%`) | 
| `UberElementalGem3` | `25%` All-Element Defense | `3×` `UberElementalGem2` (`10%`) | 
| `UberElementalGem4` | `50%` All-Element Defense | `3×` `UberElementalGem3` (`25%`) | 
| `UberElementalGemMax` | `100%` Spectrum Defense | `3×` `UberElementalGem4` (`50%`) | 
| `UberBossBoostGem` | `+1,000` Titan Level | `3×` `BossBoostGem` (T5, `+500`) | 
| `GreatParryAllGem` | Great Parry All Attacks | Any `3×` maxed `PerfectParryGem_1` (T4 MAX) and/or `PerfectParryGem_2` (T5 MAX) | 

### C. Mixed Elemental Defense (`CheckForAllElementalDefenseGems`)

* **Inputs:** Any 3 single-element Defense gems (`Gem3_1`..`Gem3_8`), not all 3 identical (3 identical gems are blocked by `InvalidCookCombo`).

* **Output:** `UberElementalGem1` (**5% All-Element Defense**), 100% deterministic.

### D. Improvable Uber Gem Upgrades (`CheckForImproveableUberGem`, `CookVar = 50`)

Upgrades an improvable Uber gem by `+1` level (up to level `255`), adding `RecipeBoostAmount` to its stat:

* **Standard 13 Improvable Ubers:** `1×` Improvable Uber (`UberAttack/Health/Shield/Magic`, `UberFire/Ice/Elec/Poison/Light/Dark/Water/Wind`, or `UberBossBoostGem`) + `2×` max-tier feeders whose `MPParent` matches that Uber. *(Uses `IsMaxGem(bMaxTierOnlyNeeded = true)` — full `RandomAddPct` is **not** required on feeders!)*

* **`SpecialUberElementalCheck` (`UberElementalAttackGem_100` — Rare Darkfire +1000):** `1×` `UberElementalAttackGem_100` + `2×` **fully maxed** (`IsMaxGem(false)`) elemental attack gems of **any** element (Normal, Indoor, Outdoor, or `UberElementalAttackGem` +500). Adds **`+500`** Darkfire per cook! *(Note: `RainbowElementalAttackGem_100` has `RecipeBoostAmount=250`, but `SpecialUberElementalCheck` only checks `UberElementalAttackGem_100`.)*

## 7. Same-Gem Upgrade RNG (`SameNameRecipeGemCheck`)

When all 3 non-maxed gems share the same template ID or `RecipeMatch`:

1. Computes `HighestTier` and `HighestIdx` (tie-broken by highest `RandPct`) using `GetRecipeBoostedGemTier()`, and sums `TierTotal`.

2. **If Gem Cook Boost Perk (`ModVars[83]`) is active:** Deterministically outputs `HighestTier + 1` at `100%` of `MaxRandomAddPct`.

3. **If `MaxRandomAddPct < 0` (`BossBoostGem`):** `GetMaxRandomAdd() == 0` forces `BoostTierPct = 1.0`, deterministically stepping to `HighestTier + 1` with `0%` random add.

4. **Otherwise (Standard Same-Gem RNG):**

$$\text{BoostTierPct} = \frac{(\text{TierTotal} + 3) - (\text{HighestTier} + 1)}{\text{HighestTier} + 1} \times \underbrace{\lfloor 1.1 \rfloor}_{=\,1}$$

Then calls `GetFullGemGeneratedName(Anchor, BoostTierPct, BoostRandMaxPct = 0.5)`:

* If `Anchor.RandPct >= MaxRand`, forces `BoostTierPct = 1.0`.

* Rolls `FRand() < BoostTierPct`:

  * On **tier-up** (if `Anchor.Tier + 1 <= MaxTier`): increments `UsedTier`, resets `UsedRand = 0`, and scales `BoostRandMaxPct *= 0.25 * (BoostTierPct + 1.0)`.

  * If already at max tier: scales `BoostRandMaxPct *= max(BoostTierPct, 1.0)`.

* Adds a uniform roll in $[0.1 \times \text{MaxAdd},\; 1.0 \times \text{MaxAdd}]$ where $\text{MaxAdd} = \text{BoostRandMaxPct} \times \text{MaxRand}$, clamped to `MaxRand`.

## 8. Random Fusion RNG Loop (`CompleteCookingGem`)

When no recipe matches, the forge runs the value-based random generator:

### Step 1: Value Floor (`MaxGemValue`) & Initial `ScaledValue`

$$\text{TotalValue} = \sum_{i=0}^{2} \text{GetCost}(\text{Gem}_i,\; \text{true})$$

$$\text{MaxGemValue} = \max\!\big(\max_i \text{GetCost}(\text{Gem}_i,\; \text{true}),\; \lfloor 0.65 \times \text{TotalValue} \rfloor\big)$$

If `ModVars[83]` (*Gem Cook Boost*) is enabled: $\text{MaxGemValue} = \lfloor \text{MaxGemValue} \times 1.75 \rfloor$.

$$\text{Scale} = \frac{\text{FInterpEaseInOut}(1.1,\; 1.55,\; \text{FRand}(),\; 0.5)}{3.333}$$

$$\text{ScaledValue} = \lfloor \text{TotalValue} \times \text{Scale} \rfloor$$

If $\text{ScaledValue} \le \text{MaxGemValue} \times \text{Scale}$, bumps $\text{ScaledValue} = \lfloor \text{MaxGemValue} \times \text{Scale} \times 1.1 \rfloor$.

### Step 2: Socket Shape Lock (`PrefType`)

* If all 3 input gems share the **same** socket shape (`ItemSubType`), `PrefType` locks to that shape.

* Otherwise `PrefType = 0` (each attempt picks a random shape in `1..maxType`).

### Step 3: 8-Attempt Generation Loop (`Attempt = 0 .. 7`)

On each attempt:

1. Sets $\text{TargetMin} = \lfloor \text{ScaledValue} \rfloor$ and $\text{TargetMax} = \lfloor \text{ScaledValue} \times 3.333 \rfloor$.

2. Filters the Normal Pool (`ItemRare == 0`) to candidates where `GetMaxCost() >= MaxGemValue` and `BaseCost <= TargetMax`:

   * **`PreferredItemIndexArray`:** Candidates matching `PrefGemType` that have at least one tier with clamped cost in $[\text{TargetMin},\; \text{TargetMax}]$ (`GetCostCloseToValue`).

   * **`ValidItemIndexArray`:** All other candidates passing `BaseCost <= TargetMax` (regardless of socket shape!).

   * Picks uniformly from `Preferred` if non-empty, else uniformly from `Valid`.

3. **Tier & `RandPct` Roll (Non-Unique Gems):**

   * Finds highest tier in $[\text{MaxGemValue},\; \text{TargetMax}]$ (falling back to $[0,\; \text{TargetMax}]$).

   * Computes $\text{Exp} = \text{FClamp}\!\big(\frac{\text{TargetMax}}{\text{TierCost}} - 1,\; 0.05,\; 1.0\big)$ and minimum `RandPct` needed to clear `MaxGemValue`.

   * Rolls $\text{RandPct} = \text{FInterpEaseOut}(\text{MinRand},\; \text{MaxRand},\; \text{FRand}(),\; \text{Exp})$.

4. **Accept / Reject Rules:**

   * If `Picked == none` **or** `Picked.GetCost(true) <= MaxGemValue`: rejects and scales **`ScaledValue = int(ScaledValue * 1.15)`**.

   * If `Picked.ID` matches **any** of the 3 input gem IDs: rejects **without** scaling `ScaledValue`.

   * Otherwise **accepts** and finishes the cook.

### Step 4: 8-Failure Fallback

If all 8 attempts fail, the forge upgrades the most expensive input gem (`FallbackAnchor`) via `GetFullGemGeneratedName`:

```
FallbackBoostPct = float(MaxGemValue) / float(TotalValue);
// UE3 quirk: int(0.2) truncates to 0, so FallbackRandMax = Max(int(FallbackBoostPct), 0)!
FallbackRandMax  = Max(int(FallbackBoostPct), 0);
Result = GetFullGemGeneratedName(FallbackAnchor, FallbackBoostPct, FallbackRandMax);

```

## 9. Notable UE3 Engine Quirks & `DefaultGems.ini` Bugs

1. **High-Value Shape-Lock Break (`ScaledValue > 1,000,000`):**
   `GetCostCloseToValue()` compares against **clamped** tier costs ($\le 1{,}000{,}000$). Whenever `TargetMin = ScaledValue > 1,000,000` (e.g., fusing 3 Triangle gems totaling $> 10\text{M}$ Gold), **zero** gems match `GetCostCloseToValue()`, emptying `PreferredItemIndexArray`. The engine falls back to `ValidItemIndexArray`, completely **breaking the same-shape lock** and rolling across all 8 socket shapes!

2. **`NormalSameGemBoostChance` Integer Truncation:**
   `DefaultGame.ini` sets `NormalSameGemBoostChance=1.1`, but `Default__SwordPlayer` declares it as an `IntProperty`, silently truncating it to `1`.

3. **`int(0.2) = 0` Fallback Truncation:**
   In the 8-failure fallback path, the minimum random-add floor is truncated via integer cast (`int(0.2) == 0`), meaning a fallback that does not trigger a tier-up adds `0%` random stat unless `FallbackBoostPct >= 1.0`.

4. **`DualGem_8` Tier Overwrite Bug (`DefaultGems.ini`):**
   `DualGem_8` declares `UpgradeTier[3]=400` followed immediately by `UpgradeTier[3]=750`, overwriting index 3 instead of defining `UpgradeTier[4]`.

5. **`ItemDropGem_4` Missing Section Header (`DefaultGems.ini`):**
   The Gold Drop gem properties were pasted directly under `[ItemDropGem_4]` without a `[ItemDropGem_5]` header, overwriting the Potion Drop gem so `ItemDropGem_4` becomes the Gold Drop gem.