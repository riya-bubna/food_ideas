# Preferences

Personal profile for this collection. **Not tracked in git** — see `.gitignore`.

`CLAUDE.md` and the `suggest-recipe` skill both defer to this file for anything about
diet, household size, or eating habits. Keep those facts here and nowhere else.

> Template. Every section below must be filled or explicitly marked "none" before
> suggesting recipes. "None" and "not yet asked" are different answers — never leave a
> section ambiguous between the two.

## Allergies and intolerances

**Status:** *not yet asked*

<!-- Fill with the allergen list, or the literal words "None reported." -->

An allergy is a **hard filter**, not a preference:

- Never suggest a recipe containing the allergen, and never present a substitution as
  making the dish safe unless it genuinely does.
- Check the **full ingredient list**, not just `key_ingredients`. Allergens hide in
  `pantry_ingredients` and in garnishes — tahini and sesame oil, soy sauce (wheat),
  ghee and paneer, nuts scattered as a topping.
- The `dietary` field carries `contains-nuts`, `contains-egg` and `gluten-free`, but it
  is a summary, not a substitute for reading the ingredients.
- Cross-contamination and shared equipment are the cook's judgement, not yours. Flag it
  where it matters; don't rule on it.
- Distinguish an **allergy** (never suggest) from an **intolerance** (flag the amount)
  from a **dislike** (offer the version without it). Record which is which.

## Diet

<!-- vegetarian / vegan / eggetarian / pescatarian / eats everything, and what happens
     to a recipe containing something they don't eat: replaced, or recipe skipped? -->

## Kitchen

<!-- What they cook on: oven, air fryer, microwave, pressure cooker, gas hob, induction.
     Note anything they DON'T have — that's the constraint that matters, and it makes
     recipes uncookable as written rather than merely inconvenient. Record the
     conversion rules alongside, so the constraint is actionable. -->

## Ingredients I don't buy

<!-- Canned goods, pre-made pastes, specific brands, anything they always make from
     scratch. Include the translation where relevant — e.g. can → drained → dry weight —
     so the substitution is arithmetic, not guesswork. -->

## Household

<!-- How many people they cook for. Recipe files keep the source's serving count; the
     scaling happens at suggestion time. -->

## What I eat outside this collection

<!-- The everyday baseline. Critical: it changes what counts as a gap, and it's what
     "something different" is measured against. Without this, gap analysis and
     "suggest something new" both go wrong. -->

## Components I skip or change

<!-- Recorded per-recipe as a blockquote callout in the recipe file itself, so the note
     travels with the recipe when shared. This section holds the rule, not the list. -->

## Voice conventions in recipe files

Three voices, always distinguishable:

| Prefix | Whose |
|---|---|
| *(unprefixed)* | The source creator |
| `Me:` | Mine — my habits, amounts, substitutions |
| `Claude:` | Yours — additions, warnings, technique fixes |

## Sharing

<!-- Who else reads these files — a cook, a partner, nobody. Drives how much context
     each recipe file has to carry on its own. -->
