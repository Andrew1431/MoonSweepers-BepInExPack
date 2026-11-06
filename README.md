# BepInEx pack for Moon Sweepers

A test build of BepInEx 6 IL2CPP that runs Moon Sweepers (Unity 6000.6.0f1, IL2CPP metadata v108, Steam app 5335510)
without ModEngine, plus everything needed to take it over and maintain it from another account. It's unofficial and
not supported by the BepInEx team.

**Download:** [BepInExPack_MoonSweepers-0.1.0.zip](https://github.com/Andrew1431/MoonSweepers-BepInExPack/releases/download/v0.1.0/BepInExPack_MoonSweepers-0.1.0.zip)

SHA-256: `0b17928d590930efcebabfeffab396406e5504a552a8616ee320e6931a5e21b1`

The zip uses the same layout as `BepInEx-BepInExPack_IL2CPP`, plus a `licenses` folder. It hasn't been uploaded to
Thunderstore.

## Status

| Check | Result |
|---|---|
| This zip boots the game with GarageLib, Late Game Vehicles and Shared Upgrades release builds | Works, 0 errors |
| Playing a save on this zip | Not tested |
| Playing saves on an earlier hand-assembled build of the same changes | Works, 0 errors |
| Multiplayer | Not tested |
| Other Unity versions, including 6000.5 | Not tested |
| Linux | Not tested |

## What's in the changes

| Repo and branch | Based on | Commits |
|---|---|---|
| [Andrew1431/Il2CppInterop `v108`](https://github.com/Andrew1431/Il2CppInterop/tree/v108) | [BepInEx/Il2CppInterop#292](https://github.com/BepInEx/Il2CppInterop/pull/292), head `72a5fd3` | Generator moved to AsmResolver 6.0.1. Runtime support for the Unity 6000.6 class layout. |
| [Andrew1431/BepInEx `v108`](https://github.com/Andrew1431/BepInEx/tree/v108) | master `dbc82f0` | Cpp2IL bumped to 2022.1.0-development.1743. |
| [Andrew1431/BepInEx `moonsweepers-pack`](https://github.com/Andrew1431/BepInEx/tree/moonsweepers-pack) | `v108` above | Pack only: references the Il2CppInterop build, adds the build script and pack README. |

The two `v108` branches are what an upstream PR would use. `moonsweepers-pack` exists only to produce this zip until
upstream ships these changes.

## Take it over

Both options below are written for your own account. They haven't been run from an account other than ours.

### Option 1: fork and copy the branches

Add `--org thunderstore-io` to `gh repo fork` to create the forks in that organization.

```
gh repo fork BepInEx/BepInEx --clone
cd BepInEx
git remote add andrew https://github.com/Andrew1431/BepInEx.git
git fetch andrew
git switch -c moonsweepers-pack andrew/moonsweepers-pack
git push -u origin moonsweepers-pack
cd ..

gh repo fork BepInEx/Il2CppInterop --clone
cd Il2CppInterop
git remote add andrew https://github.com/Andrew1431/Il2CppInterop.git
git fetch andrew
git switch -c v108 andrew/v108
git push -u origin v108
cd ..
```

### Option 2: apply the patches

The [`patches`](patches) folder has the same commits. Use `--keep-cr` because these repos have CRLF files.

```
# In a clone of BepInEx/BepInEx
git switch -c moonsweepers-pack dbc82f05b8ea34833de5b6a723eb60dd8398e0b1
git am --keep-cr path/to/patches/BepInEx/*.patch

# In a clone of BepInEx/Il2CppInterop
git fetch origin pull/292/head
git switch -c v108 72a5fd3cefbc580e1311d2e8242673e23e432023
git am --keep-cr path/to/patches/Il2CppInterop/*.patch
```

We checked that the patches apply on those commits and produce the same files as the branches.

### Build the pack from your forks

On Windows with git and a .NET SDK that builds net8.0 (we used SDK 10):

```
.\BepInEx\build-moonsweepers-pack.ps1 -WorkDir ..\pack-build -Owner YOUR_GITHUB_ACCOUNT -PackVersion 0.1.0
```

It clones both branches from that account, packages Il2CppInterop into a local NuGet feed as `1.5.3-moonsweepers.1`, and
runs BepInEx's normal build, which downloads doorstop, dobby and the .NET runtime from their official releases. It then
writes `BepInExPack_MoonSweepers-<version>.zip`, with the website link and README pointing at your account.

The script removes Il2CppInterop's `global.json` in its working copy, because that file asks for .NET SDK 8. The icon is
a placeholder drawn by the script, to avoid passing the pack off as official BepInEx.

## Known issues for review

- The AsmResolver 6 port is a compatibility shim (`Il2CppInterop.Generator/Extensions/AsmResolver6Compat.cs`). It works,
  but a direct port may be preferable upstream.
- The generator reported 45 fields and 1,166 methods failing to unstrip, and 4,715 IL bodies failing. We don't know
  whether that's normal for this game.
- Obfuscated interop names differ from the set ModEngine's loader generates (`_PDM_0`, `add_`, `rem_`), so mods written
  against those names could break. Mods compiled from source may also need `list.get_Item(i)` changed to `list[i]`.
- `ClassInjector.Debug.cs` has the 6000.6 interface fallback, but that code path has only been compiled, not run.

We measured the 6000.6 class layout from the running game through the public il2cpp API. ModEngine wasn't decompiled or
copied.

## Credit and licences

Built on [Cpp2IL](https://github.com/SamboyCoding/Cpp2IL) (MIT), [BepInEx](https://github.com/BepInEx/BepInEx)
(LGPL-2.1) and [Il2CppInterop](https://github.com/BepInEx/Il2CppInterop) (LGPL-3.0), and on the Unity 6000.5 work by
RTBRuhan and XLG-WSM in Il2CppInterop PRs #292 and #285. The patches are licensed under the licence of the project they
apply to; see [`licenses`](licenses).
