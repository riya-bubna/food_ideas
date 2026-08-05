---
name: suggest-recipe
description: Suggest what to cook from this recipe collection. Use whenever the ask is what to make — by ingredient ("what can I make with paneer?"), by quality ("something light", "high protein with potato"), by mood ("something different from what I've been eating"), or open-ended ("what should I cook tonight?"). Also use when the ask is for new ways to combine or repurpose recipes already filed.
---

# Suggesting recipes

This collection is a **parts bin, not a menu**. Half the value is retrieval; the other
half is naming a combination they haven't thought of. Every suggestion round should do
both.

Read two files before you start:

- **[`PREFERENCES.md`](../../../PREFERENCES.md)** — diet, household size, everyday
  baseline, components they skip. Untracked in git. Every scaling and substitution
  decision below depends on it.
- **[`CLAUDE.md`](../../../CLAUDE.md)** — field conventions and the `qualities`
  vocabulary.

## Load the data

Cheapest path, in order — don't read all of `recipes/` unless you need to:

1. **`INDEX.md`** — every recipe with cuisine, key ingredients, qualities, serves,
   time, effort, dietary. Most asks can be answered off this table alone.
2. **`MADE_LOG.md`** — required for any "something different" ask. Read the last 2–3
   weeks. **If it's empty or stale, say so** — you're then suggesting against what was
   *saved*, not what was *eaten*, and that difference should be stated, not hidden.
3. **The individual files** — only for the shortlist, to get quantities, preference
   callouts, and `related` / `serve_with`.

Useful greps:

```bash
grep -l "key_ingredients:.*paneer" recipes/*.md      # ingredient, central
grep -l "pantry_ingredients:.*lemon" recipes/*.md    # ingredient, incidental
grep -l "qualities:.*high-protein" recipes/*.md      # quality
grep -h "^serve_with:\|^related:" recipes/*.md       # recorded pairings
```

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
  near-miss and going to find something new.

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

- Don't suggest anything from the everyday baseline in `PREFERENCES.md` as a "something
  different" answer — that's the thing being varied *from*. Reworkings are fine: kala
  chana in a [kebab](../../../recipes/kala-chana-galouti-kebab.md) or rajma in a
  [taco](../../../recipes/crispy-bean-cheese-tacos.md) is a different dish.
- Don't surface `breakfast` or `dessert` entries in a lunch/dinner suggestion unless
  asked.
- Don't invent a recipe that isn't in the collection and present it as though it is. A
  *combination* of two filed recipes is fair game and is the point; a whole new dish
  from nowhere must be labelled as such.
- Don't put macro numbers in a suggestion unless asked. If asked, estimate in chat and
  show your working — don't file it.
