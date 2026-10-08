# FitFindr

> ### 👋 Start here
>
> **New to this repo? Read [RUNNING.md](RUNNING.md) first** — setup, every
> command, and what to do when something breaks.
>
> Once `python test.py` passes:
>
> ```bash
> python app.py listings --full -n 6      # read the data (Milestone 1)
> python app.py fields                    # what you can filter on
> python app.py ask 'vintage graphic tee under $30'
> ```
>
> All three tools are stubs, so that last command will do nothing useful yet.
> That's the starting position.
>
> **The rest of this file is your submission.** Fill it in as you go.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     HOW TO USE THIS FILE

     This is your submission. Fill each section in as you finish the milestone
     it belongs to — don't leave it all to the end.

     Unit 3 asks for the first five sections. Unit 4 adds the five below them.
     Leave the unit 4 sections alone until then; they're here so you know
     what's coming.

     Everything is pasted as TEXT. No screenshots, no images, no video links.
     A typed block of output gets full credit; a picture of the same output
     gets none.
     ───────────────────────────────────────────────────────────────────────── -->

<!-- ═══════════════════════ UNIT 3 — THE BUILD ═══════════════════════ -->

## What This Does

A user tells FitFindr what they're looking for — a vintage graphic tee, size M, under $30. The agent searches the listings, finds a match, suggests how to style it with pieces they already own, and writes a caption they could post. If nothing matches the search, it tells them what to try instead.

---

## Tool Inventory

### `search_listings`

- **What it does:** Searches the listings data and returns items matching the description, size, and price criteria.
- **Inputs:** `description` (str), `size` (str | None), `max_price` (float | None)
- **Returns:** A list of listing dicts, sorted by keyword relevance (best match first). Each dict has id, title, description, category, style_tags, size, condition, price, colors, brand, platform.
- **When it has nothing:** Returns an empty list (not None, not an exception).

### `suggest_outfit`

- **What it does:** Takes a new item and the user's wardrobe, then suggests one or two outfit combinations using pieces they already own.
- **Inputs:** `new_item` (dict — a listing dict), `wardrobe` (dict with 'items' key containing a list of wardrobe items)
- **Returns:** A non-empty string describing outfit ideas (e.g., "Pair this with your baggy jeans and chunky white sneakers for a Y2K look").
- **When it has nothing:** If the wardrobe is empty, returns general styling advice for the item instead of outfit combinations.

### `create_fit_card`

- **What it does:** Writes a short, social-media-ready caption for posting about a thrifted find.
- **Inputs:** `outfit` (str — the outfit suggestion from suggest_outfit), `new_item` (dict — a listing dict)
- **Returns:** A 2–4 sentence caption that mentions the item, price, platform, and vibe. Should sound like a real post, not a product description.
- **When it has nothing:** If outfit is empty or only whitespace, returns a descriptive message explaining that outfit info is missing.

---

## Planning Loop

<!-- Your branch rule, stated as a rule — the condition AND both paths — plus
     the file and function that holds it.

     Like this:
       "If search_listings returns an empty list, put a message in the session
        and stop. Otherwise take the first result and go to suggest_outfit."
        — agent.py::run_agent

     The grader checks your code against what you claim here, so the file and
     function have to be real. -->

**Branch rule:** If `search_listings` returns an empty list, put an error message in the session (telling the user what to change) and return early. Otherwise, take the first result and pass it to `suggest_outfit`.

**Where it lives:** `agent.py::run_agent`

**How the query is parsed:** Regex. Extract `size` using pattern `r'size\s+([a-zA-Z0-9/\s]+)'` and `max_price` using pattern `r'under\s+\$?(\d+(?:\.\d{2})?)'`. Everything else becomes the description.

**What moves through the session:** 
- `query` → `parsed` (extracted description, size, max_price)
- `parsed` → `search_results` (list of matching listings)
- `search_results` → `selected_item` (first result)
- `selected_item` + `wardrobe` → `outfit_suggestion`
- `outfit_suggestion` + `selected_item` → `fit_card`
- If `search_results` is empty, set `error` and stop before calling suggest_outfit

---

## Sample Run

**One full query**

```
$ python agent.py

=== A query the data can match ===
  found:    Y2K Baby Tee — Butterfly Print — $18.0 on depop
  outfit:   Here are two outfit combinations featuring the Y2K butterfly baby tee and pieces from your wardrobe, styled for different vibes and occasions:

### Outfit 1: Off-Duty Model Streetwear (Casual Day Out / Coffee Run)
* **Vibe:** Relaxed, effortless, and nostalgic 2000s energy.
* **Top:** Y2K Butterfly Baby Tee (white/pink/purple)
* **Bottoms:** Baggy straight-leg jeans (dark wash)
* **Outerwear:** Black cropped zip hoodie (worn open or draped over the shoulders)
* **Shoes:** Chunky white sneakers
* **Accessories:** Black crossbody bag

### Outfit 2: Edgy Retro-Chic (Dinner with Friends / Concert)
* **Vibe:** Cool-girl contrast, mixing sweet 90s/Y2K elements with tough textures.
* **Top:** Y2K Butterfly Baby Tee (white/pink/purple)
* **Bottoms:** Wide-leg khaki trousers
* **Outerwear:** Vintage black denim jacket
* **Shoes:** Black combat boots
* **Accessories:** Brown leather belt & Black crossbody bag

  fit card: Channeling peak off-duty model energy with this 2000s butterfly baby tee 🦋✨ Style it with baggy denim for a casual coffee run or wide-leg trousers and combat boots for an edgy retro-chic night out. Grab this Y2K gem in excellent condition for just $18.0 on Depop before she's gone! 🛍️

=== A query it can't ===
  stopped: No items found matching 'designer ballgown $5'. Try different keywords, a larger size range, or increase your budget.
  fit_card is None — it should still be None here
```

**The three tools, tested one at a time**

```
$ python -c "from tools import search_listings; results = search_listings('graphic tee', max_price=30); print(f'Found {len(results)} results'); [print(f'{r[\"title\"]} - ${r[\"price\"]}') for r in results[:2]]"
Found 6 results
Y2K Baby Tee — Butterfly Print - $18.0
Graphic Tee — 2003 Tour Bootleg Style - $24.0
```

```
$ python -c "from tools import suggest_outfit; from utils.data_loader import get_example_wardrobe, load_listings; print(suggest_outfit(load_listings()[0], get_example_wardrobe())[:100])"
Here are two outfit combinations featuring the Vintage Levi's 501 Jeans and pieces from your...
```

```
$ python -c "from tools import create_fit_card; from utils.data_loader import load_listings; print(create_fit_card('white sneakers and a grey sweatshirt', load_listings()[0])[:80])"
Found the perfect pair of vintage Levi's 501s in a medium wash! The light fading...
```

---

## How I Used AI

**Moment 1 — Defining the tool specs**

- *What I asked for:* Help me understand what each tool should return, with specific types and what happens when there's no data. I wasn't sure if search_listings should return None or an empty list when nothing matches.
- *What came back:* Claude suggested that search_listings should return an empty list (not None), and that suggest_outfit should return general styling advice when the wardrobe is empty. Claude also clarified that create_fit_card should describe what "different every time" means for the model output.
- *What I changed:* I updated the Tool Inventory section in the README with specific return types (e.g., "a list of listing dicts, each with id, title, description...") and made sure my implementation matched those specs exactly. This prevented a lot of debugging later.

**Moment 2 — Building the planning loop**

- *What I asked for:* How should I parse the query to extract size and price? Should I use regex, string splitting, or ask the model?
- *What came back:* Claude suggested regex as the clearest approach and provided example patterns for extracting "size M" and "under $30" from the query string.
- *What I changed:* I used regex to extract both fields, then removed those parts from the description. This let me pass clean data to search_listings instead of having it search for "size M" as keywords.

<!-- ═══════════════════════ UNIT 4 — THE TEST ═══════════════════════

     Don't fill these in during unit 3.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Run Log — Before

<!-- Five criteria, five tries each, in this exact format.

     Five, because your criteria are written out of five. Mark each try PASS
     or FAIL, count the passes, and read that count against your target — a
     row targeting 4 of 5 with three PASS cells is MISSED (3/5).

     `python run_eval.py --label before` runs everything and writes the table
     into results/. Paste it here and fill in the verdicts. -->

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |  |  |
| 2.  |  |  |  |  |  |  |  |
| 3.  |  |  |  |  |  |  |  |
| 4.  |  |  |  |  |  |  |  |
| 5.  |  |  |  |  |  |  |  |

**Real output from one try**, pasted as text, naming the file and function
that produced it:

```

```

---

## Verdicts and Diagnoses

<!-- MET or MISSED per criterion against LAST UNIT's target, plus a sentence on
     how you decided.

     Then, for every miss: which of the four places it happened — a tool, the
     loop's branch, the session, or the model's output — AND the mechanism.

     Not a diagnosis:  "The fit card was bad."
     A diagnosis:      "The fit card criterion missed on 2 of 5 items. Both had
                        an empty brand field. My prompt puts the brand in the
                        first sentence, so the card opened with a blank and read
                        like a fragment. The tool worked; the prompt assumed a
                        field that isn't always there."

     Look for a pattern. Three misses on the same tool is one problem, not
     three. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 |  |  |  |  |
| 2 |  |  |  |  |
| 3 |  |  |  |  |
| 4 |  |  |  |  |
| 5 |  |  |  |  |

**Diagnoses**



---

## Loop Trace

<!-- One full run, printed step by step, with the MCP call visible in it.

     `python app.py ask '...' --trace` once you've added the trace.step()
     calls in Milestone 2.

     Worth pasting BOTH the happy path and the empty-search path. The empty
     one should be visibly shorter, because it stops. If your two traces are
     the same length, your branch isn't working — and this is the fastest way
     anyone will ever find that out. -->

**Happy path**

```

```

**Empty search**

```

```

**On the MCP move:** <!-- what changed in your code, and whether anything
behaved differently afterwards. If the rewire didn't work, say exactly where it
broke — the error text and the last thing that worked. That earns the point in
full. -->



---

## The Improvement

<!-- What you changed, why your diagnosis pointed at it, and the after-run in
     the same table format. One change, measured properly.

     `python run_eval.py --label after` -->

**What I changed:**

**Which failure it was meant to fix:**

### Run Log — After

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |  |  |
| 2.  |  |  |  |  |  |  |  |
| 3.  |  |  |  |  |  |  |  |
| 4.  |  |  |  |  |  |  |  |
| 5.  |  |  |  |  |  |  |  |

**Did it help, and how do I know:**

<!-- If it made things worse, say that. Honestly reported, that earns full
     credit and is more interesting than one that worked. -->



---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped where
     you did. "I ran out of time" is fine if it's true. Pretending nothing is
     left is not. -->



<!-- ═════════════════════════════════════════════════════════════════════

     SUBMISSION CHECKLIST — unit 3

       [ ] criteria.md has five numbered criteria, each with a target
       [ ] Each criterion has a reason underneath it
       [ ] All five unit 3 sections above have real content
       [ ] Tool Inventory: all three tools, inputs WITH TYPES, a specific
           return value, and the empty case
       [ ] Planning Loop names the branch rule and agent.py::run_agent
       [ ] Sample Run: one full query plus the three per-tool tests, as text
       [ ] At least four new commits
       [ ] Repository URL submitted — WRITE IT DOWN, you submit the same one
           next unit

     SUBMISSION CHECKLIST — unit 4

       [ ] mcp_server.py exists with one tool registered
           (or a written record of exactly where the rewire broke)
       [ ] Run Log — Before, five criteria, five tries each
       [ ] Real output pasted underneath, naming file and function
       [ ] A verdict on every criterion
       [ ] A diagnosis for every miss, naming a place AND a mechanism
       [ ] Loop Trace, with the MCP call visible in it
       [ ] All three failure modes triggered and handled
       [ ] One improvement, with Run Log — After in the same format
       [ ] What's Still Broken
       [ ] At least four new commits
       [ ] The SAME repository URL as last unit

     Do not delete and recreate this repository. Your commit history is what
     shows your criteria existed before your results did.
     ═════════════════════════════════════════════════════════════════════ -->

---

📖 **How to run this project: [RUNNING.md](RUNNING.md)**
