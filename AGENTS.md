# Repository Notes

## SFM Scripts

- Use `SFM_REFERENCE.md` as the local source of truth for SFML syntax, naming conventions, formatting, common patterns, and review checklist before creating or changing scripts.
- Treat upstream Super Factory Manager as reference-only. Do not vendor the mod source into this repo; consult https://github.com/TeamDman/SuperFactoryManager when implementation details, examples, or parser behavior need verification.
- Put local factory programs in `scripts/`.
- Prefer lowercase SFML keywords and four-space indentation.
- Use descriptive, quoted, Title Case labels so they match the in-game label gun text exactly.
- Keep unrelated movement phases separated with `forget`.
- Use `every 20 ticks do` as the default trigger interval unless a faster loop is known to be needed.
- Move FE/energy every tick in its own focused trigger unless there is a specific reason not to.
- Match resource types between `input` and `output`, for example `fluid::water` to `fluid::water` and `fe::` to `fe::`.
- Add sides only when the target machine needs side-specific automation.
