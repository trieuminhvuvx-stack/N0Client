# N0Client — recovered, deobfuscated source

Rebuilt Fabric mod source for **N0Client 1.6.2** (Minecraft **1.21.11**, Fabric Loader **0.19.3**,
Fabric API **0.141.6+1.21.11**, Java 21, Loom 1.17.13 / Gradle 9.5.1), recovered from the shipped
`n0client.jar` and deobfuscated back into readable, buildable Java.

The original jar was **not** name-obfuscated (packages/classes/methods kept readable names). It had
three protective layers, all of which are handled here:

| Layer | What it was | What we did |
|---|---|---|
| Minecraft mappings | classes were in **intermediary** namespace (normal for Loom builds) | remapped to **Yarn** (`1.21.11+build.6`) with TinyRemapper, so the source compiles against named mappings |
| String encryption | ~1,850 string literals XOR-encrypted through `dev.sixseven.rt.Deobf.decrypt(...)` | inlined the plaintext (key recovered: `s3v3n_v31l_67c!ent_x0r_k3y_9E3779B1`) |
| Mixin annotations | `@Shadow` / `@Inject(method=...)` / `@At(target=...)` carried intermediary names (static remap) | resolved every token back to Yarn names via the tiny mapping + class hierarchy |

## Project layout

```
build.gradle / settings.gradle / gradle.properties   Loom build (versions match the original build)
gradlew / gradle/wrapper                             Gradle 9.5.1 wrapper
src/main/java/dev/sixseven/...                       deobfuscated source (263 classes)
src/main/resources/                                  fabric.mod.json, mixins, assets, nested NanoVG jars
build/libs/n0client-1.6.2.jar                        built + remapped jar (no refmap: static mixin remap, like the original)
tools/                                               recovery scripts (mixin_remap.py, deobf_strings.py, DumpHierarchy.java)
```

Build with:

```bash
./gradlew build        # jar lands in build/libs/n0client-1.6.2.jar
./gradlew runClient    # launch a dev instance (no license required)
```

## License / DRM status

* **Authentication has been removed.** The `dev.sixseven.license` package (`LicenseGuard`,
  `HardwareId`, `SignatureVerifier`, `EncryptedClassLoader`, `CryptoBox`, `ProtectedContent`, ...),
  the `n0client.build` license-key file, `assets/n0client/build.json`, and the encrypted
  `assets/n0client/enc/` blobs are gone. `N0Client.onInitializeClient()` no longer calls
  `LicenseGuard.enforce(...)` / `ProtectedContent.init(...)` / `ProtectedContent.activate(...)`, so
  the client starts with no license server, HWID, or key check.
* **Aim assist logic was not recoverable.** The actual aim computation lived in the encrypted
  `dev.sixseven.secured.AimAssistLogic` class (an AES-128-GCM blob keyed by the license server). That
  source cannot be recovered from the jar, so `AimAssistModule.logic` is now `null` and the module is
  a no-op until an `AimAssistCompute` implementation is provided (`dev.sixseven.module.AimAssistCompute`
  documents the interface).

## Notes on the recovered source

* Decompiled with the IntelliJ Fernflower decompiler after Yarn remapping; inner classes merged into
  their outer files.
* A handful of decompiler artifacts were hand-fixed to compile:
  * mixin `this`-casts rewritten as `(Target)(Object)this` / `(Object)this instanceof ...`
  * constructors reordered to put `super()` first (Fernflower emitted pre-`super()` statements)
  * a few raw-generic sites given explicit types
* `dev.sixseven.rt.Deobf` is kept (now unused) so nothing else in the tree references a missing class.

## Recovery pipeline (for reference)

1. `java -jar tools/tiny-remapper-fat.jar work work-named tools/yarn.jar intermediary named [classpath]`
   — remap the extracted jar from intermediary to Yarn (classpath = Loom's `minecraft-merged-intermediary`
   jar + Fabric jars; get it via `./gradlew printClasspath`).
2. Decompile `work-named` with Fernflower.
3. `python tools/mixin_remap.py <tiny> <hierarchy> <src>` — convert mixin annotation tokens to Yarn.
4. `python tools/deobf_strings.py <src>` — inline XOR-decrypted strings.
