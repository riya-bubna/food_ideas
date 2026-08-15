# food_ideas

A personal recipe database you talk to. Send Claude a link — Instagram reel, YouTube
short, a blog, or just pasted text — and it extracts the recipe and files it as markdown
in `recipes/`. Then ask it what to cook.

There's no app, no database and no build step. Every recipe is a plain markdown file
with YAML frontmatter, so it stays readable, greppable and diffable with no tooling at
all. The intelligence is in the conventions, not in code.

---

## Start here

Open this directory in Claude Code and ask for a recipe suggestion:

```
> /suggest-recipe what should I cook tonight?
```

The **first** time you do this, Claude won't answer. It'll notice that `PREFERENCES.md`
doesn't exist yet and run a short interview instead — two rounds of questions covering:

- **Allergies and intolerances** — always asked, never inferred
- **Diet** — and what should happen to a recipe containing something you don't eat:
  substitute it, or skip the recipe entirely?
- **Household size** — how many people you cook for
- **Kitchen** — oven, air fryer, microwave, pressure cooker. What you *lack* matters
  more than what you have
- **Your everyday baseline** — what you already eat most weeks
- **Ingredients you don't buy**, and any strong dislikes

It writes your answers to `PREFERENCES.md` and then answers your original question. You
only do this once. Edit the file by hand any time — it's yours.

**`PREFERENCES.md` is deliberately untracked in git.** So is `MADE_LOG.md`. The recipes
are the shareable part; your diet isn't.

### If you cloned this from someone else

You'll get their recipes but not their profile — that's the design working. Run the
onboarding above, then decide what to do with `recipes/`: keep it as a starting library,
or empty it along with `INDEX.md` and build your own.

---

## Adding a recipe

Paste a link or the recipe text. That's the whole interface.

```
> https://www.instagram.com/reel/C1yeguCLhhB/
> https://youtube.com/shorts/fGAgGXSpfdM  — just the chilli paneer
> I made this myself: grilled paneer, stir-fry veg, hung curd dip
```

Claude extracts it, assigns tags, writes `recipes/<slug>.md`, updates
[`INDEX.md`](INDEX.md), and reports back with the recipe rendered in chat plus anything
it noticed — a ratio that looks off, a step the source skipped, what it would pair with.

It won't ask permission first. Everything you send is meant to be saved.

Things worth knowing:

- **It never invents quantities.** If the source omits an amount, the file says so
  rather than guessing.
- **Its own additions are marked `Claude:`** and kept under `## Notes`, never mixed into
  the source's steps. Your habits are marked `Me:`. The creator's words are unprefixed.
- **Say "just the guava dip"** and it files only that part.
- **Getting the recipe out of a video is often hard.** Claude works down a five-rung
  ladder — page text, then the creator's blog or YouTube (comment-gated reels almost
  always publish the recipe openly somewhere), then pinned comments, then audio
  transcription, then reading frames off the video. It records which rung it used.

## Asking what to cook

```
> /suggest-recipe with paneer
> /suggest-recipe something light, I have 30 minutes
> /suggest-recipe high protein with potato
> I've had pasta three times this week — something different?
```

You'll get 2–4 options with a line of reasoning each, scaled to your household size,
with your preferences already applied — plus **at least one combination that isn't a
filed recipe.** That last part is the point of the collection:

> The julienne curd salad is a slaw. Put it in the falafel smash wrap in place of the
> salad layer and it's a different lunch.

Push back and it refines rather than starting over. "Something lighter", "no paneer
again", "I don't have cornmeal" — it carries the context forward.

## Logging what you cooked

```
> I made the chilli paneer tonight
```

That appends a row to `MADE_LOG.md`, which is what "something different from what I've
been eating" gets answered against. Claude won't assume you cooked something just
because you saved it — an unlogged recipe is one you saved but never made, and the
distinction matters. If the log is empty, it'll tell you it's suggesting against what
you *saved* rather than what you *ate*.

---

## The tagging vocabulary

Every recipe gets tagged from a **fixed** list of eight qualities, so "high protein" is
a lookup rather than a judgement call each time:

`high-protein` · `light` · `filling` · `low-carb` · `high-fiber` · `carb-heavy` ·
`quick` · `comfort`

The list is closed on purpose — `high-protein` in one file and `protein-rich` in another
would make neither findable. Definitions are in [`CLAUDE.md`](CLAUDE.md).

Deliberately **no macro numbers** are stored. The filter is how a meal *eats*, not what
it totals. Ask for numbers on a specific dish and Claude will estimate them in chat and
show its working — it just won't write a precise-looking guess into a file.

Two other fields do the heavy lifting:

- **`key_ingredients`** — the 3–6 things that define the dish. This is what powers "what
  can I make with X"
- **`related`** and **`serve_with`** — cross-links, so taco night surfaces both fillings
  and the falafel arrives with the tzatziki

## Layout

| Path | What it is |
|---|---|
| [`INDEX.md`](INDEX.md) | Every recipe at a glance — cuisine, ingredients, qualities, time, effort |
| `recipes/` | One markdown file per recipe. Filename = the slug in its frontmatter |
| [`CLAUDE.md`](CLAUDE.md) | How Claude files recipes: the fetching ladder, extraction rules, tag definitions |
| `.claude/skills/suggest-recipe/` | How Claude answers "what should I cook?" — including onboarding |
| `PREFERENCES.md` | Your profile: allergies, diet, kitchen, habits — **untracked** |
| `MADE_LOG.md` | What actually got cooked, and when — **untracked** |

## Making it yours

- **`PREFERENCES.md`** — anything about *you*. Edit freely; Claude reads it before every
  suggestion and every filing.
- **`CLAUDE.md`** — the *procedure*. How recipes get extracted and tagged.
- **`.claude/skills/suggest-recipe/SKILL.md`** — how suggestions get made.

The split is deliberate: `CLAUDE.md` is the procedure, `PREFERENCES.md` is the person.
When you tell Claude something new about how you eat, it belongs in the second one.
