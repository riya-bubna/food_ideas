# Recipe Database

A personal collection of recipes the owner cooks from. They send links (Instagram
reels, YouTube, TikTok, blogs) or pasted text; you extract the recipe and file it
here. They then ask things like "what can I make with paneer?" or "I've had pasta
three times this week, give me something else."

## Read this first

**[`PREFERENCES.md`](PREFERENCES.md) holds the owner's personal profile** — diet,
household size, everyday baseline, and how they want their own habits recorded. It is
deliberately untracked in git. Read it before filing a recipe or making a suggestion;
almost every judgement call here depends on it.

Keep personal facts in that file and nowhere else. This file is the *procedure*; that
one is the *person*. If you learn something new about how they eat, it goes there.

**If `PREFERENCES.md` is missing, empty, or still the template, the owner is new.** The
[`suggest-recipe` skill](.claude/skills/suggest-recipe/SKILL.md) owns that setup — it
runs a short interview and writes the file. Don't suggest food before it exists, and
don't file a recipe with substitutions or scaling guessed from nothing.

## Repository layout

```
recipes/          One markdown file per recipe. Filename = slug in frontmatter.
INDEX.md          Table of every recipe. Regenerated whenever a recipe is added.
MADE_LOG.md       Reverse-chronological log of what actually got cooked. Untracked.
PREFERENCES.md    Personal profile. Untracked.
```

`MADE_LOG.md` and `PREFERENCES.md` are both in `.gitignore` — they're personal, and
the recipes are the part meant to be shareable. Keep it that way: nothing that
identifies the owner or records their habits belongs in a tracked file.

## Adding a recipe

When a link or recipe text arrives:

1. **Extract it** — see the fetching ladder below. Work down it; stop at the first
   rung that yields the recipe.
2. **Assign `qualities`** per the section below.
3. **Write `recipes/<slug>.md`** using the template below.
4. **Update `INDEX.md`** with a new row.
5. **Report back** with the recipe rendered in chat, plus anything notable you
   observed (substitutions, ratios that look off, steps the source skipped).

Do not ask permission before filing — every recipe sent is meant to be saved. Just
save it and say where it went.

### Their own recipes

Sometimes they cook something of their own and describe it, often with a photo. File
it like any other recipe, with `source: "Own creation"`, no `source_url`, and a
`**Source:** My own` line in the body. Record what they describe; don't invent
quantities they didn't give. These are worth having — they're the only entries that
reflect what actually gets cooked unprompted rather than what got saved.

## The fetching ladder

Most links give up their recipe on rung 1 or 2. Work down only as far as you need,
and **record in `## Notes` which rung you used** and what came from the source versus
from you.

### 1. The text on the page

- **Instagram**: `WebFetch` on the `/reel/<id>/` form. Older posts often work; newer
  ones hit a login wall. If it fails, use the browser tools on the `/p/<id>/` form,
  which renders the caption and comments in a readable DOM.
- **YouTube**: `WebFetch` fails, but curling the watch page and pulling
  `"shortDescription"` out of the embedded JSON works reliably and is cheap. Full
  recipes live in the description far more often than on Instagram.

### 2. Somewhere else the creator published it

This is the highest-yield rung and the most often skipped. Comment-gated posts
("comment RECIPE and I'll DM you") almost always publish the same recipe openly.

- A recipe URL right in the caption — check before assuming it's gated.
- The creator's site via their **link in bio**. Guessing the slug from the dish name
  frequently works.
- Their site's `/recipe-index/` or `/blog` if the slug guess misses.
- **Their YouTube channel.** Instagram creators often post the same recipe there with
  a real description. `yt-dlp --flat-playlist --print "%(id)s :: %(title)s"` over
  `@handle/shorts` lists every title to grep.

Never post a comment to trigger a DM. That's the owner's account, and it isn't yours
to post from.

### 3. The pinned comment

Instagram captions read fine logged out, but pinned comments need a login, and the
session expires often. YouTube pinned comments need the browser and a scroll to the
top of the thread. Worth checking when a caption says "recipe in comments" — but note
the pinned comment is sometimes deleted, in which case say so rather than hunting.

### 4. Transcribe the audio

For narrated videos. Needs the video file:

- **YouTube**: `yt-dlp -f bestaudio -x --audio-format wav` works directly.
- **Instagram**: you cannot download it. The page serves a `blob:` URL and the signed
  CDN URLs are blocked from the browser tool by design. **Ask for the file** — they
  use `indown.io/reels/en` and drop the `.mp4` into the project.

Then whisper. Use the `small` model, not `base`. For Hindi, `task="translate"`
returns clean English — transcribing Hindi natively produces garbled Perso-Arabic
script that's unusable.

### 5. Read the frames

For videos with burned-in text captions and no narration. Extract a contact sheet
with `ffmpeg` and `Read` it as an image:

```
ffmpeg -i video.mp4 -vf "fps=1/2,scale=520:-1,tile=3x7" -frames:v 1 sheet.png
```

Sample about every 2 seconds — captions change fast and a 7-second interval misses
half of them. If text is dense, crop to the caption band to fit more frames legibly:
`crop=iw:ih/4:0:ih*3/4`.

### When nothing works

Say so plainly and offer the choice: they watch it and tell you, they send a
screenshot, or you skip it. A recipe where you'd be guessing the *ingredients* — not
just the amounts — is not worth filing.

**Always delete downloaded media when you're done**, both from the project root and
from the scratchpad.

### Rules for extraction

- **Never invent quantities.** If the source omits an amount, write `to taste` or
  `amount not specified` in the ingredient line. Do not guess and do not silently
  round.
- **Preserve the source's own measurements** (metric, cups, "1 katori") in the
  ingredient list. Add a converted amount in parentheses only if it genuinely
  helps.
- **Separate your own additions.** Anything you contribute — a substitution, a
  technique fix, a warning that the sauce will break — goes under `## Notes`, never
  mixed into the source's steps. Prefix each with `Claude:`. See the voice
  conventions in [`PREFERENCES.md`](PREFERENCES.md).
- **Steps should be actionable.** Rewrite rambling video narration into numbered
  steps, but do not change the method.
- **Check the method against the kitchen.** [`PREFERENCES.md`](PREFERENCES.md) lists
  the equipment and the ingredients that don't get bought. Where a recipe assumes
  something unavailable — an oven, a can of chickpeas — keep the source's method intact
  and add a `Claude:` note with the conversion. Never silently rewrite the source's
  steps to fit.
- **Slug** = kebab-case, descriptive, no dates. `muhammara-pasta.md`, not
  `recipe-1.md`.
- **Always put a visible `**Source:**` link in the body**, not just in the
  frontmatter — these files get shared with a cook. If the video links out to a
  fuller written recipe, record that as `recipe_url` and link both.
- **If told to take only part of a recipe** ("just the guava dip"), file only that
  part. Name the parent dish in a line under the source link so the context isn't
  lost.
- `related` — slugs of recipes that are alternatives to or components of this one.
  Add it in both directions when something is framed as "another option for X", and
  link the file in the body too. It's how taco night surfaces both fillings.
- `serve_with` — how they actually eat it, when they say so. Use it when suggesting: a
  falafel recipe with `serve_with: [couscous, tzatziki]` should be offered as that
  bowl, not as bare falafel. Prefix such notes `Me:` in the body so their habits stay
  distinguishable from the source and from you.

### Recipe template

```markdown
---
title: Muhammara Pasta with Caramelized Onion
slug: muhammara-pasta
source: "@ohcheatday"
source_url: https://www.instagram.com/reel/C1yeguCLhhB/
source_date: 2024-01-06
added: 2026-08-05
cuisine: [middle-eastern, italian]
meal_type: [dinner, lunch]
serves: 4
active_time_min: 25
total_time_min: 50
effort: medium              # easy | medium | involved
dietary: [vegetarian]       # vegetarian | vegan | gluten-free | contains-nuts | contains-egg
key_ingredients: [red pepper, walnut, pasta, onion, tomato]
pantry_ingredients: [olive oil, breadcrumbs, lemon, tomato paste, kashmiri chilli]
tags: [pasta, sauce-freezes-well, one-pot]
qualities: [filling, carb-heavy, high-fiber, comfort]
serve_with: [flatbread]     # optional — how it actually gets eaten
---

**Source:** [@ohcheatday on Instagram](https://www.instagram.com/reel/C1yeguCLhhB/)

## Ingredients

### For the sauce
- 2 red peppers
- ...

## Instructions

1. ...

## Notes

Claude: anar dana powder is the flavor backbone; substitute 1–2 tbsp pomegranate
molasses.
```

### Field conventions

- `key_ingredients` — the 3–6 things that define the dish and that someone would
  plausibly search by. Lowercase, singular, unbranded. This is what powers "what
  can I make with X", so be disciplined: `walnut`, not `1/3 cup California
  walnuts`.
- `pantry_ingredients` — everything else that's shoppable. Skip salt, pepper,
  water, oil-as-a-medium.
- `effort` — `easy` is under ~20 min and mostly hands-off. `involved` means
  marinating, proofing, multi-day, or heavy knife work.
- `dietary` — tag what's true of the recipe **as written**. If it's trivially
  veganizable, note that under `## Notes` rather than tagging it `vegan`.
- Omit a field entirely rather than writing `unknown`.

## Qualities

The filter here is how a meal *eats*, not macros — "high protein containing potato",
"something light tonight". So every recipe gets a `qualities` list drawn from a
**fixed vocabulary**. Do not invent new values; a controlled list is the whole point,
because `high-protein` and `protein-rich` in different files means neither is
findable.

| Quality | Assign when |
|---|---|
| `high-protein` | Roughly 20 g+ protein per serving — a substantial meat, fish, egg, paneer, dal, or legume component. Not a sprinkle of nuts. See [`PREFERENCES.md`](PREFERENCES.md) for why the bar sits here. |
| `light` | Around 400 kcal or less per serving. Broths, salads, steamed or grilled things. Leaves you able to eat again in a few hours. |
| `filling` | Sits heavy and holds you for hours. Big carb or fat load, around 700 kcal+ per serving. |
| `low-carb` | Roughly 20 g carbs or less — no rice, bread, pasta, potato, or sugar to speak of. |
| `high-fiber` | Legume-, whole-grain-, or vegetable-dense enough to notice. |
| `carb-heavy` | Pasta, rice, bread, or potato is the bulk of the dish. |
| `quick` | On the table in 30 minutes or less, start to finish. |
| `comfort` | Rich, warming, indulgent. The thing you want when it's raining. |

Assign every quality that applies. Most recipes get two or three. `light` and
`filling` are mutually exclusive; plenty of dishes are neither.

The gram figures above are **your calibration, not data to store**. Estimate
roughly from ingredient weights and serving count to decide which tags apply, then
write only the tags. Don't put macro numbers in the frontmatter — they aren't wanted,
and a precise-looking estimate invites more trust than it deserves.

Tag against the food, not the reputation of an ingredient — a paneer dish with
100 g of paneer split four ways is not `high-protein`.

If asked for actual numbers on a specific dish, estimate them in chat and show your
working. Just don't file them.

## Answering questions

**The full procedure lives in the [`suggest-recipe` skill](.claude/skills/suggest-recipe/SKILL.md).**
Invoke it whenever the question is what to cook — by ingredient, by quality, by mood,
or open-ended — and whenever the ask is for new ways to combine what's already here.
Keep the procedure there, not here, so the two can't drift apart.

Two things that belong in this file because they shape more than suggestions:

**Mix and match is a core purpose, not a bonus.** The point is new ways to eat what's
already here, not just retrieval of what got filed. So when something *new* arrives,
look for what it could pair with or replace in something already here, and mention it
in the filing report. The
[paneer/veg/hung-curd plate](recipes/paneer-veg-hung-curd-plate.md) is the clearest
example: three slots, each fillable from elsewhere in the collection.

**Be concrete.** "Serve the falafel with the beet hummus" is useful; "these could be
combined" is not. Name the two files and say what the combination gives that neither
does alone.

## Logging what got cooked

When they mention making something, append a line to the top of the log table in
`MADE_LOG.md`. If they don't mention it, don't assume — an unlogged recipe is one that
was saved but never cooked, and that distinction matters for suggestions.
