# food_ideas

A personal recipe database. Send Claude a link (Instagram reel, YouTube, a blog,
pasted text) and it extracts the recipe and files it in `recipes/`.

Then ask it things like:

- "What can I make with paneer?"
- "High protein, something with potato."
- "I've had pasta three times this week — something different?"
- "Something light tonight, I have 30 minutes."

Every recipe is tagged from a fixed vocabulary — `high-protein`, `light`,
`filling`, `low-carb`, `high-fiber`, `carb-heavy`, `quick`, `comfort` — so those
requirements are a single lookup rather than a judgement call each time.

## Layout

| Path | What it is |
|---|---|
| [`INDEX.md`](INDEX.md) | Every recipe at a glance |
| `recipes/` | One markdown file per recipe |
| [`CLAUDE.md`](CLAUDE.md) | Instructions Claude follows when adding and suggesting |
| `.claude/skills/suggest-recipe/` | The procedure for answering "what should I cook?" |
| `MADE_LOG.md` | What was actually cooked, and when — *untracked* |
| `PREFERENCES.md` | Diet, household size, habits — *untracked* |

The last two are personal and stay out of git. The recipes are the shareable part.

Recipes are plain markdown with YAML frontmatter, so they stay readable and
greppable without any tooling.
