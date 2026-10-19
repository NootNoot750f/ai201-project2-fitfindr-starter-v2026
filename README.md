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

**How the query is parsed:** Regex, in `agent.py::run_agent`. Size comes from `r'size\s+([a-zA-Z0-9/\s]+?)(?:\s+under|\s*$)'` and max_price from `r'under\s+\$?(\d+(?:\.\d{2})?)'`, both case-insensitive. The matched size and price text is then removed from the query with `re.sub`, and what's left (stripped) is the description.

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
$ python -c "from tools import search_listings; r = search_listings('graphic tee', max_price=30); print(len(r), 'results'); print([(x['title'], x['price']) for x in r[:2]])"
6 results
[('Y2K Baby Tee — Butterfly Print', 18.0), ('Graphic Tee — 2003 Tour Bootleg Style', 24.0)]
```

```
$ python -c "from tools import suggest_outfit; from utils.data_loader import get_example_wardrobe, load_listings; print(suggest_outfit(load_listings()[0], get_example_wardrobe()))"
Here are two outfit combinations featuring your new Vintage Levi's 501 jeans and pieces from your wardrobe, styled for different vibes and occasions:

### Outfit 1: Casual Coffee Run (Off-Duty Cool Vibe)
* **New Item:** Vintage Levi's 501 Jeans (Medium Wash)
* **Top:** White ribbed tank top
* **Outerwear:** Vintage black denim jacket
* **Shoes:** Chunky white sneakers
* **Accessories:** Black crossbody bag, Brown leather belt
...
```

(Output abridged after the first outfit. The full response is two outfits.)

```
$ python -c "from tools import create_fit_card; from utils.data_loader import load_listings; print(create_fit_card('white sneakers and a grey sweatshirt', load_listings()[0]))"
Nothing beats the effortless, off-duty vibe of a true vintage classic. 👖✨ These perfectly faded medium wash Levi's 501s give off the ultimate effortless 90s aesthetic—just pair them with a cozy grey sweatshirt and fresh white sneakers for that ultimate cool-girl look. Grab them in good condition for just $38.0 over on my depop shop before they're gone!
```

---

## How I Used AI

**Moment 1 — Drafting the Tool Inventory**

- *What I asked for:* I asked Claude to help me spec the three tools: what each takes, what it returns, and what it returns when there's nothing. I asked it to base this on the listing and wardrobe fields in the data.
- *What came back:* Claude drafted the four lines per tool, including `search_listings` returning a list of listing dicts and an empty list when nothing matches, and `suggest_outfit` falling back to general styling advice when the wardrobe is empty.
- *What I changed:* I asked Claude to draft the Tool Inventory from the listing and wardrobe fields. I checked it against the data and kept it as written.

**Moment 2 — Choosing how to parse the query**

- *What I asked for:* How should the agent pull size and max price out of a query like "vintage graphic tee under $30, size M"? Regex, string splitting, or asking the model?
- *What came back:* Claude recommended regex for these consistent formats and wrote the patterns into `run_agent`.
- *What I changed:* I chose regex. When I ran the "designer ballgown size XXS under $5" query, the leftover text became `designer ballgown $5`, so the price stayed in the search keywords.
  I left the size bug as a known issue in Unit 3. I found its cause in the Unit 4 trace and fixed it there.

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
| 1. Full three-tool run returns a fit card | 4 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 2. Empty search stops before tool 2 | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 3. Selected item is the first search result | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 4. Fit card has price, platform, length (revised) | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 5. Price ceiling is enforced | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |

**Real output from one try** (criterion 2, try 1), from `results/run_2026-10-18_2342_before.md`, produced by `run_eval.py::run_once` calling `agent.py::run_agent`:

```
[1] search_listings (via MCP)
      in:  description='designer ballgown $5', size='XXS', max_price=5.0
      out: [] (empty)
      →    branch: empty, stopping
  try 1: stopped early — No items found matching 'designer ballgown $5'. Try differen
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
| 1 | Full three-tool run returns a fit card | 4 of 5 | MET (5/5) | Five of five tries produced a fit card. |
| 2 | Empty search stops before tool 2 | 5 of 5 | MET (5/5) | Five of five stopped at the branch, with a message. The message echoes the bad `$5` keyword. |
| 3 | Selected item is the first search result | 5 of 5 | MET (5/5) | The trace's first result title matches `selected_item` in all five tries. I compared titles, not IDs. |
| 4 | Fit card has price, platform, length (revised) | 5 of 5 | MET (5/5) | All five cards name $18 or $19 and Depop, and run three to four sentences. I counted by eye. |
| 5 | Price ceiling is enforced | 5 of 5 | MET (5/5) | The selected item is $18 in all five tries. The other results' prices aren't printed, so "all results" isn't confirmed by this output. |

**Diagnoses**

No criterion was missed, so there are no misses to diagnose. The more useful finding is a defect the criteria didn't catch.

- **The search keyword has a `$5` in it.** Place: the loop's parse step, `agent.py::run_agent`. Mechanism: the size cleanup pattern `size\s+[a-zA-Z0-9/\s]+` also consumes the word "under" after "XXS", so the price cleanup finds nothing and "$5" stays in the keywords. Criterion 2 still passed because the branch works. The bug only shows in the message text, which that criterion doesn't check.
- **My targets were low.** Criterion 1 only checks that a card exists, not what it says. Criterion 2 checks the stop and that a message exists, not what the message names. I'd tighten criterion 2 first, to require the message to echo the query as the user typed it.

---

## Loop Trace

<!-- One full run, printed step by step, with the MCP call visible in it.

     `python app.py ask '...' --trace` once you've added the trace.step()
     calls in Milestone 2.

     Worth pasting BOTH the happy path and the empty-search path. The empty
     one should be visibly shorter, because it stops. If your two traces are
     the same length, your branch isn't working — and this is the fastest way
     anyone will ever find that out. -->

**Happy path** (`python app.py ask 'band tee under $25' --trace`)

```
[1] search_listings (via MCP)
      in:  description='band tee', size=None, max_price=25.0
      out: 4 items: Vintage Band Tee — Faded Grey, Y2K Baby Tee — Butterfly Print, Graphic Tee — 2003 Tour Bootleg Style … +1 more
      →    branch: results found, continuing
[2] suggest_outfit
      in:  Vintage Band Tee — Faded Grey ($19.0, depop)
      out: Here are two outfit combinations using the vintage faded grey band tee and pieces from your wardrobe, styled f…
[3] create_fit_card
      in:  Vintage Band Tee — Faded Grey ($19.0, depop)
      out: Achieve peak model-off-duty energy with this perfectly worn-in vintage faded grey band tee. 🎸 It's got that id…
```

**Empty search** (`python app.py ask 'designer ballgown size XXS under $5' --trace`, after the fix in the Improvement section)

```
[1] search_listings (via MCP)
      in:  description='designer ballgown', size='XXS', max_price=5.0
      out: [] (empty)
      →    branch: empty, stopping

  No items found matching 'designer ballgown'. Try different keywords, a larger size range, or increase your budget.

0 model calls this session
```

The empty trace is shorter because the loop stops before `suggest_outfit`. Before the fix, the same query gave `description='designer ballgown $5'`, which is the bug described in Diagnoses.

**On the MCP move:** I moved `search_listings` behind an MCP server. `mcp_server.py` registers it with a description and typed inputs, and `agent.py::run_agent` calls it through `mcp_client.call_tool`. Nothing behaved differently: the matching and empty-search runs return the same results as the direct call, and the trace shows `search_listings (via MCP)`.



---

## Failure Modes

Each failure triggered on purpose. Output is pasted as it printed.

**1. Empty search.** `python app.py ask 'designer ballgown size XXS under $5'` (after the fix)

```
  No items found matching 'designer ballgown'. Try different keywords, a larger size range, or increase your budget.
```

The branch in `agent.py::run_agent` stops here, so `suggest_outfit` is never called.

**2. Empty wardrobe.** `python app.py ask 'denim jacket under $50' --empty-wardrobe` (abridged)

```
(running with an empty wardrobe)

  Found:    Denim Jacket — Light Wash, Cropped — $42.0 on poshmark

  Outfit:   Here are two fun, distinct ways to style your cropped light-wash denim jacket:
...
```

`tools.py::suggest_outfit` returns styling advice when the wardrobe is empty, so the user gets advice rather than a crash.

**3. Model unavailable.** I changed one character of the key and ran a new query. Before the handler was added, the error came from the catch-all in `app.py`:

```
ModelUnavailable: The model rejected your API key. Check GEMINI_API_KEY in your .env file, or create a fresh key at aistudio.google.com.
```

After adding the handler in `agent.py::run_agent`, I forced the same error with a mock, because the real key is restored:

```
error: Couldn't reach the model. The model rejected your API key. Check GEMINI_API_KEY in your .env file.
fit_card: None
```

I haven't re-run this with a broken key since adding the handler.

---

## The Improvement

<!-- What you changed, why your diagnosis pointed at it, and the after-run in
     the same table format. One change, measured properly.

     `python run_eval.py --label after` -->

**What I changed:** One line in `agent.py::run_agent`, the size cleanup pattern. It was `size\s+[a-zA-Z0-9/\s]+`, and is now `size\s+[a-zA-Z0-9/\s]+?(?=\s+under|\s*$)`. The new version stops before "under", so the price cleanup can remove "under $5" and the keywords no longer carry the dollar sign.

**Which failure it was meant to fix:** The parse bug from the Diagnoses section. The search keyword `designer ballgown $5` became `designer ballgown`. Criterion 2 didn't measure this, so the run log can't show the fix as a verdict change. The evidence is the trace and the message.

### Run Log — After

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1. Full three-tool run returns a fit card | 4 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 2. Empty search stops before tool 2 | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 3. Selected item is the first search result | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 4. Fit card has price, platform, length (revised) | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 5. Price ceiling is enforced | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |

Source: `results/run_2026-10-18_2348_after.md`.

**Did it help, and how do I know:** It fixed the parse bug, but the verdicts didn't change, because they were already passing. The after-run's empty-search try shows `description='designer ballgown'` and the message reads "No items found matching 'designer ballgown'". The before-run showed `$5` in both. I checked the after-run's fit cards by eye, the same way as before, and they still name $18 or $19 and Depop.

<!-- If it made things worse, say that. Honestly reported, that earns full
     credit and is more interesting than one that worked. -->



---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped where
     you did. "I ran out of time" is fine if it's true. Pretending nothing is
     left is not. -->

- **Criterion 5 ("all results under budget") isn't fully checked.** The run log only shows the selected item's price. The fix is to print every result's price in `run_eval.py`, or to narrow the criterion to the selected item. I stopped here because the time ran out. The criterion's wording is still as it was written, so the verdict is only for the selected item.
- **Criterion 3 compares titles, not IDs.** A duplicate title would hide a mismatch. The fix is to print the IDs.
- **Criterion 2 doesn't check the message text.** The `$5` bug only showed up in the message. The fix is to tighten the criterion so the message echoes the search as the user typed it, then rerun. I didn't do it because it's a second change.
- **Criterion 4 counts sentences by eye.** A script that counts sentences would make it reliable.
- **Model unavailable message says the key was rejected.** It doesn't say the model couldn't be reached. Tightening the wording in `generate.py` would fix that. I left it, because the message does name what to check.



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
