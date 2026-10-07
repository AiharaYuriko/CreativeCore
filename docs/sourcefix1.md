# Connected model face collection: sourcefix1

This experimental Minecraft 1.21.1 / NeoForge change lets connected texture models produce their source quads before `RenderBox` clips them to a tile. It also invalidates a box's cached quads when its face definition changes.

Baseline: CreativeMD/CreativeCore `1.21`, commit `cdafb249972c864dce1d21397c654c9e2f88f265` (2.13.50).

## Implementation

- `ModelQuadSource` provides an optional collector interface. A provider returns `null` for an unhandled model; an empty list remains an intentional result. The default collector uses the existing NeoForge `BakedModel.getQuads` entry point.
- Unculled quads enter the same clipping path as direction-specific quads, grouped by their actual face direction. The per-render-layer cache belongs to one prepared model evaluation and uses a position-derived random seed.
- `RenderBox.setFace` clears cached quads when the actual face definition changes.
- `LevelAccessorFake` exposes its parent level, material position and material state so optional collectors can obtain a consistent view without accessor mixins.

The new collector classes have no LittleTiles, Yuushya, Continuity or FRAPI dependency. Backend registration and material-specific adaptation belong to the consuming mod.

## Paired implementation

Use with [the LittleTiles source change](https://github.com/AiharaYuriko/LittleTiles/tree/codex/source-native-connected-fix). Both repositories are built together in the upstream [ForgeMods development setup](https://github.com/CreativeMD/ForgeMods); use the paired source branches rather than an unmodified released Core JAR when building LittleTiles.

The tested build selected only `:LittleTiles:build` and `:CreativeCore:build`. No standalone Yuushya/LT compatibility project was evaluated or packaged. Java 21, Minecraft 1.21.1 and NeoForge 21.1.233 were used.

## Validation

The full paired source build passed. An external offline regression harness, without standalone compatibility JARs or their class directories, passed 231 native/package assertions, 28 material-state assertions, 40,000 independent coverage raster/axis checks and six real Sodium 0.8.13 emitter cases.

The harness uses test-only render-layer identities and extracted-method fixtures for game-dependent checks; those classes are not in the mod JARs. This is not a Minecraft launch or Mixin application test. Actual connected-texture scenes, resource reloads and world reloads still need in-game verification.
