# Super Factory Manager Script Reference

This is the working reference for writing Super Factory Manager scripts for our ATM10 factory.
It is based on upstream Super Factory Manager examples, template programs, and parser behavior from
the [TeamDman/SuperFactoryManager](https://github.com/TeamDman/SuperFactoryManager) repository.
The upstream source is reference-only for this repo and should not be vendored here.

The upstream README describes SFM as a Minecraft mod with a domain-specific language for moving
items, fluids, and other resource types. The latest upstream release seen during this review was
`v4.32.0`, released March 10, 2026.

## Reference Sources

- Super Factory Manager upstream: https://github.com/TeamDman/SuperFactoryManager
- ATM10 pack repository: https://github.com/AllTheMods/ATM-10

Use the ATM10 repository as the pack-specific source of truth for KubeJS overrides, recipes,
datapacks, mod customizations, and Hostile Neural Networks data models. For mob generation scripts,
check both the Hostile Neural Networks mod data and ATM10's `kubejs/data/hostilenetworks/data_models`
path before listing available prediction models or assuming item IDs. Do not vendor upstream SFM or
ATM10 pack source into this repo.

## Core Model

- A program is an optional `NAME` followed by one or more triggers.
- Keywords are case-insensitive, but this repo should prefer lowercase for normal scripts unless
  copying directly from templates.
- A trigger contains ordered statements. Statement order matters.
- `INPUT` gathers candidate resources for the current trigger execution.
- `OUTPUT` consumes the currently active inputs and inserts matching resources into destinations.
- There is no persistent item buffer between trigger runs.
- A trigger's input list is cleared after that trigger finishes.
- Use `FORGET` inside a trigger when later outputs should not see earlier inputs.

```sfm
name "descriptive program name"

every 20 ticks do
    input from input_chest
    output to output_chest
end
```

## Label Naming

Use labels that describe the machine or role in the factory. In project scripts, wrap every label
in double quotes and use Title Case capitalization so label names match the in-game label gun text
exactly. Avoid throwaway labels such as `"A"` or `"B"` except when copying official examples.

Good labels:

- `"Raw Ore Chest"`
- `"Fuel Drawer"`
- `"Ore Furnace"`
- `"Smelted Output"`
- `"Water Tank"`
- `"Phyto Gro Drawer"`
- `"Bee Product Input"`
- `"Centrifuge"`
- `"Honey Output"`

Always quote labels, even when the SFM language would allow unquoted labels:

```sfm
input from "To Be Smelted"
input redstone from "Redstone Buffer"
```

## Comments

Use brief comments for calls where the intent is not obvious from the statement itself.
Do not comment every simple `input` or `output`.

Comment these:

- Complex `WITH` / `WITHOUT` tag filters.
- `EXCEPT` filters that prevent important items from moving.
- Slot numbers, especially machine-specific input/output slots.
- Side choices for machines with directional automation.
- `FORGET` boundaries.
- `RETAIN` values that represent machine stock levels.
- Redstone conditions and pulse triggers.

```sfm
every 20 ticks do
    input from fuel_drawer
    output retain 1 coal to each ore_furnace bottom side
    -- Keep one fuel item stocked in each furnace fuel slot.
    forget

    -- Smelted outputs should not consume the fuel input above.

    input from ore_furnace bottom side
    output to smelted_output
end
```

## Triggers

Use `every 20 ticks do` as the normal starting point. This runs once per second.

```sfm
every 20 ticks do
    input from source_chest
    output to destination_chest
end
```

Other forms:

```sfm
every 5 seconds do end
every tick do end
every redstone pulse do end
every 20 global ticks do end
every 20 global plus 5 ticks do end
every 20g+5 ticks do end
```

Best practices:

- Prefer `20 ticks` or slower for item and fluid movement unless the process truly needs faster
  updates.
- Use `global` timing only when multiple managers must align to the world clock.
- Use redstone triggers for explicit on-demand movement.
- Avoid `every tick` for broad item scans. It is usually unnecessary and can increase lag.

## Input And Output

Basic item movement:

```sfm
every 20 ticks do
    input from raw_ore_chest
    output to processing_buffer
end
```

Limit extraction or insertion:

```sfm
input 16 iron_ore from raw_ore_chest
output 8 to each ore_furnace top side
```

Use `each` when the quantity applies per matching label or block:

```sfm
input 1 from each input_chest
output 1 to each machine
```

Without `each`, the limit is total across the matching label set.

## Retain

`retain` keeps stock behind on input, or prevents output beyond a destination stock level.

```sfm
input retain 5 stone from building_block_drawer
output to overflow_chest
```

```sfm
input from fuel_drawer
output retain 1 coal to each ore_furnace bottom side
```

Best practices:

- Prefer `retain` over round robin when the real goal is "keep each machine stocked".
- Use `retain` on source drawers when an item must stay available for other systems.
- Document non-obvious numbers.

## Forget Boundaries

Inputs remain active until the trigger ends or `FORGET` removes them.

```sfm
every 20 ticks do
    input from raw_ore_chest
    output to ore_furnace top side
    forget

    input from fuel_drawer
    output retain 1 coal to ore_furnace bottom side
end
```

Use:

- `forget` to clear all active inputs.
- `forget label_name` to clear only inputs that came from a specific label.
- `forget label_one, label_two` to clear a subset.

Best practice: put `FORGET` between unrelated movement phases in the same trigger. Format it with
no blank line above and one blank line below.

## Filters

Resources can be listed inline or across multiple lines.

```sfm
input
    16 iron_ore,
    16 gold_ore,
    copper_ore,
from raw_ore_chest

output to ore_processing_input
```

Use `EXCEPT` for statement-wide exclusions:

```sfm
input *ingot* except iron_ingot, gold_ingot from ingot_drawer
output to export_chest
```

Pattern notes:

- Unquoted `*` is convenient and expands like a simple wildcard.
- Quoted patterns use full regex-style matching, so use `.*` when matching arbitrary text.

```sfm
input *ingot* from storage
input ".*:(iron|copper)_ingot" from storage
```

Best practices:

- Prefer explicit item names when the list is short and stable.
- Use wildcard filters for broad categories, but comment why the category is safe.
- Pair broad wildcards with `EXCEPT` when valuable items should not move.

## Tags

Use `WITH TAG` / `WITH #tag` for tag-based matching.

```sfm
input with #forge:ingots from storage
input minecraft:* with tag minecraft:mineable/shovel from storage
```

Tag logic supports `not`, `and`, `or`, and parentheses:

```sfm
input
    * with #ores/* and not #needs_stone_tool
from mining_output
```

`WITHOUT` negates the whole tag expression:

```sfm
input without #needs_stone_tool and #ores/* from mining_output
```

Best practices:

- Comment complex tag expressions.
- Remember that `EXCEPT` is statement-wide and does not support `WITH` clauses.
- Prefer parentheses when mixing `not`, `and`, and `or`.

## Slots And Sides

Use sides for machines that expose different automation faces:

```sfm
input from raw_ore_chest
output to ore_furnace top side

forget

input from fuel_drawer
output to ore_furnace bottom side
```

Use slots for machine-specific layouts:

```sfm
output retain 2 silicon to each silicon_inscriber slots 2
input from processor_inscriber west side slots 0-2
```

Supported side forms include:

- Absolute sides: `top`, `bottom`, `north`, `east`, `south`, `west`
- Relative sides: `left`, `right`, `front`, `back`
- `null side`
- `each side`
- Multiple sides: `top, west side`

Best practices:

- Comment slot numbers with the machine's slot meaning.
- Prefer explicit sides for Mekanism and other side-sensitive machines.
- Use the inspector/network tool in-game when a modded machine's automation slots differ from GUI
  slot numbers.

## Fluids, Energy, And Resource Types

Items are the default resource type:

```sfm
input stone from storage
```

Fluids:

```sfm
input fluid::water from water_tank
output fluid::water to machine
```

All fluids:

```sfm
input fluid:: from tank
output fluid:: to destination_tank
```

Forge energy aliases may include `fe::`, `rf::`, `energy::`, and `power::` depending on version:

```sfm
input fe:: from energy_cell
output fe:: to machine
```

Best practices:

- Match resource types between input and output. Do not input `fluid::` and output plain item
  resources.
- Keep energy triggers focused and simple.
- Specify sides for machines whose energy, fluid, gas, or item handlers differ by side.

## Conditions

Conditions support inventory checks, redstone checks, boolean logic, and `else if`.

```sfm
every 20 ticks do
    if overall raw_ore_chest has >= 32 iron_ore then
        input 32 iron_ore from raw_ore_chest
        output to ore_processing_input
    end
end
```

Set operators for labels with multiple blocks:

- `overall`: count across all blocks with the label.
- `some`: at least one block matches.
- `every` / `each`: all blocks match.
- `one`: exactly one block matches.
- `lone`: zero or one block matches.

Best practices:

- Use `overall` explicitly when you mean total stock across a label group.
- Use parentheses when mixing `not`, `and`, and `or`.
- Keep condition blocks short. Use `FORGET` inside them when needed.

## Redstone

Pulse trigger:

```sfm
every redstone pulse do
    input from request_buffer
    output to machine_input
end
```

Signal strength check on the manager:

```sfm
every 20 ticks do
    if redstone >= 2 then
        input from gated_input
        output to gated_output
    end
end
```

Best practices:

- Use redstone for manual gates, batch start buttons, or safety interlocks.
- Comment the meaning of signal thresholds.

## Empty Slots

Use `empty slots in` to insert only into empty destination slots.

```sfm
every 20 ticks do
    input from mob_loot
    output to empty slots in storage
end
```

Optional condensing pass:

```sfm
every 300 seconds do
    input from storage
    output to storage
end
```

This can help with large inventories by avoiding repeated attempts to merge into full slots.

## Round Robin

Round robin exists, but prefer `retain` when the goal is keeping machines stocked.

```sfm
input from source_chest
output 128 dirt to destination ROUND ROBIN BY BLOCK
```

```sfm
input from "storage one", "storage two" ROUND ROBIN BY LABEL
output 8 bone_meal to each dispenser
```

Best practices:

- Use round robin only when alternating targets or sources is the actual goal.
- Add a short comment explaining why `retain` is not the better fit.

## Performance Practices

- Start with `every 20 ticks do`; make it faster only after testing.
- Split unrelated tasks into separate triggers when it makes behavior clearer.
- Use `FORGET` to keep active input sets small and correct.
- Avoid broad wildcard scans against huge storage systems unless needed.
- Prefer explicit resource filters for hot paths that run often.
- Use `empty slots in` for large, mostly-full inventories.
- Use the in-game performance graph to compare real programs in the actual factory.
- Avoid many managers doing the same broad storage scan every tick.

## Script Review Checklist

Before using a script in the factory:

- Labels are descriptive, double quoted, Title Case, and match the in-game label gun labels.
- No placeholder labels such as `"A"` or `"B"`.
- Complex filters, slot numbers, side choices, and retain values have brief comments.
- Every `INPUT` has a matching `OUTPUT` that uses the same resource type.
- Unrelated movement phases are separated with `FORGET`.
- `FORGET` statements have no blank line above and one blank line below.
- Sides are specified for side-sensitive machines.
- Conditions use explicit `overall`, `some`, `each`, `one`, or `lone` where ambiguity would matter.
- Broad wildcard or tag filters are intentional and safe.
- Trigger intervals are no faster than needed.
- The script was tested with small quantities before connecting high-value inventories.

## Common Patterns

Furnace array:

```sfm
name "ore furnace array"

every 20 ticks do
    input from raw_ore_chest
    output 8 to each ore_furnace top side
    forget

    input from fuel_drawer
    output retain 1 coal to each ore_furnace bottom side
    -- Keep one coal stocked in each furnace fuel slot.
    forget

    input from ore_furnace bottom side
    output to smelted_output
end
```

Fluid fill:

```sfm
name "water supply"

every 20 ticks do
    input fluid::water from water_tank
    output retain 1000 fluid::water to each machine
    -- Keep one bucket of water buffered in each machine.
end
```

Conditional batch move:

```sfm
name "batch ore export"

every 20 ticks do
    if overall raw_ore_chest has >= 64 iron_ore then
        input 64 iron_ore from raw_ore_chest
        output to ore_processing_input
    end
end
```
