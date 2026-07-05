# Building UE 5.6 from Source — Summary Guide

## 1. Get the source
```
git clone --branch 5.6 https://github.com/EpicGames/UnrealEngine.git
cd UnrealEngine
Setup.bat
GenerateProjectFiles.bat
```

## 2. Install the correct MSVC toolset
Default VS2022 "Latest" toolset (e.g. 14.44) is not preferred by UE 5.6 and can cause spurious compile errors (e.g. `C4756 overflow in constant arithmetic` from `INFINITY` macro casts in `AnimNextAnimGraph`).

Via Visual Studio Installer → Individual Components, install:
```
MSVC v143 - VS 2022 C++ x64/x86 build tools (v14.38-17.8)
```

## 3. Install a compatible Windows SDK
Newest bundled SDK (e.g. `10.0.26100.x`) can also cause build errors (linker issues in plugins like `NNERuntimeORT`). Install a known-good SDK:
```
Windows 11 SDK (10.0.22621.0)
```

## 4. Pin toolchain + SDK versions for this engine install
Create:
```
<EngineRoot>\Engine\Saved\UnrealBuildTool\BuildConfiguration.xml
```
Content:
```xml
<?xml version="1.0" encoding="utf-8" ?>
<Configuration xmlns="https://www.unrealengine.com/BuildConfiguration">
	<WindowsPlatform>
		<CompilerVersion>14.38.33130</CompilerVersion>
		<WindowsSdkVersion>10.0.22621.0</WindowsSdkVersion>
	</WindowsPlatform>
</Configuration>
```
Using the engine-local `Saved\` path (not `%APPDATA%`) keeps this pinned version scoped to this specific engine install only.

Re-run `GenerateProjectFiles.bat` after creating this file.

## 5. Build the engine
Open `UE5.sln` → set Solution Configuration to `Debug Editor` (full engine debug symbols, no inlining) or `Development Editor` (faster, optimized) → `Win64` → build the `UE5` project.

## 6. Link your game project to this build
- Right-click `.uproject` → Switch Unreal Engine version → select this source build path
- Right-click `.uproject` → Generate Visual Studio project files
- Open your project's `.sln`, pick `DebugGame Editor` or `Debug Editor`

## 7. Launching
- Double-clicking `.uproject` may fail ("Failed to launch editor") if version selector isn't registered — run `UnrealVersionSelector-Win64-Shipping.exe /register` as admin, or launch directly:
```
Engine\Binaries\Win64\UnrealEditor.exe "path\to\YourProject.uproject"
```
- Double-click launch uses `UnrealEditor.exe` (no suffix), which is only produced by a `Development Editor` build. If you only built `Debug Editor`/`DebugGame Editor`, that exe won't exist and double-click will fail even with a correct `EngineAssociation` GUID registered in `HKEY_CURRENT_USER\SOFTWARE\Epic Games\Unreal Engine\Builds`. Build `Development Editor` once to get double-click launching working, or just launch the suffixed exe (`UnrealEditor-Win64-Debug.exe`, etc.) directly.
