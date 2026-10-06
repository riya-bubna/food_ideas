---
name: suggest-recipe
description: Suggest what to cook from this recipe collection. Use whenever the ask is what to make — by ingredient ("what can I make with paneer?"), by quality ("something light", "high protein with potato"), by mood ("something different from what I've been eating"), or open-ended ("what should I cook tonight?"). Also use when the ask is for new ways to combine or repurpose recipes already filed.
---

# Suggesting recipes

This collection is a **parts bin, not a menu**. Half the value is retrieval; the other
half is naming a combination they haven't thought of. Every suggestion round should do
both.

Read two files before you start:

- **[`PREFERENCES.md`](../../../PREFERENCES.md)** — allergies, diet, kitchen, household
  size, everyday baseline. Untracked in git. Every filter, scaling and substitution
  decision below depends on it.
- **[`CLAUDE.md`](../../../CLAUDE.md)** — field conventions and the `qualities`
  vocabulary.

## Step 0: is this a new user?

**Check `PREFERENCES.md` before anything else.** Treat it as unset if it is missing,
empty, still the template, or has sections left as `not yet asked`.

If it's unset, **onboard first and suggest nothing until it's written.** A suggestion
made without this file isn't a weaker suggestion — it's one that can recommend food
someone is allergic to, can't cook, or won't eat. Say plainly that you're setting up
the profile once so you don't have to ask every time, then run the interview.

**The interview: two rounds of `AskUserQuestion`, batched.** (The one-round limit later
in this file governs *refining a suggestion*, not onboarding.)

Round 1 — the hard filters:

1. **Allergies and intolerances** — `multiSelect`. Offer the common ones (nuts, dairy,
   gluten, soy, sesame, shellfish, egg) plus "none". **Always ask this, never infer it,
   and never skip it because the diet answer seems to cover it.** Vegetarian says
   nothing about nuts.
2. **Diet** — vegetarian, vegan, eggetarian, eats everything. Follow up on what should
   happen to a recipe containing something they don't eat: substitute it, or skip the
   recipe entirely? That single answer changes what gets *filed*, not just suggested.
3. **Household size** — how many people they cook for.
4. **Kitchen** — `multiSelect` over oven, air fryer, microwave, pressure cooker,
   induction/gas hob. What they *lack* is the constraint that matters.

Round 2 — the things that make suggestions good rather than merely safe:

5. **Everyday baseline** — what they already eat most weeks. Without this, "something
   different" has no reference point and gap analysis misfires.
6. **Ingredients they don't buy** — canned goods, pre-made pastes, anything always made
   from scratch.
7. **Strong dislikes** — distinct from allergies. Coriander, mushrooms, blue cheese.

Then **write `PREFERENCES.md`** from
[`PREFERENCES.template.md`](PREFERENCES.template.md) in this skill's directory, filling
every section. Record allergies as an explicit list or the literal words "None
reported" — never leave it ambiguous whether the question was asked. Confirm `.gitignore`
covers the file. Show them what you wrote, say it's editable by hand, and only then
answer the original question.

**Partial profiles:** if the file exists but the section you need is blank, ask about
that one section rather than re-running the whole interview.

**Allergies are a hard filter on every suggestion afterwards.** Read the full ingredient
list, not just `key_ingredients` — allergens hide in `pantry_ingredients` and garnishes.
`dietary: [contains-nuts]` is a summary, not a substitute for reading the recipe.

## Load the data

Cheapest path, in order — don't read all of `recipes/` unless you need to:

1. **`INDEX.md`** — every recipe with cuisine, key ingredients, qualities, serves,
   time, effort, dietary. Most asks can be answered off this table alone.
2. **`MADE_LOG.md`** — required for any "something different" ask. Read the last 2–3
   weeks. **If it's empty or stale, say so** — you're then suggesting against what was
   *saved*, not what was *eaten*, and that difference should be stated, not hidden.
3. **The individual files** — only for the shortlist, to get quantities, preference
   callouts, and `related` / `serve_with`.

### Looking things up

```bash
grep -l "key_ingredients:.*paneer" recipes/*.md      # ingredient, central
grep -l "pantry_ingredients:.*lemon" recipes/*.md    # ingredient, incidental
grep -l "qualities:.*high-protein" recipes/*.md      # quality
grep -h "^serve_with:\|^related:" recipes/*.md       # recorded pairings

# intersect two facets
comm -12 <(grep -l "qualities:.*high-protein" recipes/*.md | sort) \
         <(grep -l "qualities:.*quick" recipes/*.md | sort)
```

The vocabulary is normalized, so one spelling finds everything — `bell pepper` returns
the nine recipes that used to be split across `capsicum`, `red pepper` and
`bell pepper`. Keep it that way when filing.

**If a grep returns nothing, check the word is one the collection actually uses before
concluding the food is absent.** An empty result means the vocabulary doesn't use that
term — that's when Epicure's `neighbors` earns its keep.

## Epicure: the flavour-pairing model

An external MCP server, registered in [`.mcp.json`](../../../.mcp.json) as `epicure`
(`https://epicure-mcp.kaikaku.ai/mcp` — public, read-only, no auth). It exposes a
300-dimensional embedding of 1,790 ingredients trained on a 4.14M-recipe corpus, plus
a 2D UMAP atlas and 20 emergent flavour factors.

**What it is for:** answering *"what goes with this?"* when the collection alone can't.
It is a pairing oracle, not a recipe source — it returns ingredient relationships, never
dishes.

**Reach for it when:**

- They name an ingredient nothing here is central to, and you'd otherwise be offering a
  near-miss. Ask Epicure what it wants to sit next to, then look for *that* in `INDEX.md`.
- You're building a recombination and want to check the pairing actually holds rather
  than trusting your own taste.
- They want a substitution — `neighbors` is a direct "what's near this in flavour space"
  answer.
- The ask is explicitly exploratory: "surprise me", "what's an unusual pairing for X".

**Don't reach for it when** the collection already answers the question cleanly. A
`grep` over `key_ingredients` is faster and the answer is a real recipe rather than a
list of nouns.

### The tools

| Tool | Use it for |
|---|---|
| `find_pairings` | The workhorse. Seed ingredients → clustered flavour directions plus bridge ingredients. Pass `is_vegetarian: true` — always, given [`PREFERENCES.md`](../../../PREFERENCES.md). |
| `neighbors` | Substitutions and "what's like X". Ranked nearest ingredients, no clustering. |
| `pairing_score` | Check one specific pair before you propose it. Cosine plus a corpus percentile. |
| `cultural_profile` | Which of eight macro-cuisines an ingredient leans toward. |
| `compare_on_axis` | Two ingredients on one named axis (sweetness, a cuisine direction). `list_targets` gives valid axes. |
| `morph` | Directed transformation — rotate an ingredient toward a cuisine or another ingredient. `angle_deg` 0–90; 45 is a balanced morph. |
| `where_on_atlas` / `closest_mode` | Spatial context; which named flavour region an ingredient occupies. |
| `pareto_navigate`, `ingredient_on_factor`, `list_factors`, `flavour_correlations`, `list_targets` | Deeper structure. Rarely needed for a cooking question. |

Two API details that aren't guessable: `morph`'s `target` is an **object**, not a
string — `{"kind": "direction", "name": "South_Asian"}` — and cuisine names use
underscores (`South_Asian`, `Eastern_European`). `list_targets` returns the valid set.

### Reading its output honestly

- **The corpus skews Western.** Ask it about zucchini and it returns feta, marinara,
  pesto, rosemary. That is a real signal about the corpus, not about what's worth
  cooking. When the answer should lean Indian or Asian, use `cultural_profile` to check
  the ingredient's own lean, or `morph` it toward `South_Asian` — don't just accept the
  first cluster. Zucchini profiles Mediterranean 0.19 (against a p90 of 0.21) and South
  Asian −0.05; morphed 45° toward South Asian it lands on ajwain, chana dal and paneer.
  That second answer is the useful one here, and you only get it by asking for it.
- **Scores are relative, not absolute.** Every response prints its own reference band
  (`p10`, `median`, `p90`). A 0.30 means nothing until you read it against that band.
  Quote the percentile label, not the bare number.
- **It knows co-occurrence, not cooking.** It cannot tell you a method, a ratio, or
  whether something is edible together in the quantities involved. It says these things
  appear in recipes together. The judgement stays yours.
- **It is not a safety check.** The server says so itself. Allergies, diet and the
  kitchen constraints in [`PREFERENCES.md`](../../../PREFERENCES.md) are filters you
  apply *after* Epicure answers, never something you delegate to it.

### How it shows up in a suggestion

Same rules as everything else here: **name the source of the idea.** If a pairing came
from Epicure rather than from a filed recipe, say so in one clause — "Epicure puts
zucchini and feta at the top of its vegetable cluster, which is why the
[asparagus feta salad](../../../recipes/asparagus-feta-salad.md) is the natural host."
That keeps the model's suggestions distinguishable from the collection's, the same way
`Claude:` stays distinguishable from `Me:` in a recipe file.

Never present an Epicure pairing as a filed recipe. It's raw material for a
recombination, and a recombination is still yours to construct and justify.

## Read the ask along five axes

Most prompts specify one or two. Note which are **unspecified** — that's what you may
need to ask about.

| Axis | Looks like | Fields |
|---|---|---|
| Ingredient | "with paneer", "I have mangoes" | `key_ingredients`, then `pantry_ingredients` |
| Quality | "light", "high protein", "filling" | `qualities` |
| Time / effort | "quick", "weeknight", "I have time" | `active_time_min`, `total_time_min`, `effort` |
| Format | "a wrap", "one-pot", "something to snack on" | `meal_type`, `tags` |
| Contrast | "different from what I've been eating" | `MADE_LOG.md` |

For a contrast ask, work out what's actually repetitive about the last few weeks —
same cuisine? same format? same protein? all pasta? — and deliberately suggest
*against* that axis. **Name the rut out loud**: "you've made three tomato-based pastas,
here's a dry-spiced stir-fry." That way they can tell you if you read it wrong.

## Ask before you search — but only once, and only if it changes the answer

This is meant to be iterative, so use `AskUserQuestion` freely at the start. Good
reasons to ask:

- The shortlist is large (8+) and one question would cut it to three.
- Two readings of the ask give genuinely different food — "something with mango" could
  be the [stir fry](../../../recipes/mango-paneer-stir-fry.md) or the
  [salad](../../../recipes/mango-peanut-lime-salad.md).
- You don't know the time budget and the shortlist spans `easy` to `involved`.
- They named an ingredient nothing here is central to, and the choice is between a
  near-miss, asking Epicure what it pairs with, and going to find something new.

Do **not** ask when a sensible default exists. Weeknight means `easy` or `medium`.
"Tonight" means don't propose `involved`. Just pick and say what you assumed.

Ask at most one round before suggesting. Further refinement happens against a real
shortlist, which is easier to react to than questions in the abstract.

## Then suggest

**2–4 options. One line of reasoning each. Always linked.** Not a single pick, not an
exhaustive dump.

For each: what it is, why it fits *this* ask, and the one thing that would make them
say no (time, an ingredient they'd have to buy, a component they skip).

Then, in the same message:

- **Screen against the allergy list first.** Drop anything containing a recorded
  allergen before you rank, and read the full ingredient list to do it, not just
  `key_ingredients`. Don't mention what you dropped unless they ask — a suggestion list
  shouldn't dangle food they can't eat.
- **Scale to the household size in `PREFERENCES.md`.** If the file says `serves: 4`,
  say "half of this" and spell out anything awkward to divide. Don't leave them doing
  the arithmetic.
- **Apply their preferences by default.** Where a file has a blockquote callout under
  the source link, lead with their version — the
  [paneer steak](../../../recipes/paneer-steak.md) is "crusted steak with muhammara",
  not "steak with tomato-basil sauce". Egg and meat get the vegetarian version first,
  the source's second.
- **Honour `serve_with`.** A recipe filed with `serve_with: [couscous, tzatziki]` is
  offered as that plate, not bare.
- **Name what you relaxed.** If nothing satisfies every constraint, say which one you
  dropped and why. Never quietly return something that meets one of two.

If genuinely nothing fits, say so and offer to go find a recipe. A forced near-miss is
worse than an honest miss.

## Always offer at least one recombination

This is the point of the collection, not a garnish. Every suggestion round includes at
least one thing that **isn't already a filed recipe**.

Check the recorded pairings first — `related`, `serve_with`, the "Pairings recorded"
section of `INDEX.md`. Then generate new ones along these axes:

**Swap the format.** Most entries are *filling + format*. Change the format and it's a
different meal. Formats already present: rice bowl, quinoa bowl, wrap/pita, taco,
toast/sandwich, glass noodles, pasta, plate-with-dip, salad.
→ *The [julienne curd salad](../../../recipes/julienne-curd-salad.md) is a slaw; put it
in the [falafel smash wrap](../../../recipes/falafel-smash-wrap.md) in place of the
salad layer and it's a different lunch.*

**Move a dip.** Every dip was filed against one dish and works against several. Dry,
crisp, or fried things need one; rich things need an acidic one.
→ *[Beet hummus](../../../recipes/beet-hummus.md) under the
[air-fryer paneer tikka](../../../recipes/air-fryer-paneer-tikka.md).*

**Fill a template's slots.** Some entries are explicitly swappable —
[paneer/veg/hung-curd plate](../../../recipes/paneer-veg-hung-curd-plate.md) is three
slots; the [kale fruit salad](../../../recipes/kale-fruit-feta-salad.md) and
[oats pancakes](../../../recipes/oats-coconut-pancake-mango.md) swap the fruit. Refill
them from elsewhere rather than repeating them as filed.

**Fix a macro gap.** Pair a `light` or `carb-heavy` side with a `high-protein` main
instead of suggesting either alone.

**Borrow a technique.** A method from one file applied to another ingredient — the
massaged-kale marinade, the air-fryer instead of shallow-frying, tearing tofu instead
of cubing.

**Follow the model.** Run `find_pairings` on the ingredient in hand, then look for its
top bridges in `INDEX.md`. Epicure's job here is to point at a recipe you'd have
overlooked, not to supply the pairing itself — the suggestion is still two named files
and a reason.

Rules for a good combination:

- **Name both files and say what the pairing gives that neither gives alone.**
  "Serve the falafel with the beet hummus" is useful. "These could be combined" is not.
- Contrast textures — creamy against crisp, rich against sharp.
- Stay in a coherent flavour register, or say explicitly that you're crossing one.
- Respect the effort budget. Two `involved` recipes is not a suggestion, it's a project.
- If a combination is good enough to keep, offer to file it as its own recipe.

## Nudging a recipe rather than replacing it

Sometimes the ask isn't "what should I cook" but "how do I make *this* better" — higher
protein, lighter, less sodium. Answer with concrete swaps, not general advice: chickpea
pasta instead of wheat, paneer stirred through at the end, hung curd in place of cream,
tamari cut with water. Say what each swap costs — texture, cooking time, flavour — so
they can decide. Where the fix already exists as another file, link it.

## Stay in the loop

Expect pushback, and treat it as refinement rather than a fresh question. "Something
lighter", "no paneer again", "I don't have cornmeal" — carry the context forward,
adjust one axis, re-suggest. Don't restart the interview.

Once they settle on one:

1. Give the full ingredient list **scaled to the household size**, alongside the
   source's own figures.
2. Flag anything they'd need to buy.
3. Point out any prep that has to start early — marinating, pressing, chilling.
4. Offer substitutions for what they don't have, and be honest about what changes.
   Mark these `Claude:` if they go into a file.
5. **Offer to log it** in `MADE_LOG.md` — but only once they say they made it. Saved
   and cooked are different things, and the log is only useful if it stays true.

## Don't

- **Don't suggest anything containing a recorded allergen**, and don't offer a
  substitution as though it makes the dish safe unless it genuinely does. This applies
  to recombinations too — a pairing you invented is as capable of carrying an allergen
  as a filed recipe.
- **Don't suggest from an unset `PREFERENCES.md`.** Onboard first. See Step 0.
- Don't suggest anything from the everyday baseline in `PREFERENCES.md` as a "something
  different" answer — that's the thing being varied *from*. Reworkings are fine: kala
  chana in a [kebab](../../../recipes/kala-chana-galouti-kebab.md) or rajma in a
  [taco](../../../recipes/crispy-bean-cheese-tacos.md) is a different dish.
- Don't surface `breakfast` or `dessert` entries in a lunch/dinner suggestion unless
  asked.
- Don't invent a recipe that isn't in the collection and present it as though it is. A
  *combination* of two filed recipes is fair game and is the point; a whole new dish
  from nowhere must be labelled as such.
- Don't hand Epicure a filter it can't enforce. It has a vegetarian flag and nothing
  else — allergies, the missing oven and the no-canned-legumes rule are yours to apply
  to its output.
- Don't quote a bare cosine number. It's meaningless without the reference band printed
  alongside it; give the percentile label instead.
- Don't put macro numbers in a suggestion unless asked. If asked, estimate in chat and
  show your working — don't file it.
