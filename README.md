# Minecraft SFM Scripts

This repository contains local
[Super Factory Manager](https://github.com/TeamDman/SuperFactoryManager) scripts
for automating an ATM10 Minecraft factory.

Super Factory Manager uses a small domain-specific language to move items,
fluids, energy, and other resources between labelled blocks in-game. The scripts
in this repo are meant to be copied into SFM managers and matched with label gun
labels in the world.

## Layout

- `scripts/` contains the factory programs.
- `SFM_REFERENCE.md` is the local reference for syntax, formatting, examples,
  and review checks.
- `AGENTS.md` records repository-specific contributor instructions.

## Reference Sources

- Super Factory Manager upstream:
  https://github.com/TeamDman/SuperFactoryManager
- ATM10 pack data and customizations:
  https://github.com/AllTheMods/ATM-10

Use the ATM10 repository when checking pack-specific item IDs, KubeJS changes,
recipes, Hostile Neural Networks data models, and available prediction outputs.
These upstream repositories are reference-only; do not vendor their source into
this repo.

## Getting Started

1. Read `SFM_REFERENCE.md` for the supported syntax and local style rules.
2. Pick a script from `scripts/` that matches the factory system you want to
   automate.
3. In Minecraft, place a Super Factory Manager block near the machines,
   inventories, tanks, or cables it needs to control.
4. Use the label gun to apply labels that exactly match the quoted labels in
   the script, such as `"Battery"`, `"Furnace"`, or `"Drawer Controller"`.
5. Copy the script into the Super Factory Manager program editor.
6. Save the program, then verify movement one phase at a time: items, fluids,
   and FE should move only between the labelled blocks named in the script.

When creating a new automation, start with `every 20 ticks do` for normal item
or fluid movement. Use a separate `every tick do` trigger for FE movement, and
separate unrelated movement phases with `forget`.

## Conventions

Use `SFM_REFERENCE.md` as the source of truth before changing scripts. In
general, scripts should use lowercase SFML keywords, four-space indentation,
quoted Title Case labels, and `every 20 ticks do` for normal item or fluid
movement. Energy movement should usually run in its own focused `every tick do`
trigger.

Upstream Super Factory Manager and ATM10 pack sources are reference-only for
this repo. Do not vendor their source here.
