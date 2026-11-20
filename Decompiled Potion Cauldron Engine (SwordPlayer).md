# **Infinity Blade III — Complete Potion Cauldron Engine & Recipes**

Reverse-engineered directly from the UE3 v868 binary (playerpawn: SwordPlayer bytecode & B\_Player\_GameObjs.Pawns.PlayerPawn archetype Export \#32320) alongside DefaultWorldItems.ini, DefaultGems.ini, DefaultGame.ini, and DefaultAbilities.ini.

## **1\. All 7 Hardcoded Potion Recipes (B\_Player\_GameObjs.Pawns.PlayerPawn)**

Unlike gems and reagents, PotionRecipes is **not** defined in any .ini file—it was authored directly in UnrealEd on the B\_Player\_GameObjs.Pawns.PlayerPawn archetype (Export \#32320, offset 0xB4AA7D).

| \# | Output (PotionResult) | In-Game Effect | Exact Ingredients (RecipeList) | Order Matters? | Cauldron Value | Base Brew Time |
| :---- | :---- | :---- | :---- | :---- | :---- | :---- |
| **1** | UberSwitchClass | **On Super: Switch Weapon Class** *(Green / Magic bottle)* | 5× **Cactus** (TRA\_World\_1\_Cactus) | No | 1,875 | 7m 45s |
| **2** | Potion\_HealthRegen 0 | **Health Regen \+10/sec** *(Tier 0, Blue bottle)* | 3× **Blue Berries** (TRA\_World\_2\_Berries) | No | 300 | 5m 00s |
| **3** | Potion\_HealthRegen 2 | **Health Regen \+100/sec** *(Tier 2, Blue bottle)* | 3× **Blue Butterfly** (TRA\_World\_8\_Butterfly) | No | 3,000 | 10m 00s |
| **4** | Potion\_HealthRegen 3 | **Health Regen \+250/sec** *(Tier 3, Blue bottle)* | 2× **Blue Berries** \+ 2× **Blue Butterfly** | No | 3,300 | 10m 36s |
| **5** | Potion\_ElementalDefense | **100% Immune to All Elements** *(ItemRare=100, Recipe-Only\!)* | **1\.** Blue Berries → **2\.** Cactus → **3\.** Pink Plant → **4\.** Cocoon → **5\.** Desert Bulb | **YES** | 39,900 | 1h 23m 48s |
| **6** | Potion\_ParryAll | **Parry All Attacks** *(ItemRare=100, Recipe-Only\!)* | **1\.** Rare Root → **2\.** Cactus → **3\.** Reeds → **4\.** Reeds → **5\.** Bones → **6\.** Cactus → **7\.** Desert Bulb → **8\.** Desert Bulb | **YES** | 939,000 | 24h 00m *(cap)* |
| **7** | UberElementalAttackGem | **Rare Darkfire \+500 Attack** *(ItemRare=100, Recipe-Only\!)* | 10× **Rare Root** (TRA\_World\_7\_Root) | No | 10,000,000 | 24h 00m *(cap)* |

### **Key Recipe Takeaways**

* **Three Recipe-Exclusive Potions (ItemRare=100):** Potion\_ElementalDefense, Potion\_ParryAll, and UberElementalAttackGem (Rare Darkfire \+500 potion) all carry ItemRare=100 in DefaultGems.ini, which excludes them from the random GetRandomPotion() pool. Recipes **\#5, \#6, and \#7** are the **only** way to brew them.  
* **Exact Input Order (bRequiresSetOrder \= true):**  
  * **Recipe \#5 (Elemental Defense):** Must be dropped into the cauldron in strict rainbow \+ booster sequence: Blue Berries → Cactus → Pink Plant → Cocoon → Desert Bulb.  
  * **Recipe \#6 (Parry All Attacks):** Must be dropped in strict 8-reagent order: Rare Root → Cactus → Reeds → Reeds → Bones → Cactus → Desert Bulb → Desert Bulb.  
* **Best Value Shortcut:** **Recipe \#4** (2× Berries \+ 2× Butterfly) costs only 3,300 cauldron value (10m 36s) yet deterministically outputs a **Tier 3 Health Regen (+250/sec)** potion (normally worth 3,125,000 unclamped value)—strictly outclassing 3× Butterfly (Tier 2, \+100/sec) for just 36 seconds more.

## **2\. All 11 Reagents (DefaultWorldItems.ini)**

Each reagent is a SwordInventoryItemWorld assigned to one of 5 PotionMixTypes (PMX\_Blue, PMX\_Green, PMX\_Yellow, PMX\_Red, PMX\_Booster).

| Internal ID (eTouchRewardActor) | Common Name | Mix Type (PotionMixTypes) | Locks Bottle Color | Gold Cost | Chip Cost | In ReagentList? |
| :---- | :---- | :---- | :---- | :---- | :---- | :---- |
| TRA\_World\_2\_Berries | Blue Berries | PMX\_Blue (1) | Blue (PotionRegen) | 100 | 5 | Yes |
| TRA\_World\_8\_Butterfly | Blue Butterfly | PMX\_Blue (1) | Blue (PotionRegen) | 1,000 | 20 | Yes |
| TRA\_World\_1\_Cactus | Green Cactus | PMX\_Green (2) | Green (PotionMagic) | 250 | 5 | Yes |
| TRA\_World\_4\_Reeds | Green Reeds / Cattail | PMX\_Green (2) | Green (PotionMagic) | 2,500 | 20 | Yes |
| TRA\_World\_10\_Cocoon | Yellow Cocoon | PMX\_Yellow (3) | Yellow (PotionShield) | 750 | 5 | Yes |
| TRA\_World\_11\_Bones | Yellow Bones | PMX\_Yellow (3) | Yellow (PotionShield) | 7,500 | 20 | Yes |
| TRA\_World\_3\_PinkPlant | Red / Pink Plant | PMX\_Red (4) | Red (PotionAttack) | 500 | 5 | Yes |
| TRA\_World\_6\_Mushroom | Red Mushroom | PMX\_Red (4) | Red (PotionAttack) | 5,000 | 20 | Yes |
| TRA\_World\_5\_DesertBulb | Desert Bulb | PMX\_Booster (5) | *Stripped (Neutral)* | 25,000 | 50 | Yes |
| TRA\_World\_7\_Root | Rare Root | PMX\_Booster (5) | *Stripped (Neutral)* | 250,000 | 100 | Yes |
| TRA\_World\_9\_Skullfly | Skullfly *(Cut / Unused)* | PMX\_Red (4) | Red (PotionAttack) | 1 | — | **No** |

## **3\. Bottle Color Locks & Dedicated Potions (DefaultGems.ini & DefaultGame.ini)**

In SwordGemIcons (DefaultGame.ini), the 4 PotionType slots map 1-to-1 to the 4 non-booster PotionMixTypes values:

| PotionType / PMX | Bottle Icon (PotionSubTypeIcon) | Dedicated SIT\_Potion Items in This Color |
| :---- | :---- | :---- |
| **1 — Blue** | PotionRegen | Potion\_HealthRegen (10..1000/s, 25k), Potion\_SecondChance (+1 revive, 650k), Potion\_ElementalDefense *(Recipe \#5 only, ItemRare=100)* |
| **2 — Green** | PotionMagic | Potion\_UnlimitedDodge (+5 dodge/0.5s, 25k) *(plus UberElementalAttackGem via Recipe \#7)* |
| **3 — Yellow** | PotionShield | Potion\_ShieldRegen (1..10/2.5s, 80k), Spawn\_SmallGoldTouch (100k), Spawn\_LargeGoldTouch (200k), Spawn\_RandomTouch (300k), Spawn\_RandomIngredientTouch (500k) |
| **4 — Red** | PotionAttack | Potion\_BlockAll (100k), Potion\_DoubleXP (+200% XP, 200k), Potion\_ParryAll *(Recipe \#6 only, ItemRare=100)* |

**Shared Gem-Potion Pool:** In addition to the 12 dedicated SIT\_Potion entries above (plus TRA\_Potion\_HealthL in DefaultPotions.ini), **63 regular gems** in DefaultGems.ini also carry PotionType=1..4 and can roll as temporary combat potions when brewing random combinations\!

## **4\. Reagent Validation & Value Multipliers**

### **SwordPlayer.IsValidPotionBrew (Export \#25535)**

function bool IsValidPotionBrew(array\<eTouchRewardActor\> Reagents)  
{  
    return Reagents.Length \> 1; // Minimum 2 reagents required  
}

### **SwordPlayer.GetReagentsValue (Export \#25490)**

Sums the store cost (GetCost(false)) of every reagent in the cauldron, then applies a **quantity multiplier** based on total reagent count:

function int GetReagentsValue(array\<eTouchRewardActor\> Reagents)  
{  
    local int I, Value;

    for (I \= 0; I \< Reagents.Length; \++I)  
    {  
        Value \+= GetConsumableObjectFromType(Reagents\[I\]).GetCost();  
    }

    switch (Reagents.Length)  
    {  
        case 4:  
        case 5:  
            Value \*= 1.5;  
            break;  
        case 6:  
        case 7:  
            Value \*= 2.0;  
            break;  
        case 8:  
        case 9:  
            Value \*= 3.0;  
            break;  
        case 10:  
            Value \*= 4.0;  
            break;  
    }  
    return Value;  
}

| Reagent Count | Cauldron Value Multiplier |
| :---- | :---- |
| 2 – 3 | 1.0× |
| 4 – 5 | 1.5× |
| 6 – 7 | 2.0× |
| 8 – 9 | 3.0× |
| 10 | 4.0× |

## **5\. Brew Time, Chip Speed-Up & Potion Perks**

### **SwordPlayer.CalculatePotionBrewCost (Export \#25531)**

function CalculatePotionBrewCost(int CauldronValue, out int RealSeconds, out int CurrencyRequired)  
{  
    local float Hours;

    Hours \= float(CauldronValue) / PotionCraftingTotalValuePerHour; // / 30,000.0  
    Hours \= FMax(Hours, PotionCraftingMinTimeHour)                  // max(Hours, 1 min)  
          \+ PotionCraftingForcedExtraTimeHour;                      // \+ 4 min forced extra  
    Hours \*= (1.0 \- FClamp(ModVars\[59\], 0.0, 0.75));                // BE\_Var\_PotionCraftTimeMod (capped 75%)  
    Hours \= FMin(Hours, PotionCraftingMaxTimeHour);                 // min(Hours, 24.0h)

    CurrencyRequired \= GetSpeedUpTimeCost(Hours);                   // max(int(Hours \* 10), 1\) Chips  
    RealSeconds \= int(Hours \* 3600.0);  
}

### **Relevant Character Perks (DefaultAbilities.ini)**

* **Isa Health 10 (Isa\_Health\_Perk\_2):** BE\_VarAdd\_PotionFightCount \= \+2 (potions last 2 extra battles).  
* **Isa Health 15 (Isa\_Health\_Perk\_3):** BE\_VarAdd\_PotionCraftSlot \= \+1 (adds an extra cauldron slot; additional slots cost 250, 500, 1,000 Chips).  
* **Isa Shield 10 (Isa\_Shield\_Perk\_2):** BE\_Var\_PotionCraftTimeMod \= 0.5 (-50% brew time).  
* **Siris Shield 200 (Siris\_Shield\_Perk\_6):** BE\_VarAdd\_PotionBoostScale \= \+10 (sets ValueBoost \= float(GetModVarInt(84)) on brewed/dropped potions).  
* **Siris Magic 5 / 15 (Siris\_Magic\_Perk\_1 / 3):** \+1 extra reagent picked per node & \+10 max reagent carry capacity.

## **6\. Exact Recipe Matching (CheckForPotionRecipe)**

### **SwordPlayer.CheckForPotionRecipe (Export \#25520)**

function bool CheckForPotionRecipe(  
    out array\<eTouchRewardActor\> SortedReagents,  
    out string RecipePotionFullName,  
    out array\<eTouchRewardActor\> UnSortedReagents)  
{  
    local int I, J;

    for (I \= 0; I \< PotionRecipes.Length; \++I)  
    {  
        if (PotionRecipes\[I\].SortedRecipeList.Length \== 0\)  
        {  
            PotionRecipes\[I\].SortedRecipeList \= PotionRecipes\[I\].RecipeList;  
            PotionRecipes\[I\].SortedRecipeList.Sort(SortReagentList);  
        }

        // Exact reagent count must match  
        if (SortedReagents.Length \!= PotionRecipes\[I\].SortedRecipeList.Length)  
            continue;

        // Multiset match (sorted comparison)  
        for (J \= 0; J \< SortedReagents.Length; \++J)  
        {  
            if (int(SortedReagents\[J\]) \!= int(PotionRecipes\[I\].SortedRecipeList\[J\]))  
                break;  
        }  
        if (J \!= SortedReagents.Length)  
            continue;

        // Order-sensitive check (only when bRequiresSetOrder \== true)  
        if (PotionRecipes\[I\].bRequiresSetOrder)  
        {  
            for (J \= 0; J \< UnSortedReagents.Length; \++J)  
            {  
                if (int(UnSortedReagents\[J\]) \!= int(PotionRecipes\[I\].RecipeList\[J\]))  
                    break;  
            }  
            if (J \!= UnSortedReagents.Length)  
                continue;  
        }

        RecipePotionFullName \= PotionRecipes\[I\].PotionResult;  
        return true;  
    }  
    return false;  
}

## **7\. Complete Brewing Flow & Random Potion RNG (CompleteBrewingPotion)**

### **SwordPlayer.CompleteBrewingPotion (Export \#25511)**

When a cauldron finishes brewing:

1. **Deterministic Recipe Check First:** Calls CheckForPotionRecipe(). If one of the 7 recipes matches, immediately spawns GetFixedPotion(RecipePotionFullName) (applying ValueBoost \= float(GetModVarInt(84)) for BE\_VarAdd\_PotionBoostScale).  
2. **Value Scaling (reuses Gem Forge constants):![][image1]![][image2]![][image3]**  
   If ![][image4], bumps ![][image5].  
3. **Booster Stripping:** Removes every PMX\_Booster (DesertBulb, Root) from SortedReagents **after** MaxPotionValue is computed—so Boosters inflate your cauldron value and quantity multiplier without breaking a pure-color lock\!  
4. **Pure-Color Lock (ForcePotionType):** If **all** remaining non-booster reagents share the exact same PotionMixTypes color (Blue, Green, Yellow, or Red), ForcePotionType locks to that color. Mixing two or more colors (or brewing with Boosters only) sets ForcePotionType \= 0\.  
5. **16-Attempt Generation Loop (I \= 0 .. 15):**  
   * If I \> 8 (starting on the 10th attempt), ForcePotionType is cleared to 0 (unlocking colors as a safety valve).  
   * Picks a random non-booster reagent from SortedReagents and sets PotionFavorType to its color (so your reagent color ratio directly weights FavorType).  
   * Calls GetRandomPotion(RewardLevel=1, GoldScale=10.0, GoldValue=ScaledValue, FavorType=PotionFavorType), which delegates to GetRandomGem(..., GoldScale \* GemToPotionValueScale, ..., CacheIdx=14, true).  
   * **Accept/Reject Check:** Rejects and scales ScaledValue \*= 1.15 if:  
     * NewPotion \== none, **OR**  
     * **![][image6]** (GemToPotionValueScale), **OR**  
     * ForcePotionType \> 0 and NewPotion.PotionType \!= ForcePotionType.  
   * Otherwise accepts NewPotion and exits the loop.

[image1]: <data:image/png;base64,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>

[image2]: <data:image/png;base64,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>

[image3]: <data:image/png;base64,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>

[image4]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAbEAAAAVCAYAAADIKtD9AAARd0lEQVR4Xu1dC6wdRRnem1ZT4wsftdDHzt62UltEMAVJRbAaIG0Ag4JYU6IgUQgRhJKCEEDEEIsotFAeQUlbDSnQy8O0pTwaEGqkUoOQWNsIDUpqCRhoJJQESHv9vjP/v2d2dnbPnnvOufeC+yWTPTvPf/7H/PPae6OoRo0aNWrUqFGjRo0aNWrUqFGjRo0aNdpHf3//hDiOT8TzC0mSjGPc5MmTpyN8wM/bKcaPH/+hSZMmfQI/+/y0dgFa92Pw43uJbtJfo7uYPXv2+6ZNm/YpPv200YqR0OH/F7xXbZVjNMJcjNfHSf+iiRMnfhJx+/t5OwXbok3h5xg/bVSAxg4ir0F4yhhzJsKlCM/KczPCAX6ZoQIO8WDU9wrCIMIfqGB+nqqg8FDHXtYF2lcxTurfIfVreBvO+RgtR2Eg/z/cPHif16y5GN2kfzQAfTgA4V8uL0rCDX750YQJEyZ8EHL8vdC6BzKfzXjEnY33N7y+7EJ4G2Ev8j0EXTo6GoFBLqTDwwHaANp80uUJ2l/g51Nwgos825z8r6lNSR/W4fk5v1wZKC+UW+PSIHQsc/OhnVvddL676UV4r9mqC/RnPsLzCJcZO2ZvRLgJvHuYCxE//1AhevI34SHHia75gq5CjHyzenNi6tSpHxXGdJ1wOk3UeZ/pgmKRZtTztD8ATJky5SjEv4WwacaMGR920wRjIOw7kX4Zf/uJZegm/d0GZYWwHGFd0sbMHnycjDI7Q/xCXD/qegLPO6IRGOjbBWg93ThOTGHspGwQYaHGUZbI933EvYPnLSOxeivS4eEA9SWxk9c3EQYQNdbPQyDPt4xMDsGni720VaH4qmD/lQbY7eF+OoG6T0X6Jjimj/tpZRjNtjpUgBcHURYc45zoPsRfJHLomhMT9EE+15ge+IKugIKlgBGu9tMQNwdhay8IF8XvWLGUfn8AECfMVSQd2Rw3zUlfB0X4rJ9WBd2iv1sALf0ItyNskBlxW86GMjZWSYN9Ip+Qdk8obbSBRmwCToyDrAkb+VjEDzAtKVmN9ApFOjwcELn/FuF3CLsL7GEsaLsSYZnwL+OsYEsHIv7ypINtLNYv/L/STyMk/Yd+fBWMNlvtFKLHW/D8mBuvY15AvzuGtDk6nZgoMYnLzcLIJMb3gvBuKVbZAEClLzIMxM9B/1ZGBTPPVugW/R2ijw4LdGygnPB7hp+hKhw9yPRJVmVj+ETaHb3QhW5jCE5M5Vl5q6qbKNPhXoPylL5/HWGf76Akz0zaEMKPhUe5PJ1CJkm7ETZzMHbTZHC+F+1OdeOrYiRslSvGFqvGMejzxKjNySYh/XkpxA/EXx3S704xqp0YGDIOhK03dl9+Ed+dZC5RZ/lbLFz+I/40MozKP3369I84ydymm434c/C8SAaS3HZdkWLxIgnLsTx/u2mCPs78kOcUKMFhVBTWExoAHMPYygNPNw35lyaBWTf7CmdwtNLAtqKAonn0j+HesbQ3Xy7C9JFPqMtQqRA3yS3PPIg/AfX8FGFem5dn+lDmCLT1GMJytD3Fz9AuTNiJsZ0lTOML6D0V/ZnQLJXKehH1wd2OFr3iADmXtMqW2XyfVtnu+aqxunRmQOZ01IcYO9P/rtLigmcryHMc+Uz6+DTtObGylVgrfc7JXmmgHvm2I6isw2Jr30T6pX7byL+fq1/kOfnNd0cWDfoRfxJCEnm6LDJaJdvJ3LnIORGkn2HspE/5lzox6grLkv9Im8k4saEJ/faC2Fw5cydO8vugEHt40FhHmp5fE3wnjVF+wtlKNg2wLPk7nLaKPJ9G/fexTj9NtrCvQPriyJNHFYCGsygHhHvEEaagLPyxroWNVeKhyD7nxDj2x9YXXI7nl6JA2WEBGPE1I4fLErZTYP6AA9CpfQfpL8f2HKEf4QGEbWQe01FuKdJepAIbO4PjQfuGgGFknJgwmoeUXCbT6Obh9wsIF7Je5pEZ2ZrE7p8vRDgTeR/C87+i5BkkTQe9j/VpPA2M8SFlRfxa1k8lFOE+HQfOSlz68Xs//L7bWB42BC1t/8LIhQIah5bF+0yEZ5DnEvB+Gn7fjPBsSOE9jEG+4xE2IVzrOo1OQZqF9lQmYog8F805DtC+P3mP5xLSjedc9oH6EVk9Ie/0QJhbHCtEVv/B8yDWYSwf/o5wm/D7PPzeS31kOp0Tyxl7pjFTdQ/hG0qHsfxgHGegp4COpfj9V1PdiY1BmW8j/h2EX3pybqnPnux3Jvbs4Gd4LsDzTwgbXd1vQ4fV1raQt+QxcC3CI+Q9M+B5gbGTtEFj+79C2r0B4TWEkxF3J8IP8Hsxwh6kn++0kToxylz4sy92nIjQewOfDv9SJ4bfJyBuuxuP+hL8flzoouxvNXaiw/7wXO02356knA7Oy6Pm4N6H9xtVJxy0lI2C/TPDb6ts99DYXhpK83bqwAiZcLiXbHajzpV4fjHynIjQX2RjlXkoss84MWMd43Mi18KywwUaDJVRb/Jo4O2tdMBI7Ox/j5G9aRE+B5iX++1WFme0qxFex/shzKOrIVfxpWzGibEdljPO+RXb0bpF+DTSp9zBu9WheGKNetA4hoG4eajren1XiMFyNrqjX1YczGucgVXh008Ye3EgI2gaBNtXw9CtI4TVkcws2ZaxSvkrLReCsQPRLiqcn9YpSLPQTuP+tzwH/f4Q6uwR7nYHoyn2Mg1lfSzfnb6+IgZ0LsLjNELyGjx8Au8P6szW2FtW+/jkuxgOL5v0SxMc0Hhp5VnKXfTxdeQ7TdIbSAoudkh9g8YaNeni7Tz2c4fk9QeVyvpsmpdG0nMbY3V6XyxOoR0dJg/ZDnmqcTLR4wC8VnmG9MPx/qbLRxnkyLddzqxbV5u+zqZOTPtmHFvB7zlIP4O/lX9+37U9L56y4vZzZgJpLJ9eNbJqc2HspJh0b1P7E9vgVrm7A0BUls1I2KoicRxZNxyYIrHgZatBN8TOhLuCjVXmocg+5RcXOPj9XOIc1UhZyrZhvyOFPhAyMbbLw0eEMWoIdHQrhMhUAdGJca5y8D1xbsWx0+w8FUnjJJ87O9IVU2bbzzHQE2M7s6cDzVxAUUXz61eYpmHoYMhD6lWxt2WhYH1uf7RdX6gu/U7e3GzFNwy2a6xhn6V5oqbBZ+oLQfoz5AscRSDNQrs7seAsLrcSK+iDW8cAXsc6g0CuX5wUIJ7bd24d3NrgYXUf9cDYS0XrqR+awVjHsAflDxN9zJ0NCM8LnZjKgqBuJ/Zzi8yKScG2kwr6LHVnbCMg+0o6zDbZbxPYBpd2eJOSWzdpnS4flUbj3SYN6SzzMl7i1NGprfQl9kJFo08O/3wH3mjPj5f2Mn2QOnKyEeigmm7r8omw1M9IkE9JBdmE+i109NRWFSh/KPI/ivBr0wUH5oI0kM7YrnZ5w5T0NyZ1rWyML1V56POLdZI3bFvzyLn5Jr9szyGzg1lRnrFU4EuUCc6AlBF8ANzyOtnYFc12Y43iLb9jrmIp44zdalqB521uoBI4CpYxFH8ACIBKx9l7wzCMdQJrQgMWIauMcxCekbDOBA68Xfo1zhe0xPmG0RgIUP5h4/UTYXGSPZMsBNswtl+PocwRUV5+bUHqyzixyA4q17n9kbyL3D458VpHYyVb5sSUD34dirg54L9g8vqwjDordef0UXieGyiL2qR+S3xGxoJK+lxR9pV0mLwzdtutkG8Ii+RdJ1nuFlhwIArpLPMyXuMSayMcuBeQjsTOtBurEId/7TixkI3kZKNI7M4HVwqrxRZ/YwK3iwWVZFNCRyt5dcVWZQXNLV6u+HOXMdoFdzW41e7HJ82dssbEz5FX0MYElXjo80t4SjlxJ87nzfCuxEgUCfKNhSDDjZ3pXlzFiUmeu5H/Ra6iGMe8LOMzxVUsZ9aduzaqcBSsdAAIIWkaBoV7fhK4rUiQVtT/Z9KBPAnjaGymiyuxpLnvn36r1AlkK4pnJTwnOz4a4sEqaRbaM33S24lO1sI+OHU05Fgmm1YGZuwqkCubgSh/oJ/KXdrryImpjIy3cmlHn6vI3nkv1eEye9A+GOE9+2i66MTEgW5DfQ8i/gy3Xod/PXNi2j7Cq8hzCp5rQxPOdmRTQkepvJICPW8H4sBon4tj+33X+v4K52llQB03FvAvs2XsyCtoY+3w0OeXsU453REYUQjRf+F+pp/m7K03hJjYrYUc4RQKZgeTYll+m+y5QMoUpjMw3lOsPmNXFbmtISjwgQixaQ5qmb8a4Q8AIcT2U4Etxn4ztjXUV0KUNrOHT2Ux4sTwXIh+Hix5KxmGsdtfqSI55w6ZLSVZER8ZBQbsKpBLED8y9mxlQejgvAykWWjP9CkEY78fJC8vdeMpO8S9ZORMpUw2Th0ZPshNw0M4C4/tbbXcQM706dOnjzfWkKgTmfMV8toEBkqRT86oJf+g8T70jtvQ55DstV5tj3QKva10WLfVdsqFqRTG3jBLv+diH00XnVjUtEXO6B+gU9G8Dv965sQI6eMgwgum4NuwdmRTQkdPbdV1YJFMjozVgY4cmfQnxBfKjludA5HVoVIba4eHPr+S5sIg4+A54ejvwZl9KZRohEcne981xPZsbKsaEhmf2PODAT0oFEEtR96DlCnssFMHD6j3CuMXqoL4isXyxm4n/jySmT/rRr5leiaX2JtnO/BMnPqPNNbgGoLTeB+JfEwZO4ecPhKZeXEv2Yk7W8pdLKFhfD79Tvn0kFQP8lneNC/IaD924fkZLYs2j8L7JUzXuKFAtmC+h3bv9+VZBmfCkjHsAoxF/bck3gWF2OoLb642bh/q4Iyw0d/+EL3h1kOGD3hfSD7yt+jOO3g/PRK+yMrzZj5VZ1y+ifNbyXKxN9kyzcsX6WUlgvmY38gkio4MdS6JrRPaR7k7eYP6LHVnnI6Ud51mZR3G788b+6ed0ksrcpnlKdYRSX/13Njtkyl3YpkzKmMH1Q3yd/E0Ljj4kQ/SnypOTAdUf2JQxYlp+0UfX6dOrIps5H1YbVXGLv4pvwsiL5/p0JFJf+hceRsxBcdJY/+SR2P8amVj7fBQ5JaODTLOrEXYwgmlWzeDvg8LSBTC5sR+yPhPELsysd/j8Cv+l8CQw9z8XBkl9nrw88ZeMNiYyMpFBWesUfFvoq1GfVcIA95G2AQDOibJ/t3C9G+wOXXzajaZzwFwvrYtjOPMhucFbPteyccr1axrp66UfMTNM5ZCBss14juRZzeFiHCXsTckbzLWqNag/iOK6DeWl3809tYb6RpA2rnGtsu893IwF+VabOysnNet2c7toW2TXkP+Nhq3Iikf0sjAvzX5ovYrBHEW/JaO216UBc8On6QMmc6yxl7z1jr5+zq3DvnbeVxNvS48oDyvcleRiP8y4p4zlka2wyvmh2q6DOLbjW2f6fzkg3UOMiDvssRORPybt7u0fzKA3WLszPJa/D6Pejxr1qz3t9Jn0iI6q/Uynucl7I/e8OSgcBf7244Ox/bcj7qkfSOvFyt/kH4965ZyDKSRetW4Ks4guspBy9XZN2K7Vadtamjop0w+1hs5ixJZBvknvE3bM9ZGeF2b/dM4tscJDvuiceRTRh8UzuSncGLa6VhjemyryPMV6lFU4OjE4VxFHfPTWiG2N1yXJPZ2IvnEM3x+nkQaL4ycNstsrAoPqd8mK7c3KHOv7lcTu3K7H7+vdu13WECjkpVOQzHotKjgmCUUfajZQGI/tJwQypPY21UHuDMfEVZQoD7K6ia0fj7x2vhQkWX8fC5YF/tUZXUiRqT1N+D+bgXSon/xWdrlAXmuvJNWSvtoRjf6ENIXDw0ZK0/9xEjSZVXI2fO4Mv0pAMsR/CD3WNqFJoToa0efQ9A6RS9KdZjxQ+jPuxp05CZwDd9Hp7Ihb99tttovnxxFWZ11P3DPIcSnsrQ2eZgrX6NGEKosVYIO6H4dNWrUqFGjxojA2H+/wK2PlgEzs5/UM6MaNWrUqFGjRo0aNUYp/gd/MnvCBOpUEAAAAABJRU5ErkJggg==>

[image5]: <data:image/png;base64,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>

[image6]: <data:image/png;base64,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>