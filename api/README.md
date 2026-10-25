# QuantaWorks API for mod developers

API version **1** · package `dev.cptgummiball.quantaworks.api` · mod id `quantaworks`

**Most mods need nothing from this page** if QuantaWorks uses the standard loader interfaces.

## Getting the API

`quantaworks-api-1.jar` and `quantaworks-api-1-javadoc.jar` from the [releases](../../../releases),
as a **compile-only** dependency. Do not ship the API or any QuantaWorks files in your mod. If
QuantaWorks is optional for your mod, check that the mod `quantaworks` is loaded before touching its
classes.

The license (section 5) allows compiling against the API and publishing mods that use it.

## What else is there

Everything QuantaWorks processes is also open to **data packs**: machine recipes, materials, processing templates, Foundry
alloys, crop rules, growth profiles, reconstruction rules, Deep Core ore tables, plant wall parts through block tags, and
the Assembler block list. The formats, the loader interfaces per version and the gas conventions are on the wiki page
[For Mod Developers](../../../wiki/For-Mod-Developers).

## Java API v1 (`quantaworks-api-1.jar`, Minecraft-free)

Compile against the API jar only (`compileOnly`, never shade it); QuantaWorks brings the classes at runtime. The API has
no Minecraft classes, so one jar serves every Minecraft version and both loaders.

```java
// in your mod initialiser (Fabric onInitialize, NeoForge mod constructor)
QuantaWorksApi.materials().family("example:exampleium")
    .form(MaterialForm.ORE, "#c:ores/exampleium").form(MaterialForm.INGOT, "#c:ingots/exampleium")
    .form(MaterialForm.DUST, "#c:dusts/exampleium").meltingPoint(1450).byproduct("example:shinium", 0.10).register();
QuantaWorksApi.recipes().pulverizing("example:crush_shard")
    .input("#c:gems/example_shard", 1).output("example:shard_dust", 2).energy(40).duration(120).register();
QuantaWorksApi.conditions().register("example:in_overworld", ctx -> ctx.dimension().equals("minecraft:overworld"));
QuantaWorksApi.listeners().add(new ProcessListener() { public void onFinish(ProcessEvent e) { /* … */ } });
```

| Part | Use |
|---|---|
| `QuantaWorksApi.recipes()` | `recipe(type, id)` and shortcuts (`pulverizing`, `inductionSmelting`, `pressing`, `rolling`, `gearForming`, `wireDrawing`, `compacting`, `plantProcessing`, `assembly`); builder: `input`, `inputAny`, `fluid`, `gas`, `output` (with chance), `fluidOutput`, `gasOutput`, `energy` (E/t), `duration`, `tier`, `priority`, `requirement`, `condition`, `register()` |
| `QuantaWorksApi.materials()` | `family("ns:material")` – forms (`MaterialForm` or the data key), `preferred`, `units`, `meltingPoint`, `color`, `byproduct` |
| `QuantaWorksApi.conditions()` | `register(id, ProcessCondition)`; a recipe naming it (`condition(id)` / JSON `"process_conditions"`) only starts while it holds. `ProcessContext`: machine, recipe, dimension, position, day time, rain. An unknown condition never holds |
| `QuantaWorksApi.listeners()` | `ProcessListener`: `onStart`, `onFinish`, `onFail(reason)`; `onFormed`/`onCollapsed` for plants  |
| `MachineQuery` lookup | id `quantaworks:machine_query` – Fabric `BlockApiLookup<MachineQuery, Direction>`, NeoForge sided `BlockCapability<MachineQuery, Direction>`; `info()` gives `MachineInfo` (machine id, energy, usage, progress, active, status, recipe) |

**Rules (checked):** registration only during mod initialisation – QuantaWorks freezes the API when the server starts,
later calls throw `IllegalStateException` ("frozen"). Ids are namespaced and unique (a second registration throws).
Amounts must be positive, chances in (0, 1], tiers 1–4; unknown forms throw. Nothing is silently adjusted. API recipes
are rebuilt with every data reload (tags resolve like in data packs); a data pack file with the same id wins; a recipe
whose output does not exist in the pack is skipped. Listeners and conditions run on the server thread; their exceptions
are logged and never stop a machine.

## Stability

`QuantaWorksApi.API_VERSION` changes with every incompatible change of `dev.cptgummiball.quantaworks.api`. Everything
else is internal.
