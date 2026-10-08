# Acceptance criteria — FitFindr

Five criteria that say what "working" means for this agent, written in unit 3
**before** any results existed.

An acceptance criterion names a target: a number, a count, a rate, or something
a person could plainly observe. _"The agent handles errors"_ is an opinion.
_"When search returns nothing, the agent stops before calling the second tool,
in 5 of 5 tries"_ is a criterion.

Under each one, write a sentence or two on **why that target** and not a
stricter one. A reason that says something about your tools, your loop, or the
data earns credit; _"80% seemed reasonable"_ does not.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

**Two are written for you. You write three.**

---

## 1. A matching query completes all three tools

Given a query that matches at least one listing, the agent completes all three
tool calls and returns a fit card — in at least 4 of 5 tries.

**Why this target:**
My search is a plain keyword match and some phrasings will miss — "Y2K baby tee" might not match a listing titled "Butterfly graphic crop top from the 2000s," so 4 of 5 allows for that natural variance in language.

---

## 2. An impossible query stops before the second tool

Given a query that matches no listings, the agent stops before calling
`suggest_outfit` and returns a message naming what to change — 5 of 5 tries.

**Why this target:**
This path is deterministic — if search returns empty, the branch should always trigger. There's no randomness or language matching involved, so 5 of 5 is reasonable. This is the critical safety check.

---

## 3. The selected item is the first search result

In 5 of 5 tries with a matching query, `session["selected_item"]["id"]` equals `session["search_results"][0]["id"]` — the item passed to suggest_outfit is the same one search_listings found.

**Why this target:**
This is deterministic. The code always picks the first result, so the ID should always match. If it doesn't, something corrupted the data or swapped the wrong item into the session, which would be a real bug.

---

## 4. Fit cards are useful and varied

For five different items, all five fit cards should mention the price and platform, be 2–4 sentences long, and no two opening sentences should be identical.

**Why this target:**
The model varies its wording, which is fine, but every caption needs price and platform to be useful on social media. Long captions don't work. Different items should produce different openings, or the tool isn't actually paying attention to what it's describing.

---

## 5. Price ceiling is enforced

In 5 of 5 tries with a `max_price` set, all results from `search_listings` have `price <= max_price`. No results exceed the budget.

**Why this target:**
If I set a budget, I want the agent to stay within it. I can always raise my budget manually if I want to, but the agent shouldn't surprise me with results above what I asked for. This is deterministic — filtering on a number should always work.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     UNIT 4 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath it, like this:

         ## 4. Something about the fit card

         The fit card is different every time.

         **Why this target:** ...

         > **Revised in unit 4:** For 5 different items, the 5 fit cards share
         > no opening sentence.
         >
         > **Why revised:** "different" wasn't checkable — two cards that
         > differed by one word still counted. The new version is something I
         > can actually score.

     That's a revision because the criterion couldn't be MEASURED.

     Lowering a target because you missed it is not a revision, and it costs
     you the point:

         ✗ "I said the empty search stops it 5 of 5 times, but I got 3 of 5,
            so 3 of 5 is more realistic."

     A number you missed stays where it is, gets diagnosed, and gets a fix
     attempted. That's where the points are.
     ───────────────────────────────────────────────────────────────────────── -->
