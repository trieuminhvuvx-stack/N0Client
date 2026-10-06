# N0Client 1.6.2

N0Client is a renamed/rebranded build of the supplied 67Client Fabric source for Minecraft 1.21.11.
The existing ClickGUI, module manager, settings/config system, HUD, themes, and existing modules are preserved.

## Included
- N0Client branding and mod id: `n0client`
- Fabric client entrypoint: `dev.sixseven.N0Client`
- Existing ClickGUI with category panels, search, settings and config persistence
- Existing modules such as AutoClicker, AutoWalk, Freecam, FreeLook, ESP modules, ChunkFinder,
  SusChunkFinder, combat/misc/render modules, HUD and more
- Existing NanoVG runtime resources
- Existing mixins and configuration system

## Build

Requirements: Java 21 and a working Gradle environment.

```bash
./gradlew build
```

The remapped jar is produced under `build/libs/`.

The build cannot be performed in this sandbox because the Gradle distribution must be downloaded
from the internet and network access is unavailable here.

## Supplied SusChunkFinder.java

The extra Java file supplied with the request is preserved under `reference/SusChunkFinder.java`.
It uses a different API (`com.water.*`) and therefore cannot be dropped into this project unchanged.
The existing N0Client SusChunkFinder module remains enabled and is already connected to the ClickGUI.
