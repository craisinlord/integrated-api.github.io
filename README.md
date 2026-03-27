# Integrated API (Minecraft 1.21.1)

Integrated API is a structure-focused library mod for Fabric and NeoForge.

This README is split into exactly two sections:
1. Datapack Guide (no Java required)
2. Backend Worldgen Reference (advanced)

---

## 1) Datapack Guide (No Java Required)

If you only make datapacks and structure JSON files, use this section.

### What You Can Do Without Writing Java

- Disable specific structures globally.
- Prevent specific structures from being skipped by overlap checks.
- Increase `/locate structure` search radius for specific structures.
- Skip selected worldgen features inside generated structure space.
- Define structure spawner mob pools.
- Define villager and wandering trader structure map trades.
- Define per-piece spawn caps/requirements for jigsaw pieces.
- Define workstation replacement pools used by the workstation processor.
- Reuse included biome collection tags for cleaner biome targeting.

### Tags You Should Know

Path root: `data/integrated_api/tags/worldgen/`

#### Structure tags

- `structure/disabled_structures.json`
  - Tagged structures do not generate.
  - `/locate structure` is blocked for those structures.

- `structure/unskippable_structures.json`
  - Overlap prevention will not skip these structures.

- `structure/larger_locate_search.json`
  - `/locate structure` uses a larger search radius for these structures.
  - Geodes are prevented from generating inside these structure starts.

#### Feature tags

- `feature/skippable_features.json`
  - Tagged features are skipped when they overlap recorded structure Y-ranges.

### Tag Example

```json
{
  "replace": false,
  "values": [
    "minecraft:desert_pyramid",
    "yourmod:my_large_temple"
  ]
}
```

### Datapack Reload Systems (JSON-Driven)

Integrated API reads these data-driven systems on reload:

- `integrated_structure_spawners`
- `integrated_structure_map_trades`
- `integrated_pieces_spawn_counts`
- `integrated_workstations`

Use these names as your mental model for what each dataset controls.

#### Example: `integrated_structure_map_trades`

```json
{
  "villagerMaps" : {
    "minecraft:cartographer": [
      {
        "tradeLevel": 2,
        "structure" : "idas:labyrinth",
        "mapName": "Desert Labyrinth Map",
        "mapIcon": "BANNER_YELLOW",
        "emeraldsRequired": 20,
        "tradesAllowed": 2,
        "xpReward": 10,
        "spawnRegionSearchRadius": 100
      },
      {
        "tradeLevel": 2,
        "structure": "idas:pillager_fortress",
        "mapName": "Pillager Fortress Map",
        "mapIcon": "MANSION",
        "emeraldsRequired": 20,
        "tradesAllowed": 2,
        "xpReward": 10,
        "spawnRegionSearchRadius": 100
      }
    ]
  },
  "wanderingTraderMap" : {
    "rare": [],
    "common": [
      {
        "structure": "idas:treetop_tavern",
        "mapName": "Treetop Tavern Map",
        "mapIcon": "BANNER_ORANGE",
        "emeraldsRequired": 15,
        "tradesAllowed": 2,
        "xpReward": 100,
        "spawnRegionSearchRadius": 100
      }
    ]
  }
}
```

#### Example: `integrated_structure_spawners`

```json
{
  "mobs": [
    {
      "name": "minecraft:skeleton",
      "weight": 10
    },
    {
      "name": "quark:forgotten",
      "weight": 10
    }
  ]
}
```

#### Example: `integrated_pieces_spawn_counts`

```json
{
  "target_structure": "idas:castle",
  "pieces_spawn_counts": [
    {
      "nbt_piece_name": "idas:castle/start",
      "always_spawn_this_many": 1,
      "never_spawn_more_than_this_many": 1,
      "minimum_distance_from_center_piece": 0
    },
    {
      "nbt_piece_name": "idas:castle/tower",
      "never_spawn_more_than_this_many": 3,
      "condition": "integrated_api:always_true"
    }
  ]
}
```

#### Example: `integrated_workstations`

```json
{
  "workstations": [
    {
      "required_mod": "minecraft",
      "output_block": "minecraft:beehive",
      "weight": 1
    },
    {
      "required_mod": "new_villagers",
      "output_block": "new_villagers:bee_station",
      "weight": 1
    }
  ]
}
```

### Practical Datapack Workflow

1. Put structures you want disabled into `disabled_structures`.
2. Put structures that must always generate (even if overlapping) into `unskippable_structures`.
3. Put rare/far structures into `larger_locate_search` for better `/locate` behavior.
4. Put intrusive features into `skippable_features` if they should not carve through structures.
5. Add optional JSON datasets for map trades, spawners, piece caps, and workstation replacement.

### Biome Helper Tags Included

Biome helper tags are provided under:
`data/integrated_api/tags/worldgen/biome/collections/`

Included collections:

- `any_forests`, `any_taiga`, `badlands`, `bamboos`, `beaches`, `birch_forests`, `caves`, `dark_oak_forests`, `deep_oceans`, `deserts`, `end_island_land`, `end_voids`, `floral`, `flower_forests`, `giant_taigas`, `giant_trees`, `hills`, `icy`, `jungles`, `mangroves`, `meadows`, `mountains`, `mushrooms`, `nether`, `nether_basalts`, `nether_crimson_forests`, `nether_souls`, `nether_warped_forests`, `nether_wastes`, `oceans`, `orange_deserts`, `overworld`, `plains`, `regular_forests`, `rivers`, `rocky_mountains`, `savannas`, `snowy_forests`, `snowy_mountains`, `snowy_plains`, `spooky_forests`, `stony_shores`, `swamps`, `taigas`, `vanilla_deserts`, `vanilla_jungles`, `warm_beaches`, `windswept_deserts`.

Use these tags in your structure biome selectors to reduce manual biome lists.

---

## 2) Backend Worldgen Reference (Advanced)

If you are extending Integrated API in code or building a mod API on top of it, use this section.

### Platform and Scope

- Minecraft: `1.21.1`
- Loaders: Fabric + NeoForge
- Shared logic is in `common/` with loader-specific adapters/mixins in `fabric/` and `neoforge/`.

### Core Registry Types

#### Structure types (`BuiltInRegistries.STRUCTURE_TYPE`)

- `integrated_api:generic_structure` -> `JigsawStructure`
- `integrated_api:optional_dependency_structure` -> `OptionalDependencyStructure`
- `integrated_api:mod_adaptive_structure` -> `ModAdaptiveStructure`
- `integrated_api:nether_structure` -> `NetherJigsawStructure`
- `integrated_api:over_lava_nether_structure` -> `OverLavaNetherStructure`
- `integrated_api:biome_facing_structure` -> `BiomeFacingStructure`

#### Structure placements (`BuiltInRegistries.STRUCTURE_PLACEMENT`)

- `integrated_api:advanced_random_spread` -> `AdvancedRandomSpread`
- `integrated_api:stronghold` -> `StrongholdPlacement`

#### Feature placement modifiers (`BuiltInRegistries.PLACEMENT_MODIFIER_TYPE`)

- `integrated_api:minus_eight_placement`
- `integrated_api:unlimited_count`
- `integrated_api:snap_to_lower_non_air_placement`
- `integrated_api:min_distance_from_world_origin_placement`

#### Processors (`BuiltInRegistries.STRUCTURE_PROCESSOR`)

- `block_removal_post_processor`
- `flood_with_water_processor`
- `replace_air_only_processor`
- `replace_liquids_only_processor`
- `spawner_randomizing_processor`
- `fill_end_portal_frame_processor`
- `remove_floating_blocks_processor`
- `close_off_fluid_sources_processor`
- `close_off_air_sources_processor`
- `random_replace_with_properties_processor`
- `waterlogging_fix_processor`
- `waterlogging_when_replacing_water_processor`
- `capped_structure_surface_processor`
- `post_process_list_processor`
- `windmill_bearing_processor`
- `fluid_tank_processor`
- `mechanical_bearing_processor`
- `clockwork_bearing_processor`
- `elevator_pulley_processor`
- `tick_blocks_processor`
- `integrated_block_replace_processor`
- `workstation_processor`

#### Rule tests / pos rule tests

- `integrated_api:matter_phase_rule_test`
- `integrated_api:piece_origin_axis_aligned_linear_pos_rule_test`
- `integrated_api:y_value_pos_rule_test`

#### Pool element type

- `integrated_api:integrated_api_single_pool_element` -> `IASinglePoolElement`

### Structure Family Behavior

#### `JigsawStructure`

Adds optional controls on top of vanilla jigsaw behavior:

- Y range allowance
- heightmap projection
- liquid-spawn prevention
- terrain height variance checks
- biome radius checks
- center-distance checks
- fixed/random rotation
- burying modes (`LOWEST_CORNER`, `AVERAGE_LAND`, `LOWEST_SIDE`)
- enhanced terrain adaptation and liquid setting overrides

Generation is delegated through `PieceLimitedJigsawManager` with piece-count and condition support.

#### `OptionalDependencyStructure`

- Enables/disables generation based on `required_mods` and `illegal_mods`.

#### `ModAdaptiveStructure`

- Swaps to `new_pool` when all `change_pool_mods` are loaded.

#### `NetherJigsawStructure`

- Adds nether-focused land search and `ledge_offset_y`.

#### `OverLavaNetherStructure`

- Adds over-lava viability checks before generation.

#### `BiomeFacingStructure`

- Chooses rotation toward configured target biome groups.
- It can be used to make a structure face a biome target, like a pirate dock spawning on a beach facing the ocean.

### Integrated Pool Element Extensions (`IASinglePoolElement`)

- `name`
- `max_count`
- `min_required_depth` / `max_possible_depth`
- `is_priority`
- `ignore_bounds`
- `condition`
- `enhanced_terrain_adaptation`
- `deadend_pool`
- `modifiers`
- `override_liquid_settings`

### Condition / Action / TargetSelector System

#### Condition types

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

#### Action types

- `transform`
- `delay_generation`

#### Target selectors

- `self`

### Structure Block and Jigsaw Runtime Changes

- Structure block size limit raised to `512`.
- Structure block name length raised to `256`.
- Structure block render distance increased.
- Structure block packet handling supports larger ints.
- Structure block corner detection optimized.
- Jigsaw max generation distance raised to `512`.
- Pool element weight cap raised to `5000` (Fabric + NeoForge mixins).

### Overlap and Disable Pipeline

- Existing structure bounds are recorded per-dimension/per-chunk.
- Overlap checks are used to decide whether to skip new structures.
- `unskippable_structures` tag bypasses skip behavior.
- `disabled_structures` tag hard-cancels generation.
- Locate command mixin blocks locate on disabled structures.

### Datapack-Driven Runtime Managers

- `MobSpawnerManager` (`integrated_structure_spawners`)
- `StructureMapManager` (`integrated_structure_map_trades`)
- `StructurePieceCountsManager` (`integrated_pieces_spawn_counts`)
- `WorkstationManager` (`integrated_workstations`)

### Performance and QoL Improvements

- Async map locating (`AsyncLocator`).
- Height lookup caching (`GeneralUtils.getCachedFreeHeight`).
- Octree overlap checks (`BoxOctree`).
- Reduced noisy `BlockAttachedEntity` logging.
- Optional geode suppression and feature skipping in tagged structure contexts.

### Loader-Specific Notes

- Fabric:
  - rail rotate bug fix (MC-196102 family)
  - registry bootstrap mixin support
  - structure pool weight cap mixin
- NeoForge:
  - biome modifier serializers and implementations
  - structure NBT updater datagen helper
  - structure pool weight cap mixin
