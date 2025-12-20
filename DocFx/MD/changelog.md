# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

## [1.3.0] - 2025-12-16
### Added
#### Runtime Features
- Added `PRV[N]` and `AT[N]` keywords for growth formulas' expressions. 
  - `PRV[N]`: Represents the n-previous Growth Formula value. For example, `PRV[3]` at level 8 would return the Growth Formula value at level 5.
  - `AT[N]`: Represents the Growth Formula value at a specific level N. For example, `AT[10]` would return the Growth Formula value at level 10.
- `BoundedValue.Clamp` now supports also clamping of double values. For now is used internally by the framework and is reserved for future use is Astra RPG Health.

#### Editor Features
- Added support for `enum` formatting in `InspectorTypography`.
- Added utility to provide compact representation of large numbers in the editor (e.g., 1.5K, 4M). Reserved for future use in Astra RPG Health.

### Changed
#### Runtime Features
- Changed the internal `AttributePointsTracker` used by `EntityAttributes` to track spendable attribute points. `EntityPointsTracker` replaces the legacy tracker, and relies on a new `AttributePortfolio` to manage the allocation of attribute points to attributes. This new system is more readable, more robust, and more maintainable. The old system is left for backward compatibility. By updating the framework to this version, existing projects will automatically migrate to the new system.

### Fixed
#### Runtime Features
- Fixed Round Robin Strategy for attribute points removal on level down or reset to level 1.

## [1.2.0] - 2025-11-25
### Added
#### Runtime Features
- Added `OnLevelDownEvent` Game Event. This event is triggered whenever an entity levels down, allowing you to respond to level down events in your game.
- Added XP deduction feature. Now you can deduct XP from an entity using the `EntityCore.Level.RemoveExp(long amount)` method. This will also handle level downs if the deducted XP causes the entity to drop below the current level's XP threshold. Triggers the `OnLevelDownEvent` once for each level the entity drops.
- Added reset to level 1 feature. You can now reset an entity's level and XP to level 1 using the `EntityCore.Level.ResetToLevelOne()` method. This will set the entity's level to 1 and XP to 0. Triggers the `OnLevelDownEvent` once for each level the entity drops.
- Added basic support for statistics, attributes, and classes names presentation. This includes:
  - `IDisplaySONameProvider<in T> where T : ScriptableObject` interface for providing custom display names for ScriptableObjects.
  - `DefaultDisplaySONameProvider<T>` class that implements the above interface and returns the ScriptableObject's name as the display name.
  - `DefaultAttributeCompactDisplaySONameProvider` class that provides compact display names for attributes (e.g., "Str" for "Strength"). This one simply takes the first three letters of the attribute's name, capitalize them, and return that as the display name.  
  You can implement your own display name providers by implementing the `IDisplaySONameProvider<T>` interface to provide custom display names for your statistics, attributes, and classes or to use **localization systems**.

#### Editor Features
- Added *Read Only* (`RO`) property in InspectorTypography. Marks a field in the inspector as read only, in the sense that the component automatically manages it, and the user should not manually edit any associate LongVar/IntVar associated values. Currently reserved for future use.
- Added `IValidatable` interface. Used by the framework to propagate validation events to depending objects. Used to ensure that when a MonoBehaviour/ScriptableObject is validated in the inspector, all depending objects are also validated to ensure data consistency.
- Added `TypeSelectionMenu`. Provides a reusable editor context menu to pick a concrete type derived from a given base type.
Configurable display (optional "None" or "Default" entries, optional name trimming, exclusion of test types, extra filters).
Returns the chosen type via a callback so callers can create or clear instances accordingly.
- Added `[TypeSelectable]` attribute for use on `[SerializeReference]` fields; supports an optional DefaultType to present a "Default" choice. Shows also "None" to clear the reference.
- Added `SerializeReferenceDrawer` custom property drawer. New editor PropertyDrawer showing current concrete type and a Select button for managed references.
Opens centralized `TypeSelectionMenu`, supports choosing None, Default or concrete types and creates/assigns instances.
- Added `SerializeReferenceInitializer`. Runs in the editor to ensure `[SerializeReference]` fields marked with `[TypeSelectable]` that declare a default type are populated when scenes/prefabs are opened or on demand (during script compilation).
Automatically instantiates and assigns default instances where fields are null, provides a menu command to initialize all applicable fields, and marks modified assets/scenes as dirty.

#### Samples
- Added `ExpUtils` MonoBehaviour. Adds utility methods that can be invoked from the Unity Editor (in play mode) to add or remove XP from an entity for testing purposes.

### Changed
#### Runtime
- `SoSetScalingComponentBase` now logs a warning if no entity set is assigned during the lookup of values to use for scaling calculations.
- `AttributeScalingComponent` and `StatScalingComponent` now return `0` as the scaled value if no entity set is assigned, instead of rasing an error. This behavior allows for more flexible setups with entities with different sets of attributes/stats. The warning logged by the base class will help identify potential misconfigurations.
- Marked `OnEntityLevelUp` and `OnEntityLevelDown` events in `EntityCore` as required (red *).
Such methods are becoming crucial for the correct functioning of the various components of the framework. Therefore, it's important to ensure that these events are properly assigned.  
Entities that are missing these events will raise an error during their `Start()` method.

#### Editor
- Improved hot-reloading support for StatSet and AttributeSet.
- Attributes of `AttributeSet` are not "Re-play" (yellow R) any more. This means that it supports changes (addition/removal) of attributes in the AttributeSet without exiting play-mode.
- Improved `InspectorTypography` class. Now it supports also property drawn within GUI rects.

#### Samples
- `ValuesReader` and `ValueEntry` now use `[SerializeReference]` and `[TypeSelectable]` attributes for allowing easier selection of the presentation strategy for each value entry (`IDisplaySONameProvider` implementation).

### Deprecated
- Deprecated `DerivedTypePicker` in favor of the new `TypeSelectionMenu` system. Use the new system for better type selection in the editor. `DerivedTypePicker` will be removed in future releases.

### Fixed
- Added missing invalidation of the cached values for the statistics in `EntityStats.OnEnable()` method.
- Fixed `SetTotalCurrentExp` that was not raising level up and level down events when appropriate.
- Fixed the assignment of total available attribute points. Previously, this was handled only in OnValidate, which caused issues in builds (where editor-only code is stripped). Initialization is now also performed in `EntityCore.Start()`. Additionally, total available attribute points are now updated within `AddPoints()`, ensuring they are correctly recalculated whenever points are added—not only during `Init()`.

## [1.1.2] - 2025-11-12
### Changed
- Updated the "Example scene and instances" samples. Now by default the Hero prefab uses a IntVar for the attribute spendable points per level. This makes more consistent the amount of spendable attribute points acquired by each character on level up
- Updated offline docs with Installation Guide section

## [1.1.1] - 2025-11-06
### Changed
- Updated the links in `package.json` to match the new docs URL

## [1.1.0] - 2025-11-06
### Changed
#### Updated branding from "SOAP RPG Framework" to "Astra RPG Framework"
IMPORTANT: Refer to the [migration guide](./migration-guide.md) for updating existing projects that used the previous version.
- UPM package name changed to "com.electricdrill.astra-rpg-framework".
- UPM package display name changed to "Astra RPG Framework".
- Namespaces updated to reflect new branding: "ElectricDrill.AstraRPGFramework".
- Menu paths in Unity Editor updated to "Astra RPG Framework".


### Deprecated
- `_includedStatSets` field in `StatSet` will become an internal property in future releases.

## [1.0.0] - 2025-10-30
### Added
- Initial release of SOAP RPG Framework.
- ScriptableObject-based toolkit for core RPG mechanics.
- MonoBehaviour-based components for easy integration.

#### ScriptableObject Features
- Statistics & Attributes: Define core stats like Strength and Intelligence, as well as derived attributes like Health or Mana.
- StatSets & AttributeSets: Group related stats and attributes together for organization. Sets can be nested for complex structures.
- Experience & Leveling System: Custom experience curves and character progression.
- Classes: Define character classes with unique progressions and abilities.
- Progression System: Control how stats and attributes grow as characters level up.
- Scaling Formulas: Custom formulas to calculate final values of stats and attributes.
- Scaling Components: Modular scaling formulas reusable across stats and attributes.
- Game Events: Design and trigger custom game events.
- Growth Formulas: Determine how values progress based on a character's level.
- Game Event Generators: Define custom game event types.
- Variables: Use Long and Int variables as ScriptableObjects for persistent data storage.

#### MonoBehaviour Features
- EntityCore: Manage a character's core data, including experience and level.
- EntityStats & EntityAttributes: Assign statistics and attributes to any entity. Components can retrieve values from the assigned EntityClass based on level.
- EntityClass: Assign a class to an entity, linking it to specific progression paths.

#### API & Modifiers
- Comprehensive API for advanced customization (see API Docs).
- Dynamic application of modifiers to stats and attributes:
  - Flat & Percentage Modifiers for attributes.
  - Flat, Percentage, & Stat-to-Stat Modifiers for statistics.

