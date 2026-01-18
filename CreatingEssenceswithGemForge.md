# Creating Essences with GemForge

**Version 1.0** | A comprehensive guide to creating custom gems (Essences) for Shape of Dreams

---

## Table of Contents

1. [Quick Start](#quick-start)
2. [Setup & Installation](#setup--installation)
3. [Your First Gem](#your-first-gem)
4. [Gem Lifecycle & Hooks](#gem-lifecycle--hooks)
5. [Common Patterns](#common-patterns)
6. [Advanced Techniques](#advanced-techniques)
7. [Testing & Debugging](#testing--debugging)
8. [Game Mechanics Reference](#game-mechanics-reference)
9. [Troubleshooting](#troubleshooting)
10. [API Reference](#api-reference)

---

## Quick Start

### The 3-Minute Gem

```csharp
using DewInternal;
using GemForge;

// 1. Create your gem class
public class Gem_E_Vampirism : GemForgeGem
{
    protected override void OnDealDamage(EventInfoDamage info)
    {
        base.OnDealDamage(info);
        if (!isServer) return;
        
        float totalDamage = info.damage.amount + info.damage.discardedAmount;
        owner.Heal(totalDamage * 0.10f, owner); // 10% lifesteal
        
        NotifyUse();
    }
}

// 2. Register it in your mod
public class VampirismMod : ModBehaviour
{
    private void Awake()
    {
        GemForgeAPI.Initialize("com.yourname.vampirism");
        
        GemForgeAPI.RegisterGem<Gem_E_Vampirism>(new GemDefinition
        {
            Guid = "mod-gem-e-vampirism-guid",
            AssetId = GemDefinition.GenerateAssetId("mod-gem-e-vampirism-guid"),
            Rarity = Rarity.Epic,
            Name = "Vampirism",
            ShortDescription = "Heal from damage dealt",
            Description = "Heal for <color=#88ff88>10%</color> of damage dealt.",
            Memory = "Life feeds on life."
        });
    }
}
```

**That's it!** Your gem is now in the game, spawnable, droppable, and fully functional.

---

## Setup & Installation

### Prerequisites

- **Game:** Shape of Dreams (Steam version)
- **IDE:** Visual Studio 2022, VS Code, or Rider
- **.NET SDK:** .NET Framework 4.8

### Project Setup

1. **Create a new Class Library project:**

```bash
dotnet new classlib -n YourModName -f net4.8
```

2. **Edit your `.csproj`:**

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net4.8</TargetFramework>
    <LangVersion>latest</LangVersion>
    <AllowUnsafeBlocks>true</AllowUnsafeBlocks>
    
    <!-- Point to your game installation -->
    <GameDir>C:\Program Files (x86)\Steam\steamapps\common\Shape of Dreams</GameDir>
    
    <!-- Output directly to Mods folder for easy testing -->
    <OutputPath>$(GameDir)\Mods\$(AssemblyName)\</OutputPath>
    <AppendTargetFrameworkToOutputPath>false</AppendTargetFrameworkToOutputPath>
  </PropertyGroup>

  <ItemGroup>
    <!-- Game assemblies -->
    <Reference Include="Assembly-CSharp">
      <HintPath>$(GameDir)\Shape of Dreams_Data\Managed\Assembly-CSharp.dll</HintPath>
      <Private>False</Private>
    </Reference>
    
    <Reference Include="Sirenix.Serialization">
      <HintPath>$(GameDir)\Shape of Dreams_Data\Managed\Sirenix.Serialization.dll</HintPath>
      <Private>False</Private>
    </Reference>
    
    <!-- Unity -->
    <Reference Include="UnityEngine">
      <HintPath>$(GameDir)\Shape of Dreams_Data\Managed\UnityEngine.dll</HintPath>
      <Private>False</Private>
    </Reference>
    
    <Reference Include="UnityEngine.CoreModule">
      <HintPath>$(GameDir)\Shape of Dreams_Data\Managed\UnityEngine.CoreModule.dll</HintPath>
      <Private>False</Private>
    </Reference>
    
    <!-- Modding tools -->
    <Reference Include="Mirror">
      <HintPath>$(GameDir)\Shape of Dreams_Data\Managed\Mirror.dll</HintPath>
      <Private>False</Private>
    </Reference>
    
    <Reference Include="0Harmony">
      <HintPath>$(GameDir)\Shape of Dreams_Data\Managed\0Harmony.dll</HintPath>
      <Private>False</Private>
    </Reference>
    
    <Reference Include="IngameDebugConsole.Runtime">
      <HintPath>$(GameDir)\Shape of Dreams_Data\Managed\IngameDebugConsole.Runtime.dll</HintPath>
      <Private>False</Private>
    </Reference>
  </ItemGroup>

  <ItemGroup>
    <!-- Auto-copy assets to output -->
    <None Update="*.png;manifest.json">
      <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
    </None>
  </ItemGroup>
</Project>
```

3. **Add GemForge:**

Copy `GemForgeCore.cs` to your project.

4. **Create a manifest.json:**

```json
{
  "id": "com.yourname.yourmod",
  "name": "Your Mod Name",
  "version": "1.0.0",
  "author": "Your Name",
  "description": "Custom gems for Shape of Dreams"
}
```

5. **Create a ModBehaviour class to implement your gems later:**

```csharp
using GemForge;
using UnityEngine;
using IngameDebugConsole;
using Mirror;
using System.Collections.Generic;

public class YourModName : ModBehaviour
{
    // ===== CONFIGURATION - FILL THIS IN =====
    private static readonly List<GemInfo> GEMS = new List<GemInfo>
    {
        new GemInfo
        {
            GemType = typeof(Gem_E_YourFirstGem),
            UniqueGuid = "yourmodname-gem-e-firstgem-guid",
            Name = "First Gem",
            ShortDescription = "Does something cool",
            Description = "Deal <color=#ff8888>25% bonus damage</color>.",
            Rarity = Rarity.Epic,
            Memory = "First of many.",
            Template = "Empowered {0}"
        },
        // Add more gems here:
        // new GemInfo { GemType = typeof(Gem_E_SecondGem), UniqueGuid = "yourmodname-gem-e-secondgem-guid", ... }
    };
    // ===== END CONFIGURATION =====

    private void Awake()
    {
        GemForgeAPI.Initialize("com.yourname.yourmodname");  // CHANGE THIS UNIQUE ID
        
        RegisterAllGems();
        RegisterDebugCommands();
        
        Debug.Log($"[YourModName] Loaded {GEMS.Count} gems successfully!");
    }

    private void RegisterAllGems()
    {
        foreach (var gemInfo in GEMS)
        {
            try
            {
                var def = new GemDefinition
                {
                    Guid = gemInfo.UniqueGuid,
                    AssetId = GemDefinition.GenerateAssetId(gemInfo.UniqueGuid),
                    Rarity = gemInfo.Rarity,
                    ExcludeFromPool = false,
                    Name = gemInfo.Name,
                    ShortDescription = gemInfo.ShortDescription,
                    Description = gemInfo.Description,
                    Memory = gemInfo.Memory,
                    Template = gemInfo.Template,
                    IconTexture = GetSafeFallbackIcon()
                };

                GemForgeAPI.RegisterGem(gemInfo.GemType, def);
                Debug.Log($"[SUCCESS] {gemInfo.Name} registered!");
            }
            catch (System.Exception e)
            {
                Debug.LogError($"[FAILED] {gemInfo.Name}: {e.Message}");
            }
        }
    }

    private void RegisterDebugCommands()
    {
        foreach (var gemInfo in GEMS)
        {
            string cmdName = $"spawn_{gemInfo.Name.ToLower().Replace(" ", "")}";
            DebugLogConsole.AddCommand(cmdName, $"Spawn {gemInfo.Name}", 
                (int q) => SpawnGem(gemInfo.GemType, q));
        }
        DebugLogConsole.AddCommand("checkgems", "Check all gems", CheckAllGems);
    }

    private static void SpawnGem(System.Type gemType, int quality)
    {
        if (!NetworkServer.active) { Debug.LogWarning("Must be host"); return; }
        
        var hero = DewPlayer.local?.hero;
        if (hero == null) { Debug.LogWarning("No hero"); return; }

        quality = Mathf.Clamp(quality, 0, 100);
        var prefab = DewResources.GetByType(gemType);
        
        if (prefab == null) { Debug.LogError($"No prefab: {gemType.Name}"); return; }

        var gem = Dew.CreateGem(prefab, hero.position + hero.transform.forward * 2f, quality, DewPlayer.local);
        if (gem != null)
            Debug.Log($"✨ Spawned {gemType.Name} (Q:{quality})");
    }

    private static void CheckAllGems()
    {
        Debug.Log("=== GEM REGISTRATION ===");
        var db = DewResources.database;
        foreach (var gemInfo in GEMS)
        {
            bool ok = db.allGuids?.Contains(gemInfo.UniqueGuid) ?? false;
            Debug.Log($"{gemInfo.Name}: {(ok ? "✅" : "❌")}");
        }
    }

    private static Texture2D GetSafeFallbackIcon()
    {
        string[] fallbacks = { "Gem_C_Efficiency", "Gem_C_Vampirism", "Gem_E_Expression" };
        foreach (string name in fallbacks)
        {
            try
            {
                var gem = DewResources.GetByShortTypeName<Gem>(name);
                if (gem?.icon?.texture != null) return gem.icon.texture;
            }
            catch { }
        }
        return null;
    }

    // Simple data class - don't touch this
    private class GemInfo
    {
        public System.Type GemType;
        public string UniqueGuid;
        public string Name;
        public string ShortDescription;
        public string Description;
        public Rarity Rarity = Rarity.Epic;
        public string Memory;
        public string Template;
    }
}
```

---

## Your First Gem

### Step 1: Create the Gem Class

```csharp
using DewInternal;
using GemForge;
using UnityEngine;

public class Gem_E_TestEssence : GemForgeGem
{
    private const float BONUS_MULT = 0.30f; // 30% bonus

    protected override void OnDealDamage(EventInfoDamage info)
    {
        base.OnDealDamage(info);
        if (!isServer) return;
        if (info.chain.DidReact(this)) return;
        if (!owner.CheckEnemyOrNeutral(info.victim)) return;

        float totalDamage = info.damage.amount + info.damage.discardedAmount;
        float bonusDamage = totalDamage * BONUS_MULT;

        DamageData bonus = MagicDamage(bonusDamage, 1.0f);
        //bonus.SetElemental(ElementalType.Fire);
        bonus.SetSourceType(info.damage.type);
        bonus.Dispatch(info.victim, info.chain.New(this));

        NotifyUse();
    }
}
```

### Step 2: Register the Gem

**In your ModBehaviour Class**

```csharp
private void Awake()
{
    GemForgeAPI.Initialize("com.zakrow.zakrowessenceexpansion");
    RegisterGems();
    RegisterDebugCommands();
    Debug.Log("[zakrowessenceexpansion] Awake complete.");
}
```

```csharp
private void RegisterGems()
{
    try
    {
        string uniqueGuid = "mygemmod-gem-e-test-v2";

        // CRITICAL: Get a fallback icon BEFORE registration
        Texture2D fallbackIcon = null;
        try
        {
            // Try to grab ANY existing gem's icon as fallback
            var existingGem = DewResources.GetByShortTypeName<Gem>("Gem_C_Efficiency");
            if (existingGem != null && existingGem.icon != null)
            {
                fallbackIcon = existingGem.icon.texture;
                Debug.Log("[Icon] Found fallback from Gem_C_Efficiency");
            }
        }
        catch { /* Ignore fallback failures */ }

        var def = new GemDefinition
        {
            Guid = uniqueGuid,
            AssetId = GemDefinition.GenerateAssetId(uniqueGuid),
            Rarity = Rarity.Epic,
            ExcludeFromPool = false,
            Name = "Test Essence",
            ShortDescription = "Adds bonus magic damage",
            Description = "Deal <color=#ff8888>30% bonus magic damage</color>.",
            Memory = "The first spark of creation.",
            Template = "Empowered {0}",
            IconTexture = fallbackIcon  // THIS IS REQUIRED
        };

        GemForgeAPI.RegisterGem<Gem_E_TestEssence>(def);
        Debug.Log($"[SUCCESS] Test Essence registered with fallback icon: {(fallbackIcon != null)}");
    }
    catch (System.Exception e)
    {
        Debug.LogError($"[REGISTRATION FAILED] {e.Message}");
    }
}
```

```csharp
private Texture2D LoadSafeIcon()
{
    try
    {
        // Try custom icon first
        string iconPath = System.IO.Path.Combine(instance.mod.path, "TestGem.png");
        if (System.IO.File.Exists(iconPath))
        {
            byte[] iconData = System.IO.File.ReadAllBytes(iconPath);
            Texture2D texture = new Texture2D(2, 2);
            //ImageConversionModule.LoadImage(texture, iconData);
            texture.filterMode = FilterMode.Bilinear;
            return texture;
        }
    }
    catch (System.Exception e)
    {
        Debug.LogWarning($"[Icon] Failed to load custom icon: {e.Message}");
    }

    // Safe fallback to ANY existing gem
    return DewResources.GetByShortTypeName<Gem>("Gem_C_Efficiency")?.icon?.texture;
}
```

**Add Debugging Tools**

```csharp
private void RegisterDebugCommands()
{
    DebugLogConsole.AddCommand("spawn_testgem", "Spawn Test Essence", (int quality) => SpawnTestGem(quality));
}
```

```csharp
private static void SpawnTestGem(int quality)
{
    if (!NetworkServer.active)
    {
        Debug.LogWarning("Must be host/server");
        return;
    }

    var hero = DewPlayer.local?.hero;
    if (hero == null) return;

    quality = Mathf.Clamp(quality, 0, 100);

    var prefab = DewResources.GetByType<Gem_E_TestEssence>();
    if (prefab == null)
    {
        Debug.LogError("Test Essence prefab not found!");
        return;
    }

    var gem = Dew.CreateGem(
        prefab,
        hero.position + hero.transform.forward * 2f,
        quality,
        DewPlayer.local);

    if (gem != null)
        Debug.Log($"Spawned Test Essence with quality {quality}");
}
```

### Step 3: Build & Test

```bash
dotnet build
```

---

## Gem Lifecycle & Hooks

### Lifecycle Overview

```
1. Prefab Creation (GemForge)
   ↓
2. Spawning (game calls Dew.CreateGem)
   ↓
3. OnQualityChange (quality set)
   ↓
4. PrepareAndSpawn (activation)
   ↓
5. OnEquipSkill (player equips to skill)
   ↓
6. Runtime Events (OnDealDamage, OnTakeDamage, etc.)
   ↓
7. OnUnequipSkill (player removes)
   ↓
8. Destroy
```

### Available Hooks

All hooks are **virtual methods** you override in your gem class:

#### Combat Hooks

| Hook | Called When | Use For |
|------|-------------|---------|
| `OnDealDamage(EventInfoDamage)` | Your skill damages an enemy | Bonus damage, effects on hit |
| `OnTakeDamage(EventInfoDamage)` | Owner takes damage | Counter-attacks, damage reduction |
| `OnKill(EventInfoDamage)` | You kill an enemy | On-kill explosions, chains |
| `OnCrit(EventInfoDamage)` | Your skill crits | Crit-specific effects |

#### Skill Hooks

| Hook | Called When | Use For |
|------|-------------|---------|
| `OnEquipSkill(SkillTrigger)` | Gem equipped to skill | AoE expansion, stat mods |
| `OnUnequipSkill(SkillTrigger)` | Gem removed from skill | Cleanup, restore values |
| `OnCastStart(EventInfoCast)` | Skill starts casting | Cast speed mods, prep |
| `OnCastComplete(EventInfoCast)` | Skill finishes | Spawn extra effects |
| `OnCastCancel(EventInfoCast)` | Cast interrupted | Refunds, penalties |

#### Utility Hooks

| Hook | Called When | Use For |
|------|-------------|---------|
| `OnQualityChange(int, int)` | Quality updated | Quality-dependent logic |
| `OnPickup(Hero)` | Player picks up gem | First-time effects |
| `OnDrop()` | Gem dropped | Cleanup |

### Hook Best Practices

**ALWAYS check `isServer`:**
```csharp
protected override void OnDealDamage(EventInfoDamage info)
{
    base.OnDealDamage(info);
    if (!isServer) return; // CRITICAL
    
    // Your logic here
}
```

**ALWAYS prevent recursion:**
```csharp
if (info.chain.DidReact(this)) return;
```

**ALWAYS call base:**
```csharp
base.OnDealDamage(info);
```

---

## Common Patterns

### Pattern: Quality Scaling

Scale effects with gem quality:

```csharp
private const float BASE_BONUS = 0.10f;      // 10% at quality 0
private const float QUALITY_SCALING = 0.002f; // +0.2% per quality

private float GetBonusMultiplier()
{
    // At quality 100: 10% + (100 * 0.2%) = 30%
    return BASE_BONUS + (quality * QUALITY_SCALING);
}

protected override void OnDealDamage(EventInfoDamage info)
{
    base.OnDealDamage(info);
    if (!isServer) return;
    if (info.chain.DidReact(this)) return;

    float totalDamage = info.damage.amount + info.damage.discardedAmount;
    float bonusDamage = totalDamage * GetBonusMultiplier();
    
    DamageData bonus = MagicDamage(bonusDamage, 1.0f);
    bonus.Dispatch(info.victim, info.chain.New(this));
    
    NotifyUse();
}
```

### Pattern: Bonus Damage

Add extra damage to attacks:

```csharp
protected override void OnDealDamage(EventInfoDamage info)
{
    base.OnDealDamage(info);
    if (!isServer) return;
    if (info.chain.DidReact(this)) return;
    if (!owner.CheckEnemyOrNeutral(info.victim)) return;

    float totalDamage = info.damage.amount + info.damage.discardedAmount;
    float bonusDamage = totalDamage * 0.25f; // 25% bonus

    DamageData bonus = MagicDamage(bonusDamage, 1.0f);
    bonus.SetElemental(ElementalType.Fire);
    bonus.Dispatch(info.victim, info.chain.New(this));
    
    NotifyUse();
}
```

### Pattern: Lifesteal

Heal based on damage dealt:

```csharp
protected override void OnDealDamage(EventInfoDamage info)
{
    base.OnDealDamage(info);
    if (!isServer) return;

    float totalDamage = info.damage.amount + info.damage.discardedAmount;
    float healAmount = totalDamage * 0.15f; // 15% lifesteal
    
    owner.Heal(healAmount, owner);
    NotifyUse();
}
```

### Pattern: On-Kill Explosion

Explode when killing enemies:

```csharp
protected override void OnKill(EventInfoDamage info)
{
    base.OnKill(info);
    if (!isServer) return;

    Vector3 position = info.victim.agentPosition;
    float explosionDamage = 100f;
    float explosionRadius = 3f;

    // Find nearby enemies
    ListReturnHandle<Entity> handle;
    List<Entity> enemies = DewPhysics.OverlapCircleAllEntities(
        out handle, position, explosionRadius, tvDefaultHarmfulEffectTargets
    );

    foreach (var enemy in enemies)
    {
        if (enemy != info.victim) // Don't hit the corpse
        {
            DamageData explosionDmg = MagicDamage(explosionDamage, 1.0f);
            explosionDmg.SetElemental(ElementalType.Fire);
            explosionDmg.Dispatch(enemy, DamageChain.Default);
        }
    }

    handle.Return();
    NotifyUse();
}
```

### Pattern: Thorns (Counter-Attack)

Damage enemies when they hit you:

```csharp
protected override void OnTakeDamage(EventInfoDamage info)
{
    base.OnTakeDamage(info);
    if (!isServer) return;
    if (info.attacker == null) return;

    float reflectDamage = info.damage.amount * 0.20f; // 20% reflect

    DamageData thorns = MagicDamage(reflectDamage, 0.5f);
    thorns.Dispatch(info.attacker, DamageChain.Default);
    
    NotifyUse();
}
```

### Pattern: Crit Multiplier

Increase crit damage:

```csharp
protected override void OnCrit(EventInfoDamage info)
{
    base.OnCrit(info);
    if (!isServer) return;
    if (info.chain.DidReact(this)) return;
    if (!owner.CheckEnemyOrNeutral(info.victim)) return;

    // Add 50% of the original crit as extra damage
    float extraCritDamage = info.damage.amount * 0.50f;

    DamageData bonus = MagicDamage(extraCritDamage, 1.0f);
    bonus.SetSourceType(info.damage.type);
    bonus.Dispatch(info.victim, info.chain.New(this));
    
    NotifyUse();
}
```

### Pattern: Status Effect Application

Apply burn, freeze, etc.:

```csharp
protected override void OnDealDamage(EventInfoDamage info)
{
    base.OnDealDamage(info);
    if (!isServer) return;
    if (info.chain.DidReact(this)) return;

    // 30% chance to burn
    if (UnityEngine.Random.value < 0.30f)
    {
        StatusEffect.CreateStatusEffect<Se_C_Burn>(
            info.victim,  // target
            owner,        // source
            5.0f          // duration in seconds
        );
        NotifyUse();
    }
}
```

### Pattern: AoE Expansion

Expand skill radius when equipped:

```csharp
private DewCollider _originalCollider;
private Vector3 _originalScale;

public override void OnEquipSkill(SkillTrigger skill)
{
    base.OnEquipSkill(skill);
    if (!isServer) return;

    // Find the skill's range collider
    var rangeField = skill.GetType().GetField("range", 
        System.Reflection.BindingFlags.Public | 
        System.Reflection.BindingFlags.Instance);
    
    if (rangeField != null)
    {
        _originalCollider = rangeField.GetValue(skill) as DewCollider;
        if (_originalCollider != null)
        {
            _originalScale = _originalCollider.transform.localScale;
            
            // Expand by 50%
            _originalCollider.transform.localScale *= 1.5f;
        }
    }
}

public override void OnUnequipSkill(SkillTrigger skill)
{
    // Restore original size
    if (_originalCollider != null)
    {
        _originalCollider.transform.localScale = _originalScale;
        _originalCollider = null;
    }
    
    base.OnUnequipSkill(skill);
}
```

---

## Advanced Techniques

### Skill-Specific Behavior

Different effects based on which skill it's equipped to:

```csharp
private SkillTrigger _currentSkill;

public override void OnEquipSkill(SkillTrigger skill)
{
    base.OnEquipSkill(skill);
    _currentSkill = skill;
}

protected override void OnDealDamage(EventInfoDamage info)
{
    base.OnDealDamage(info);
    if (!isServer) return;
    if (info.chain.DidReact(this)) return;

    string skillType = _currentSkill?.GetType().Name;

    if (skillType == "St_C_Fireball")
    {
        // Fireball gets 50% bonus
        ApplyBonus(info, 0.50f);
    }
    else if (skillType == "St_E_ChainLightning")
    {
        // Chain Lightning gets 25% bonus
        ApplyBonus(info, 0.25f);
    }
    else
    {
        // Everything else gets 15%
        ApplyBonus(info, 0.15f);
    }
}

private void ApplyBonus(EventInfoDamage info, float multiplier)
{
    float totalDamage = info.damage.amount + info.damage.discardedAmount;
    DamageData bonus = MagicDamage(totalDamage * multiplier, 1.0f);
    bonus.Dispatch(info.victim, info.chain.New(this));
    NotifyUse();
}
```

### Stat Tracking

Track gem usage for achievements or special effects:

```csharp
private int _totalHits = 0;
private int _totalKills = 0;

protected override void OnDealDamage(EventInfoDamage info)
{
    base.OnDealDamage(info);
    if (!isServer) return;

    _totalHits++;
    
    // Every 100 hits, do something special
    if (_totalHits % 100 == 0)
    {
        // Bonus effect!
        owner.Heal(100f, owner);
        Debug.Log($"[Gem] Milestone: {_totalHits} hits!");
    }
}

protected override void OnKill(EventInfoDamage info)
{
    base.OnKill(info);
    if (!isServer) return;

    _totalKills++;
}
```

### Conditional Effects

Trigger based on player state:

```csharp
protected override void OnDealDamage(EventInfoDamage info)
{
    base.OnDealDamage(info);
    if (!isServer) return;
    if (info.chain.DidReact(this)) return;

    var hero = owner as Hero;
    if (hero == null) return;

    // Bonus if health below 50%
    if (hero.healthPercent < 0.50f)
    {
        float bonusDamage = (info.damage.amount + info.damage.discardedAmount) * 0.50f;
        DamageData bonus = MagicDamage(bonusDamage, 1.0f);
        bonus.Dispatch(info.victim, info.chain.New(this));
        NotifyUse();
    }
}
```

### Spawning Visual Effects

Create visual feedback:

```csharp
protected override void OnDealDamage(EventInfoDamage info)
{
    base.OnDealDamage(info);
    if (!isServer) return;

    // Spawn a visual effect at the hit location
    var hitEffect = DewResources.GetByShortTypeName<GameObject>("Fx_FireExplosion");
    if (hitEffect != null)
    {
        Instantiate(hitEffect, info.victim.agentPosition, Quaternion.identity);
    }
    
    NotifyUse();
}
```

### Cooldown Management

Implement per-gem cooldowns:

```csharp
private float _lastTriggerTime = -999f;
private const float COOLDOWN = 5.0f; // 5 seconds

protected override void OnDealDamage(EventInfoDamage info)
{
    base.OnDealDamage(info);
    if (!isServer) return;
    if (info.chain.DidReact(this)) return;

    // Check cooldown
    if (Time.time - _lastTriggerTime < COOLDOWN)
        return;

    _lastTriggerTime = Time.time;

    // Your effect here
    float bonusDamage = 200f;
    DamageData bonus = MagicDamage(bonusDamage, 1.0f);
    bonus.Dispatch(info.victim, info.chain.New(this));
    
    NotifyUse();
}
```

---

## Testing & Debugging

### Custom Debug Commands

Add debug commands to your mod:

```csharp
using IngameDebugConsole;

public class YourMod : ModBehaviour
{
    private void Awake()
    {
        GemForgeAPI.Initialize("com.yourname.yourmod");
        RegisterGem();
        RegisterDebugCommands();
    }

    private void RegisterDebugCommands()
    {
        DebugLogConsole.AddCommand("testgem", "Spawn test gem", 
            (int quality) => SpawnTestGem(quality));
        
        DebugLogConsole.AddCommand("checkgem", "Check gem registration",
            () => CheckRegistration());
    }

    private static void SpawnTestGem(int quality)
    {
        if (!NetworkServer.active)
        {
            Debug.LogWarning("Must be host/server");
            return;
        }

        var hero = DewPlayer.local?.hero;
        if (hero == null) return;

        quality = Mathf.Clamp(quality, 0, 100);

        var prefab = DewResources.GetByType<Gem_E_YourGem>();
        if (prefab == null)
        {
            Debug.LogError("Prefab not found!");
            return;
        }

        var gem = Dew.CreateGem(prefab, hero.position + hero.transform.forward * 2f, 
            quality, DewPlayer.local);
        
        if (gem != null)
            Debug.Log($"Spawned gem with quality {quality}");
    }

    private static void CheckRegistration()
    {
        var db = DewResources.database;
        string guid = "your-gem-guid";
        
        Debug.Log("=== Registration Check ===");
        Debug.Log($"GUID in database: {db.allGuids.Contains(guid)}");
        Debug.Log($"Type registered: {db.guidToType.ContainsKey(guid)}");
        Debug.Log($"Prefab found: {DewResources.GetByType<Gem_E_YourGem>() != null}");
    }
}
```

### Debug Logging

Add debug logs to your gem:

```csharp
protected override void OnDealDamage(EventInfoDamage info)
{
    base.OnDealDamage(info);
    if (!isServer) return;
    
    if (DewDebug.isOn)
    {
        Debug.Log($"[YourGem] Dealt {info.damage.amount} damage to {info.victim.name}");
        Debug.Log($"[YourGem] Quality: {quality}, Owner: {owner.name}");
    }
    
    // Your logic
}
```

### Unity Log File

Detailed logs are written to:
```
C:\Users\YourName\AppData\LocalLow\LizardSmoothie\Shape of Dreams\Player.log
```

Tail this file with [BareTail](https://www.baremetalsoft.com/baretail/) for real-time log viewing.

---

## Game Mechanics Reference

### Damage System

#### Creating Damage

```csharp
// Magic damage (scales with AP)
DamageData magicDmg = MagicDamage(100f, 1.0f);

// Physical damage (scales with AD)
DamageData physicalDmg = PhysicalDamage(100f, 1.0f);

// True damage (ignores resistance)
DamageData trueDmg = TrueDamage(100f, 1.0f);
```

**Proc Coefficient:**
- `1.0f` = Full effect (can trigger other gems, status effects)
- `0.5f` = Half effect
- `0.0f` = No secondary effects (just damage)

#### Setting Damage Properties

```csharp
DamageData dmg = MagicDamage(100f, 1.0f);

// Set element
dmg.SetElemental(ElementalType.Fire);

// Set damage type
dmg.SetSourceType(DamageType.MagicDamage);
dmg.SetSourceType(DamageType.PhysicalDamage);
dmg.SetSourceType(DamageType.TrueDamage);

// Set amount origin (for display/tracking)
dmg.SetAmountOrigin(info.damage);

// Add flags
dmg.AddFlag(DamageFlag.CannotCrit);
dmg.AddFlag(DamageFlag.IsDoT);

// Dispatch
dmg.Dispatch(target, chain);
```

#### Damage Types

```csharp
public enum DamageType
{
    PhysicalDamage,  // Scales with AD, reduced by armor
    MagicDamage,     // Scales with AP, reduced by MR
    TrueDamage       // Ignores all resistance
}
```

#### Elemental Types

```csharp
public enum ElementalType
{
    Physical,   // No element
    Fire,       // Burns
    Cold,       // Slows/Freezes
    Light,      // Holy damage
    Dark        // Unholy damage
}
```

### Status Effects

#### Applying Status Effects

```csharp
// Burn for 5 seconds
StatusEffect.CreateStatusEffect<Se_C_Burn>(
    target,   // who gets the effect
    owner,    // who caused it
    5.0f      // duration in seconds
);

// Freeze
StatusEffect.CreateStatusEffect<Se_C_Freeze>(target, owner, 3.0f);

// Slow
StatusEffect.CreateStatusEffect<Se_C_Slow>(target, owner, 2.0f);

// Stun
StatusEffect.CreateStatusEffect<Se_C_Stun>(target, owner, 1.5f);

#### Common Status Effects

| Type | Effect | Good For |
|------|--------|----------|
| `Se_C_Burn` | Fire DoT | Damage over time |
| `Se_C_Freeze` | Frozen in place | Hard CC |
| `Se_C_Slow` | Movement slow | Soft CC |
| `Se_C_Stun` | Cannot act | Hard CC |
| `Se_C_Bleed` | Physical DoT | Bleed effects |

### Physics & Targeting

#### Finding Entities in Radius

```csharp
// Find all entities in radius
ListReturnHandle<Entity> handle;
List<Entity> entities = DewPhysics.OverlapCircleAllEntities(
    out handle,
    position,              // center
    radius,                // search radius
    tvDefaultHarmfulEffectTargets  // target filter
);

// Use the entities
foreach (var entity in entities)
{
    // Do something
}

// CRITICAL: Always return the handle!
handle.Return();
```

#### Target Filters

```csharp
// Common filters
tvDefaultHarmfulEffectTargets  // Enemies
tvDefaultHelpfulEffectTargets  // Allies
tvDefaultNeutralEffectTargets  // Neutral
```

#### Entity Checks

```csharp
// Check if entity is an enemy
if (owner.CheckEnemyOrNeutral(entity))
{
    // It's hostile
}

// Check if entity is an ally
if (owner.CheckFriendly(entity))
{
    // It's friendly
}

// Check if entity is dead
if (entity.isDead)
{
    // Don't target corpses
}
```

### Hero API

```csharp
var hero = owner as Hero;
if (hero != null)
{
    // Health
    float currentHP = hero.health;
    float maxHP = hero.maxHealth;
    float healthPercent = hero.healthPercent; // 0.0 to 1.0
    
    // Healing
    hero.Heal(100f, owner);
    
    // Stats
    float attackDamage = hero.attackDamage;
    float abilityPower = hero.abilityPower;
    float critChance = hero.critChance;
    float critDamage = hero.critDamage;
    float attackSpeed = hero.attackSpeed;
    float moveSpeed = hero.moveSpeed;
    
    // Position
    Vector3 pos = hero.agentPosition;
    Vector3 forward = hero.transform.forward;
}
```

### Chain System (Preventing Recursion)

The chain system prevents infinite loops when gems trigger other gems.

```csharp
protected override void OnDealDamage(EventInfoDamage info)
{
    base.OnDealDamage(info);
    if (!isServer) return;
    
    // Check if this gem already reacted in this damage chain
    if (info.chain.DidReact(this)) return;
    
    // Create new damage
    DamageData bonus = MagicDamage(100f, 1.0f);
    
    // Dispatch with NEW chain (marks this gem as "used")
    bonus.Dispatch(target, info.chain.New(this));
}
```

**Chain Types:**
- `info.chain.New(this)` - Create new chain with this gem marked
- `DamageChain.Default` - Fresh chain (for explosions, thorns)

---

## Troubleshooting

### Gem Doesn't Spawn

**Check:**
1. Is `ExcludeFromPool = false`?
2. Does `checkgem` command show all `true`?
3. Are you the host/server (not client in multiplayer)?
4. Check console for errors during registration

**Fix:**
```csharp
// Verify registration
var prefab = DewResources.GetByType<Gem_E_YourGem>();
Debug.Log($"Prefab found: {prefab != null}");
```

### Gem Spawns But Doesn't Work

**Check:**
1. Is `isServer` check present?
2. Is hook being called? Add debug logs
3. Is chain check preventing trigger?
4. Is enemy/ally check wrong?

**Debug:**
```csharp
protected override void OnDealDamage(EventInfoDamage info)
{
    Debug.Log("[YourGem] OnDealDamage called!");
    
    if (!isServer)
    {
        Debug.Log("[YourGem] Not server, skipping");
        return;
    }
    
    if (info.chain.DidReact(this))
    {
        Debug.Log("[YourGem] Already reacted, skipping");
        return;
    }
    
    Debug.Log("[YourGem] Applying effect!");
    // Your logic
}
```

### NullReferenceException on Quality Change

**Cause:** `netIdentity` not initialized yet

**Fix:** Use `GemForgeGem` base class or add this override:
```csharp
protected override void OnQualityChange(int oldQuality, int newQuality)
{
    try
    {
        var _ = netIdentity;
        base.OnQualityChange(oldQuality, newQuality);
        // Your logic
    }
    catch (NullReferenceException)
    {
        // Not initialized yet
    }
}
```

### Infinite Damage Loop

**Cause:** Not using chain system correctly

**Fix:**
```csharp
// Always check chain
if (info.chain.DidReact(this)) return;

// Always use chain.New(this) when dispatching
bonus.Dispatch(target, info.chain.New(this));
```

### Gem Not Showing in Deja Vu

**Add to profileStats:**
```csharp
if (DewSave.profileStats?.gems != null)
{
    if (!DewSave.profileStats.gems.ContainsKey("Gem_E_YourGem"))
    {
        var data = new DewProfileStats.ItemData { wins = 1 };
        DewSave.profileStats.gems.Add("Gem_E_YourGem", data);
    }
}
```

### Build Errors

**CS7069 (ReadOnlySpan):**
- Check `<TargetFramework>net4.8</TargetFramework>`

**CS0012 (SerializedScriptableObject):**
- Add Sirenix.Serialization.dll reference

**CS1929 (Contains not found):**
- Add `using System.Linq;`

---

## API Reference

### GemForgeAPI

```csharp
public static class GemForgeAPI
{
    // Initialize framework (call once in Awake)
    public static void Initialize(string harmonyId);
    
    // Register a custom gem
    public static void RegisterGem<T>(GemDefinition definition) where T : Gem;
    
    // Get gem definition by type name
    public static bool TryGetDefinition(string typeName, out GemDefinition definition);
    
    // Get all registered gems
    public static IEnumerable<string> GetRegisteredGems();
    
    // Enable debug logging
    public static bool EnableDebugLogs { get; set; }
}
```

### GemDefinition

```csharp
public class GemDefinition
{
    public string Guid { get; set; }                    // Unique ID
    public uint AssetId { get; set; }                   // Network asset ID
    public Rarity Rarity { get; set; }                  // Common/Rare/Epic/Legendary
    public bool ExcludeFromPool { get; set; }           // Hide from loot?
    public string Name { get; set; }                    // Display name
    public string ShortDescription { get; set; }        // Tooltip
    public string Description { get; set; }             // Full description
    public string Memory { get; set; }                  // Deja Vu text
    public string Template { get; set; }                // Name template
    public Texture2D IconTexture { get; set; }          // Icon (64x64)
    
    // Generate asset ID from GUID
    public static uint GenerateAssetId(string guid);
}
```

### GemForgeGem (Base Class)

```csharp
public abstract class GemForgeGem : Gem
{
    // Handles OnQualityChange with null safety
    protected override void OnQualityChange(int oldQuality, int newQuality);
    
    // Override this for quality-dependent logic
    protected virtual void OnQualityChangeImpl(int oldQuality, int newQuality);
}
```

### Gem Base Methods

```csharp
public class Gem : Item
{
    // Properties
    public int quality { get; set; }                    // 0-100
    public Rarity rarity { get; set; }
    public Entity owner { get; }                        // Who owns this gem
    public bool isServer { get; }                       // Server-side check
    
    // Damage creation
    protected DamageData MagicDamage(float amount, float procCoeff);
    protected DamageData PhysicalDamage(float amount, float procCoeff);
    protected DamageData TrueDamage(float amount, float procCoeff);
    
    // Tracking
    protected void NotifyUse();                         // Mark gem as used
    
    // Hooks (override these)
    protected virtual void OnDealDamage(EventInfoDamage info);
    protected virtual void OnTakeDamage(EventInfoDamage info);
    protected virtual void OnKill(EventInfoDamage info);
    protected virtual void OnCrit(EventInfoDamage info);
    protected virtual void OnEquipSkill(SkillTrigger skill);
    protected virtual void OnUnequipSkill(SkillTrigger skill);
    protected virtual void OnCastStart(EventInfoCast info);
    protected virtual void OnCastComplete(EventInfoCast info);
    protected virtual void OnCastCancel(EventInfoCast info);
    protected virtual void OnQualityChange(int oldQuality, int newQuality);
}
```

### EventInfoDamage

```csharp
public struct EventInfoDamage
{
    public Entity attacker;              // Who dealt damage
    public Entity victim;                // Who took damage
    public FinalDamageData damage;       // Final damage info
    public DamageChain chain;            // Chain tracking
    public bool isCrit;                  // Was it a crit?
    public bool isKillingBlow;          // Did it kill?
}

public struct FinalDamageData
{
    public float amount;                 // Damage dealt
    public float discardedAmount;        // Overkill damage
    public DamageType type;              // Physical/Magic/True
    public ElementalType? elemental;     // Element (if any)
}
```

### DamageChain

```csharp
public class DamageChain
{
    // Check if gem already reacted
    public bool DidReact(Gem gem);
    
    // Create new chain with gem marked
    public DamageChain New(Gem gem);
    
    // Fresh chain
    public static DamageChain Default { get; }
}
```

---

## Examples Repository

### Complete Gem Examples

All examples WILL BE available in the `/examples` folder once I MAKE IT:

1. **Vampirism** - Simple lifesteal
2. **Expression** - Random elemental damage
3. **Retaliation** - Thorns effect
4. **Executioner** - Bonus damage to low-health enemies
5. **Chain Reaction** - On-kill explosions
6. **Frostbite** - Freeze on crit
7. **Berserker** - More damage at low health
8. **Lucky Strike** - Random mega-crits
9. **Plague Doctor** - Spread poison on kill
10. **Time Warp** - Cooldown reduction

### Template Gem

Use this as a starting point:

```csharp
using DewInternal;
using GemForge;
using UnityEngine;

public class Gem_E_Template : GemForgeGem
{
    // Constants
    private const float BASE_EFFECT = 0.20f;
    private const float QUALITY_SCALING = 0.001f;

    // State tracking (if needed)
    private SkillTrigger _equippedSkill;

    // Helper methods
    private float GetEffectMultiplier()
    {
        return BASE_EFFECT + (quality * QUALITY_SCALING);
    }

    // Hooks
    protected override void OnDealDamage(EventInfoDamage info)
    {
        base.OnDealDamage(info);
        if (!isServer) return;
        if (info.chain.DidReact(this)) return;
        if (!owner.CheckEnemyOrNeutral(info.victim)) return;

        // Your effect here
        
        NotifyUse();
    }
}

// Registration
public class TemplateMod : ModBehaviour
{
    private void Awake()
    {
        GemForgeAPI.Initialize("com.yourname.template");
        
        GemForgeAPI.RegisterGem<Gem_E_Template>(new GemDefinition
        {
            Guid = "mod-gem-e-template-guid",
            AssetId = GemDefinition.GenerateAssetId("mod-gem-e-template-guid"),
            Rarity = Rarity.Epic,
            Name = "Template",
            ShortDescription = "Does something cool",
            Description = "Your description here",
            Memory = "Your flavor text here"
        });
    }
}
```

---

## Contributing

Found a bug? Have a suggestion? Want to contribute examples?

**GitHub:** https://github.com/zaKroWz/GemForgeAPI
**Discord:** _zakrow

---

## License

GemForge is released under the MIT License. Free to use, modify, and distribute.

---

## Changelog

### v1.0.0 (2024-01-18)
- Initial release
- Core framework with database registration
- Profile integration (Deja Vu support)
- Localization system
- `GemForgeGem` base class
- Comprehensive documentation
- 10 example gems

---

**Happy modding! 🎮💎**