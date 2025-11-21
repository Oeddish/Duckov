# Duckov Mod Solution

This repository hosts multiple Escape from Duckov mods in a single solution. Shared build logic handles:
- Game/Steam/local path configuration
- Asset (info.ini / preview.png) discovery
- Automatic deployment of built mod DLLs and associated assets to local and optional Steam Workshop folders

Original modding example and API references:
https://github.com/xvrsl/duckov_modding/

## Contained Mods 

| Project | Purpose |
|---------|---------|
| Generic | Generic template to be modified to create new mods. |
| DisplayItemValue | Copy from modding example to show how multiple mods are hosted. |

(Each mod has its own README.md for detailed documentation.)

## Path Configuration

Global path properties live in `Directory.Build.props`:

| Property | Default (Windows) | Default (macOS) | Purpose |
|----------|-------------------|-----------------|---------|
| DuckovPath | `$(ProgramFiles(x86))\Steam\steamapps\common\Escape from Duckov` | `/Users/Somebody/Library/Application Support/Steam/steamapps/common/Escape from Duckov` | Root game install (used to reference game DLLs). |
| SubPath | `\Duckov_Data\Managed\` | `/Duckov.app/Contents/Resources/Data/Managed/` | Managed assemblies folder under game install. |
| SteamWorkshopPath | `$(ProgramFiles(x86))\Steam\steamapps\workshop\content\3167020` | macOS analog path | Workshop root for app id 3167020 (used when publishing). |
| LocalModPath | `$(UserProfile)\AppData\LocalLow\TeamSoda\Duckov` | `/Users/Somebody/Library/Application Support/TeamSoda/Duckov` | Local user data (manual mods directory). |

Override any of these per machine by creating `Directory.Build.props.local` (ignored by Git). An example template is tracked as `Directory.Build.props.local.example`.

### Example Local Override File (Copy & Rename)

Example `Directory.Build.props.local`:
```xml
<Project>
  <PropertyGroup>
    <DuckovPath>C:\Path\To\Your\Duckov</DuckovPath>
    <SubPath>\Duckov_Data\Managed\</SubPath>
    <SteamWorkshopPath>C:\Path\To\Your\Steam\Workshop\Content\3167020</SteamWorkshopPath>
    <LocalModPath>C:\Path\To\Your\Local\Mods\Directory</LocalModPath>
  </PropertyGroup>
</Project>
```

## Mod Discovery Pattern

A project is treated as a mod when it has an `info.ini` in its root folder.

Optional `preview.png` in the same folder is also copied if present.

## Deployment Flags

Centralized in `Directory.Build.targets`:

| Flag | Default | Effect |
|------|---------|--------|
| DeployLocal | true | Copy DLL + PDB + info.ini (+ preview.png) to `<LocalModPath>\Mods\<LocalModFolderName>`. |
| DeployWorkshop | false | Copy same artifacts to `<SteamWorkshopPath>\<PublishedFileId>` (requires `PublishedFileId`). |

Defaults:
- `ModAssetRoot = ReleaseExample/<ProjectName>`
- `LocalModFolderName = <ProjectName>` (override in the project file if needed)
- `PublishedFileId` must be set in the project file to enable Workshop deployment.

Ways to toggle:
1. Command line:
   - Local only (default): `dotnet build`
   - Enable Workshop: `dotnet build -c Release /p:DeployWorkshop=true`
   - Disable local: `dotnet build /p:DeployLocal=false`
2. Per project (temporary change):
   - Add inside the mod `.csproj`:
     ```xml
     <PropertyGroup>
       <DeployWorkshop>true</DeployWorkshop>
     </PropertyGroup>
     ```

## Adding a New Mod Project

1. Create a new Class Library targeting `netstandard2.1`.
2. Place `info.ini` (and optional `preview.png`) under `ReleaseExample/<YourProjectName>/`.
3. Add references to needed Duckov DLLs (wildcards already handled if you reuse existing project pattern).
4. Optionally set:
   - `ModAssetRoot` to change the asset root folder.
   - `LocalModFolderName` to change the local mod folder name.
   - `PublishedFileId` to enable Workshop deployment.

(Full automation for version stamping can be added later.)

## Git Hygiene

- Do not commit `Directory.Build.props.local`.
- Root `.gitignore` already ignores build output (`bin/`, `obj/`) and local overrides.
- Tag releases after successful in-game test runs.

## Troubleshooting

| Issue | Resolution |
|-------|------------|
| DLL not copied | Ensure `info.ini` exists (mod discovery condition). |
| Workshop deployment skipped | Set `/p:DeployWorkshop=true` AND define `<PublishedFileId>` in project. |
| Wrong game path | Override `DuckovPath` in `Directory.Build.props.local`. |
| Duplicate mod load | Avoid deploying same DLL to both Workshop + Local unless intentional. |