# Integrated API (Minecraft 1.21.1)

Library mod for building advanced structure mods across Fabric, Forge, and NeoForge.

## What Integrated API Adds

Integrated API provides:

- Custom structure types built on jigsaw generation with extra controls.
- Custom structure processors, placement modifiers, structure placements, and rule tests.
- A condition/action/target-selector system for per-piece logic.
- Extended structure block and jigsaw limits.
- Datapack-driven systems for spawners, workstation replacement, map trades, and piece count constraints.
- Optional overlap prevention and structure disabling via tags.
- Async locate helpers for map trades.
- Enhanced terrain adaptation (custom bearding/carving kernels).

## Core Registries and IDs

### Structure Types (`BuiltInRegistries.STRUCTURE_TYPE`)

- `integrated_api:generic_structure` -> `JigsawStructure`
- `integrated_api:optional_dependency_structure` -> `OptionalDependencyStructure`
- `integrated_api:mod_adaptive_structure` -> `ModAdaptiveStructure`
- `integrated_api:nether_structure` -> `NetherJigsawStructure`
- `integrated_api:over_lava_nether_structure` -> `OverLavaNetherStructure`
- `integrated_api:biome_facing_structure` -> `BiomeFacingStructure`

### Structure Placement Types (`BuiltInRegistries.STRUCTURE_PLACEMENT`)

- `integrated_api:advanced_random_spread` -> `AdvancedRandomSpread`
- `integrated_api:stronghold` -> `StrongholdPlacement`

### Placement Modifiers (`BuiltInRegistries.PLACEMENT_MODIFIER_TYPE`)

- `integrated_api:minus_eight_placement`
- `integrated_api:unlimited_count`
- `integrated_api:snap_to_lower_non_air_placement`
- `integrated_api:min_distance_from_world_origin_placement`

### Structure Processors (`BuiltInRegistries.STRUCTURE_PROCESSOR`)

- `block_removal_post_processor` -> remove listed blocks during finalize pass.
- `flood_with_water_processor` -> flood up to a Y level, optionally waterlog, surround with cracked stone bricks.
- `replace_air_only_processor` -> keep existing world blocks unless world block is air.
- `replace_liquids_only_processor` -> keep existing world blocks unless world block is liquid.
- `spawner_randomizing_processor` -> replace spawner NBT from datapack-weighted mob lists.
- `fill_end_portal_frame_processor` -> randomize Eyes of Ender in frames.
- `remove_floating_blocks_processor` -> cleanup unsupported plants/blocks after placement.
- `close_off_fluid_sources_processor` -> seal nearby source fluids with weighted replacement blocks.
- `close_off_air_sources_processor` -> seal nearby air/liquid mismatch around fluid interiors.
- `random_replace_with_properties_processor` -> random block replacement preserving shared state properties.
- `waterlogging_fix_processor` -> force `waterlogged=false`.
- `waterlogging_when_replacing_water_processor` -> set waterlogged based on replaced world water.
- `capped_structure_surface_processor` -> run delegate processor only on top-cap style surface candidates.
- `post_process_list_processor` -> run a list of delegate processors in sequence.
- `windmill_bearing_processor` -> queue Create windmill assembly.
- `fluid_tank_processor` -> remove problematic tank NBT (`Luminosity`) and tick block.
- `mechanical_bearing_processor` -> queue Create mechanical bearing assembly.
- `clockwork_bearing_processor` -> queue Create clockwork bearing assembly.
- `elevator_pulley_processor` -> queue Create elevator pulley assembly.
- `tick_blocks_processor` -> schedule ticks on configured blocks.
- `integrated_block_replace_processor` -> conditional/random block swaps based on loaded mods.
- `workstation_processor` -> replace workstation placeholder blocks from datapack entries.

### Rule Tests / Pos Rule Tests

- `integrated_api:matter_phase_rule_test`
- `integrated_api:piece_origin_axis_aligned_linear_pos_rule_test`
- `integrated_api:y_value_pos_rule_test`

### Structure Pool Element Type

- `integrated_api:integrated_api_single_pool_element` -> `IASinglePoolElement`

## Structure Types: How They Work

### `JigsawStructure` (base)

Adds many controls on top of vanilla jigsaw structures:

- Optional Y allowance (`min_y_allowed`, `max_y_allowed`)
- Optional project-to-heightmap
- Liquid spawn prevention (`cannot_spawn_in_liquid`)
- Terrain variation checks (`terrain_height_radius_check`, `allowed_terrain_height_range`)
- Biome radius checks (`valid_biome_radius_check`)
- Optional max distance from center
- Rotation mode (`rotation_fixed` vs random). You want rotation fixed on when dealing with complex create contraptions or rotation sensitie blocks such as quark wooden poles.
- Optional burying modes (`LOWEST_CORNER`, `AVERAGE_LAND`, `LOWEST_SIDE`)
- Optional enhanced terrain adaptation kernel
- Optional liquid settings override

Generation is delegated to `PieceLimitedJigsawManager`, which supports per-piece limits/conditions.

### `OptionalDependencyStructure`

Same base behavior as `JigsawStructure`, but adds:

- `required_mods` (comma-separated)
- `illegal_mods` (comma-separated)

If checks fail, structure does not generate.

### `ModAdaptiveStructure`

Adds runtime pool switching:

- `change_pool_mods` (comma-separated required loaded mods)
- `new_pool`

If all listed mods are loaded, uses `new_pool`; otherwise uses `start_pool`.

### `NetherJigsawStructure`

Nether-oriented variant:

- Searches for highest/lowest usable land around structure bounds.
- Supports `ledge_offset_y`.
- `land_search_direction`: `HIGHEST_LAND` or `LOWEST_LAND`.

### `OverLavaNetherStructure`

Extra spawn safety checks for over-lava contexts:

- Verifies sampled area where piece would sit is mostly air/fluid-compatible.

### `BiomeFacingStructure`

Rotates structure orientation toward target biomes:

- `target_biome_radius_check_blocks`
- `target_biomes`

Uses custom biome sampling sectors to choose one of:

- `NONE`
- `CLOCKWISE_90`
- `COUNTERCLOCKWISE_90`
- `CLOCKWISE_180`

## Integrated Jigsaw Pool Element Extensions

`IASinglePoolElement` extends single pool element with:

- `name`
- `max_count`
- `min_required_depth` / `max_possible_depth` (deprecated in favor of conditions)
- `is_priority`
- `ignore_bounds`
- `condition` (custom condition codec)
- `enhanced_terrain_adaptation` (piece-level override)
- `deadend_pool` fallback support
- `modifiers` list (`StructureModifier`)
- `override_liquid_settings`

## Conditions, Actions, Target Selectors

### Conditions (`StructureConditionType`)

- `always_true`
- `any_of`
- `all_of`
- `not`
- `altitude`
- `depth`
- `random_chance`
- `piece_in_range`
- `mod_loaded`
- `piece_in_horizontal_direction`
- `rotation`

### Actions (`StructureActionType`)

- `transform` -> replace target piece with one from output template list (+ optional offsets).
- `delay_generation` -> mark target piece for delayed generation.

### Target Selectors (`StructureTargetSelectorType`)

- `self` -> target currently processed piece.

## Structure Block and Jigsaw Changes

- Structure block size limits increased from vanilla `48`/`80` behavior to `NEW_STRUCTURE_SIZE=512`.
- Structure block filename max length increased to `256`.
- Structure block render distance updated (`NEW_STRUCTURE_SIZE / 2`).
- Packet read/write for structure block settings changed to full ints for larger values.
- Corner detection optimized by searching outward from center (`getRelatedCorners` overwrite).
- Jigsaw max generation distance increased to `512`.
- Structure pool element weight cap raised to `5000` (Fabric + NeoForge mixins).

## Tags and What They Do

### Structure Tags

- `integrated_api:larger_locate_search`
  - `/locate structure` uses larger radius (`2000`) if queried set includes this tag.
  - Geode placement is blocked inside structure starts matching this tag.
- `integrated_api:unskippable_structures`
  - Overlap-skip system ignores these structures (they are never skipped for overlap).
- `integrated_api:disabled_structures`
  - Structure generation is cancelled for tagged structures.
  - `/locate structure` throws an error if query includes tagged structures.

### Feature Tags

- `integrated_api:skippable_features`
  - Features in this tag are skipped when overlapping recorded structure Y-ranges in chunk.

### Biome Collection Tags

Biome helper tags live under:

- `data/integrated_api/tags/worldgen/biome/collections/`

Included collections:

- `any_forests`, `any_taiga`, `badlands`, `bamboos`, `beaches`, `birch_forests`, `caves`, `dark_oak_forests`, `deep_oceans`, `deserts`, `end_island_land`, `end_voids`, `floral`, `flower_forests`, `giant_taigas`, `giant_trees`, `hills`, `icy`, `jungles`, `mangroves`, `meadows`, `mountains`, `mushrooms`, `nether`, `nether_basalts`, `nether_crimson_forests`, `nether_souls`, `nether_warped_forests`, `nether_wastes`, `oceans`, `orange_deserts`, `overworld`, `plains`, `regular_forests`, `rivers`, `rocky_mountains`, `savannas`, `snowy_forests`, `snowy_mountains`, `snowy_plains`, `spooky_forests`, `stony_shores`, `swamps`, `taigas`, `vanilla_deserts`, `vanilla_jungles`, `warm_beaches`, `windswept_deserts`.

Most of these are compatibility wrappers over vanilla/Forge/common-tag namespaces.

## Datapack-Driven Systems

Registered reload listeners:

- `integrated_structure_spawners`
- `integrated_structure_map_trades`
- `integrated_pieces_spawn_counts`
- `integrated_workstations`

### 1) Structure Spawners (`integrated_structure_spawners`)

Expected JSON key:

- `mobs`: array of `{ "name": "modid:entity", "weight": number, "optional": boolean? }`

Used by `spawner_randomizing_processor`.

### 2) Structure Map Trades (`integrated_structure_map_trades`)

Expected JSON keys:

- `villagerMaps`: map of profession id -> trade list
- `wanderingTraderMap`: map of trade type (`rare`/`common`) -> trade list

Each trade entry includes structure (id or `#tag`), icon, name, price, uses, xp, search radius.

### 3) Piece Spawn Counts (`integrated_pieces_spawn_counts`)

Expected JSON keys:

- `target_structure`
- `pieces_spawn_counts`: entries with:
  - `nbt_piece_name`
  - `always_spawn_this_many`
  - `never_spawn_more_than_this_many`
  - `minimum_distance_from_center_piece`
  - `condition` (custom json condition registry key)

Used by `PieceLimitedJigsawManager` to enforce required/min/max counts.

### 4) Workstations (`integrated_workstations`)

Expected JSON key:

- `workstations`: entries with:
  - `required_mod` (default `minecraft`)
  - `output_block`
  - `weight`

Used by `workstation_processor`.

## Placement Types

### Feature Placement Modifiers

- `minus_eight_placement`: subtracts 8 from X and Z.
- `unlimited_count`: repeating placement without vanilla capped codec range.
- `snap_to_lower_non_air_placement`: drops Y until non-air.
- `min_distance_from_world_origin_placement`: filters out positions too close to 0,0.

### Structure Placements

- `advanced_random_spread`: extended random spread with:
  - optional `min_distance_from_world_origin`
  - optional `super_exclusion_zone` + optional allowed radius behavior.
- `stronghold`: ring-based placement with configurable first-ring distance and thickness.

## Optimizations and Improvements

- Async structure locating for merchant maps (`AsyncLocator` thread pool).
- Cached terrain height helper (`GeneralUtils.getCachedFreeHeight`).
- Octree-based piece overlap tests (`BoxOctree`) in jigsaw assembly.
- Structure corner detection optimization in structure blocks.
- Reduced non-actionable log spam (`BlockAttachedEntityMixin`).
- Larger locate radius and selective structure overlap avoidance.

## Other Behavior Changes

- Optional geode suppression in structures tagged with larger locate search.
- Optional structure overlap prevention and feature skipping system.
- Rail rotation MC-196102 fix on Fabric mixins.
- NeoForge biome modifiers included:
  - additions, removals, and temperature-bucket additions.
- NeoForge datagen helper to update legacy structure NBT.

## Cross-Loader Internal API Surface

- Event wrappers:
  - lifecycle (`Setup`, `ServerGoingToStart`, `ServerGoingToStop`, reload registration)
  - villager/wandering trade registration events
- `PlatformHooks` (`isModLoaded`, `isDevEnvironment`) with Fabric/NeoForge impls.
- Custom registry support (`IAConditionsRegistry` for json conditions).

## Minimal Usage Recommendations for Structure Mod Authors

1. Use `integrated_api:generic_structure` (or one of the specialized variants) for your structure type.
2. Use `integrated_api:integrated_api_single_pool_element` when you need max counts, conditions, dead-end pools, or modifiers.
3. Put compatibility and worldgen behavior in datapacks:
   - piece count constraints
   - spawner mob lists
   - workstation replacement pools
   - villager/wandering map trades
4. Use structure tags:
   - `disabled_structures` for hard disable
   - `unskippable_structures` to bypass overlap skipping
   - `larger_locate_search` for wider locate search
