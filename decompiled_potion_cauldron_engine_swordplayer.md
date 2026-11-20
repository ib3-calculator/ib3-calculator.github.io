# Infinity Blade III — Complete Potion Cauldron Engine & Recipes

Reverse-engineered directly from the UE3 v868 binary (`playerpawn`: `SwordPlayer` bytecode & `B_Player_GameObjs.Pawns.PlayerPawn` archetype Export `#32320`) alongside `DefaultWorldItems.ini`, `DefaultGems.ini`, `DefaultGame.ini`, and `DefaultAbilities.ini`.

## 1. All 7 Hardcoded Potion Recipes (`B_Player_GameObjs.Pawns.PlayerPawn`)

Unlike gems and reagents, `PotionRecipes` is **not** defined in any `.ini` file—it was authored directly in UnrealEd on the `B_Player_GameObjs.Pawns.PlayerPawn` archetype (Export `#32320`, offset `0xB4AA7D`).

| \# | Output (`PotionResult`) | In-Game Effect | Exact Ingredients (`RecipeList`) | Order Matters? | Cauldron Value | Base Brew Time | 
| ----- | ----- | ----- | ----- | ----- | ----- | ----- | 
| **1** | `UberSwitchClass` | **On Super: Switch Weapon Class** *(Green / Magic bottle)* | `5×` **Cactus** (`TRA_World_1_Cactus`) | No | `1,875` | `7m 45s` | 
| **2** | `Potion_HealthRegen 0` | **Health Regen +10/sec** *(Tier 0, Blue bottle)* | `3×` **Blue Berries** (`TRA_World_2_Berries`) | No | `300` | `5m 00s` | 
| **3** | `Potion_HealthRegen 2` | **Health Regen +100/sec** *(Tier 2, Blue bottle)* | `3×` **Blue Butterfly** (`TRA_World_8_Butterfly`) | No | `3,000` | `10m 00s` | 
| **4** | `Potion_HealthRegen 3` | **Health Regen +250/sec** *(Tier 3, Blue bottle)* | `2×` **Blue Berries** + `2×` **Blue Butterfly** | No | `3,300` | `10m 36s` | 
| **5** | `Potion_ElementalDefense` | **100% Immune to All Elements** *(`ItemRare=100`, Recipe-Only!)* | **1.** Blue Berries → **2.** Cactus → **3.** Pink Plant → **4.** Cocoon → **5.** Desert Bulb | **YES** | `39,900` | `1h 23m 48s` | 
| **6** | `Potion_ParryAll` | **Parry All Attacks** *(`ItemRare=100`, Recipe-Only!)* | **1.** Rare Root → **2.** Cactus → **3.** Reeds → **4.** Reeds → **5.** Bones → **6.** Cactus → **7.** Desert Bulb → **8.** Desert Bulb | **YES** | `939,000` | `24h 00m` *(cap)* | 
| **7** | `UberElementalAttackGem` | **Rare Darkfire +500 Attack** *(`ItemRare=100`, Recipe-Only!)* | `10×` **Rare Root** (`TRA_World_7_Root`) | No | `10,000,000` | `24h 00m` *(cap)* | 

### Key Recipe Takeaways

* **Three Recipe-Exclusive Potions (`ItemRare=100`):** `Potion_ElementalDefense`, `Potion_ParryAll`, and `UberElementalAttackGem` (Rare Darkfire +500 potion) all carry `ItemRare=100` in `DefaultGems.ini`, which excludes them from the random `GetRandomPotion()` pool. Recipes **#5, #6, and #7** are the **only** way to brew them.

* **Exact Input Order (`bRequiresSetOrder = true`):**

  * **Recipe #5 (Elemental Defense):** Must be dropped into the cauldron in strict rainbow + booster sequence: `Blue Berries → Cactus → Pink Plant → Cocoon → Desert Bulb`.

  * **Recipe #6 (Parry All Attacks):** Must be dropped in strict 8-reagent order: `Rare Root → Cactus → Reeds → Reeds → Bones → Cactus → Desert Bulb → Desert Bulb`.

* **Best Value Shortcut:** **Recipe #4** (`2× Berries + 2× Butterfly`) costs only `3,300` cauldron value (`10m 36s`) yet deterministically outputs a **Tier 3 Health Regen (+250/sec)** potion (normally worth `3,125,000` unclamped value)—strictly outclassing `3× Butterfly` (Tier 2, `+100/sec`) for just 36 seconds more.

## 2. All 11 Reagents (`DefaultWorldItems.ini`)

Each reagent is a `SwordInventoryItemWorld` assigned to one of 5 `PotionMixTypes` (`PMX_Blue`, `PMX_Green`, `PMX_Yellow`, `PMX_Red`, `PMX_Booster`).

| Internal ID (`eTouchRewardActor`) | Common Name | Mix Type (`PotionMixTypes`) | Locks Bottle Color | Gold Cost | Chip Cost | In `ReagentList`? | 
| ----- | ----- | ----- | ----- | ----- | ----- | ----- | 
| `TRA_World_2_Berries` | Blue Berries | `PMX_Blue` (`1`) | Blue (`PotionRegen`) | `100` | `5` | Yes | 
| `TRA_World_8_Butterfly` | Blue Butterfly | `PMX_Blue` (`1`) | Blue (`PotionRegen`) | `1,000` | `20` | Yes | 
| `TRA_World_1_Cactus` | Green Cactus | `PMX_Green` (`2`) | Green (`PotionMagic`) | `250` | `5` | Yes | 
| `TRA_World_4_Reeds` | Green Reeds / Cattail | `PMX_Green` (`2`) | Green (`PotionMagic`) | `2,500` | `20` | Yes | 
| `TRA_World_10_Cocoon` | Yellow Cocoon | `PMX_Yellow` (`3`) | Yellow (`PotionShield`) | `750` | `5` | Yes | 
| `TRA_World_11_Bones` | Yellow Bones | `PMX_Yellow` (`3`) | Yellow (`PotionShield`) | `7,500` | `20` | Yes | 
| `TRA_World_3_PinkPlant` | Red / Pink Plant | `PMX_Red` (`4`) | Red (`PotionAttack`) | `500` | `5` | Yes | 
| `TRA_World_6_Mushroom` | Red Mushroom | `PMX_Red` (`4`) | Red (`PotionAttack`) | `5,000` | `20` | Yes | 
| `TRA_World_5_DesertBulb` | Desert Bulb | `PMX_Booster` (`5`) | *Stripped (Neutral)* | `25,000` | `50` | Yes | 
| `TRA_World_7_Root` | Rare Root | `PMX_Booster` (`5`) | *Stripped (Neutral)* | `250,000` | `100` | Yes | 
| `TRA_World_9_Skullfly` | Skullfly *(Cut / Unused)* | `PMX_Red` (`4`) | Red (`PotionAttack`) | `1` | — | **No** | 

## 3. Bottle Color Locks & Dedicated Potions (`DefaultGems.ini` & `DefaultGame.ini`)

In `SwordGemIcons` (`DefaultGame.ini`), the 4 `PotionType` slots map 1-to-1 to the 4 non-booster `PotionMixTypes` values:

| `PotionType` / `PMX` | Bottle Icon (`PotionSubTypeIcon`) | Dedicated `SIT_Potion` Items in This Color | 
| ----- | ----- | ----- | 
| **1 — Blue** | `PotionRegen` | `Potion_HealthRegen` (`10..1000/s`, `25k`), `Potion_SecondChance` (`+1 revive`, `650k`), `Potion_ElementalDefense` *(Recipe #5 only, `ItemRare=100`)* | 
| **2 — Green** | `PotionMagic` | `Potion_UnlimitedDodge` (`+5 dodge/0.5s`, `25k`) *(plus `UberElementalAttackGem` via Recipe #7)* | 
| **3 — Yellow** | `PotionShield` | `Potion_ShieldRegen` (`1..10/2.5s`, `80k`), `Spawn_SmallGoldTouch` (`100k`), `Spawn_LargeGoldTouch` (`200k`), `Spawn_RandomTouch` (`300k`), `Spawn_RandomIngredientTouch` (`500k`) | 
| **4 — Red** | `PotionAttack` | `Potion_BlockAll` (`100k`), `Potion_DoubleXP` (`+200% XP`, `200k`), `Potion_ParryAll` *(Recipe #6 only, `ItemRare=100`)* | 

> **Shared Gem-Potion Pool:** In addition to the 12 dedicated `SIT_Potion` entries above (plus `TRA_Potion_HealthL` in `DefaultPotions.ini`), **63 regular gems** in `DefaultGems.ini` also carry `PotionType=1..4` and can roll as temporary combat potions when brewing random combinations!

## 4. Reagent Validation & Value Multipliers

### `SwordPlayer.IsValidPotionBrew` (Export `#25535`)

```
function bool IsValidPotionBrew(array<eTouchRewardActor> Reagents)
{
    return Reagents.Length > 1; // Minimum 2 reagents required
}

```

### `SwordPlayer.GetReagentsValue` (Export `#25490`)

Sums the store cost (`GetCost(false)`) of every reagent in the cauldron, then applies a **quantity multiplier** based on total reagent count:

```
function int GetReagentsValue(array<eTouchRewardActor> Reagents)
{
    local int I, Value;

    for (I = 0; I < Reagents.Length; ++I)
    {
        Value += GetConsumableObjectFromType(Reagents[I]).GetCost();
    }

    switch (Reagents.Length)
    {
        case 4:
        case 5:
            Value *= 1.5;
            break;
        case 6:
        case 7:
            Value *= 2.0;
            break;
        case 8:
        case 9:
            Value *= 3.0;
            break;
        case 10:
            Value *= 4.0;
            break;
    }
    return Value;
}

```

| Reagent Count | Cauldron Value Multiplier | 
| ----- | ----- | 
| 2 – 3 | `1.0×` | 
| 4 – 5 | `1.5×` | 
| 6 – 7 | `2.0×` | 
| 8 – 9 | `3.0×` | 
| 10 | `4.0×` | 

## 5. Brew Time, Chip Speed-Up & Potion Perks

### `SwordPlayer.CalculatePotionBrewCost` (Export `#25531`)

```
function CalculatePotionBrewCost(int CauldronValue, out int RealSeconds, out int CurrencyRequired)
{
    local float Hours;

    Hours = float(CauldronValue) / PotionCraftingTotalValuePerHour; // / 30,000.0
    Hours = FMax(Hours, PotionCraftingMinTimeHour)                  // max(Hours, 1 min)
          + PotionCraftingForcedExtraTimeHour;                      // + 4 min forced extra
    Hours *= (1.0 - FClamp(ModVars[59], 0.0, 0.75));                // BE_Var_PotionCraftTimeMod (capped 75%)
    Hours = FMin(Hours, PotionCraftingMaxTimeHour);                 // min(Hours, 24.0h)

    CurrencyRequired = GetSpeedUpTimeCost(Hours);                   // max(int(Hours * 10), 1) Chips
    RealSeconds = int(Hours * 3600.0);
}

```

### Relevant Character Perks (`DefaultAbilities.ini`)

* **Isa Health 10 (`Isa_Health_Perk_2`):** `BE_VarAdd_PotionFightCount = +2` (potions last 2 extra battles).

* **Isa Health 15 (`Isa_Health_Perk_3`):** `BE_VarAdd_PotionCraftSlot = +1` (adds an extra cauldron slot; additional slots cost `250`, `500`, `1,000` Chips).

* **Isa Shield 10 (`Isa_Shield_Perk_2`):** `BE_Var_PotionCraftTimeMod = 0.5` (-50% brew time).

* **Siris Shield 200 (`Siris_Shield_Perk_6`):** `BE_VarAdd_PotionBoostScale = +10` (sets `ValueBoost = float(GetModVarInt(84))` on brewed/dropped potions).

* **Siris Magic 5 / 15 (`Siris_Magic_Perk_1 / 3`):** `+1` extra reagent picked per node & `+10` max reagent carry capacity.

## 6. Exact Recipe Matching (`CheckForPotionRecipe`)

### `SwordPlayer.CheckForPotionRecipe` (Export `#25520`)

```
function bool CheckForPotionRecipe(
    out array<eTouchRewardActor> SortedReagents,
    out string RecipePotionFullName,
    out array<eTouchRewardActor> UnSortedReagents)
{
    local int I, J;

    for (I = 0; I < PotionRecipes.Length; ++I)
    {
        if (PotionRecipes[I].SortedRecipeList.Length == 0)
        {
            PotionRecipes[I].SortedRecipeList = PotionRecipes[I].RecipeList;
            PotionRecipes[I].SortedRecipeList.Sort(SortReagentList);
        }

        // Exact reagent count must match
        if (SortedReagents.Length != PotionRecipes[I].SortedRecipeList.Length)
            continue;

        // Multiset match (sorted comparison)
        for (J = 0; J < SortedReagents.Length; ++J)
        {
            if (int(SortedReagents[J]) != int(PotionRecipes[I].SortedRecipeList[J]))
                break;
        }
        if (J != SortedReagents.Length)
            continue;

        // Order-sensitive check (only when bRequiresSetOrder == true)
        if (PotionRecipes[I].bRequiresSetOrder)
        {
            for (J = 0; J < UnSortedReagents.Length; ++J)
            {
                if (int(UnSortedReagents[J]) != int(PotionRecipes[I].RecipeList[J]))
                    break;
            }
            if (J != UnSortedReagents.Length)
                continue;
        }

        RecipePotionFullName = PotionRecipes[I].PotionResult;
        return true;
    }
    return false;
}

```

## 7. Complete Brewing Flow & Random Potion RNG (`CompleteBrewingPotion`)

### `SwordPlayer.CompleteBrewingPotion` (Export `#25511`)

When a cauldron finishes brewing:

1. **Deterministic Recipe Check First:** Calls `CheckForPotionRecipe()`. If one of the 7 recipes matches, immediately spawns `GetFixedPotion(RecipePotionFullName)` (applying `ValueBoost = float(GetModVarInt(84))` for `BE_VarAdd_PotionBoostScale`).

2. **Value Scaling (reuses Gem Forge constants):** 

   $$\text{Scale} = \frac{\text{FInterpEaseInOut}(1.1,\; 1.55,\; \text{FRand}(),\; 0.5)}{3.333}$$

   $$\text{ScaledValue} = \lfloor \text{MaxPotionValue} \times \text{Scale} \rfloor$$

   $$\text{ForcedRandomMinValue} = \lfloor \text{MaxPotionValue} \times 0.65 \rfloor$$

    If $\text{ScaledValue} \le \text{ForcedRandomMinValue} \times \text{Scale}$, bumps $\text{ScaledValue} = \lfloor \text{ForcedRandomMinValue} \times \text{Scale} \times 1.1 \rfloor$.

3. **Booster Stripping:** Removes every `PMX_Booster` (`DesertBulb`, `Root`) from `SortedReagents` **after** `MaxPotionValue` is computed—so Boosters inflate your cauldron value and quantity multiplier without breaking a pure-color lock!

4. **Pure-Color Lock (`ForcePotionType`):** If **all** remaining non-booster reagents share the exact same `PotionMixTypes` color (`Blue`, `Green`, `Yellow`, or `Red`), `ForcePotionType` locks to that color. Mixing two or more colors (or brewing with Boosters only) sets `ForcePotionType = 0`.

5. **16-Attempt Generation Loop (`I = 0 .. 15`):**

   * If `I > 8` (starting on the 10th attempt), `ForcePotionType` is cleared to `0` (unlocking colors as a safety valve).

   * Picks a random non-booster reagent from `SortedReagents` and sets `PotionFavorType` to its color (so your reagent color ratio directly weights `FavorType`).

   * Calls `GetRandomPotion(RewardLevel=1, GoldScale=10.0, GoldValue=ScaledValue, FavorType=PotionFavorType)`, which delegates to `GetRandomGem(..., GoldScale * GemToPotionValueScale, ..., CacheIdx=14, true)`.

   * **Accept/Reject Check:** Rejects and scales `ScaledValue *= 1.15` if:

     * `NewPotion == none`, **OR**

     * $\text{NewPotion.GetCost(true)} \le \dfrac{\text{MaxPotionValue}}{10.0}$ (`GemToPotionValueScale`), **OR**

     * `ForcePotionType > 0` and `NewPotion.PotionType != ForcePotionType`.

   * Otherwise accepts `NewPotion` and exits the loop.
