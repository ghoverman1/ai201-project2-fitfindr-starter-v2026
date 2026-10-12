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

<!-- Three or four sentences: what a user asks for, and what they get back. -->
## What This Does

FitFindr takes a plain-language request for a thrifted clothing item, such as a vintage graphic tee under $30. It searches a listings dataset using the requested description, size, and maximum price, then selects the best match. The agent uses the user's wardrobe to suggest outfits around that item and generates a short fit-card caption. If no listings match, it stops before the later tools and tells the user which search constraints they could loosen.


---

## Tool Inventory

<!-- Four lines per tool. This is worth 2 points and it's the single most
     common place students lose them.

     "Returns a list" earns NOTHING. The description has to say what is IN
     the list.

     The empty case isn't optional either — it's the thing your loop branches
     on, and if you don't decide it here you'll discover it as a crash in
     Milestone 5. -->

### `search_listings`

- **What it does:** Filters `data/listings.json` by price and size, scores what's
  left by keyword overlap with the description (title, description, style_tags,
  category, colors), drops anything scoring zero, and returns the best matches
  first. Doesn't call the model.
- **Inputs:** <!-- name and type each: `max_price` (float), not "a price" -->
  `description` (str) — keywords like `"vintage graphic tee"`;
  `size` (str | None) — `None` skips the size filter;
  `max_price` (float | None) — inclusive ceiling, `None` skips the price filter.
  A size matches when it equals one of the listing's size tokens, case-insensitive,
  after splitting on `/` and spaces — so `"M"` matches `"S/M"` and `"M/L"`, but
  not `"XL"` or `"US 9"`. A whole-string match also counts, so `"US 9"` matches
  `"US 9"`. "One Size" listings match any size.
- **Returns:** `list[dict]`, best match first, at most `config.SEARCH_RESULT_LIMIT`
  (10). Each dict is a full listing: `id`, `title`, `description`, `category`,
  `style_tags` (list), `size`, `condition`, `price` (float), `colors` (list),
  `brand` (str or None — usually None), `platform`.
- **When it has nothing:** Returns `[]` — an empty list, not `None` and not an
  exception. The loop checks for this and stops before calling `suggest_outfit`.

### `suggest_outfit`

- **What it does:** Asks the model for one or two outfits built around the
  thrifted item, naming pieces the user already owns.
- **Inputs:** `new_item` (dict) — one listing dict from `search_listings`
  (the loop passes the top result); `wardrobe` (dict) — `{"items": [...]}` in the
  format of `data/wardrobe_schema.json`, where each item has `name`, `category`,
  `colors`, `style_tags`, `notes`.
- **Returns:** `str` — a non-empty block of outfit suggestions, e.g. "Pair the
  butterfly baby tee with your baggy dark-wash jeans and white sneakers…".
- **When it has nothing:** If `wardrobe["items"]` is empty, it asks the model for
  general styling ideas for the item instead, and still returns a non-empty
  string. It never returns `""` or raises.

### `create_fit_card`

- **What it does:** Asks the model to write a short caption someone would
  actually post about the find — reads like a real post, not a product
  description.
- **Inputs:** `outfit` (str) — the suggestion string from `suggest_outfit`;
  `new_item` (dict) — the same listing dict passed to `suggest_outfit`.
- **Returns:** `str` — a 2–4 sentence caption that mentions the item, its price,
  and its platform once each and is specific about the vibe. Doesn't rely on
  `brand`, since it's usually None.
- **When it has nothing:** If `outfit` is empty or only whitespace, it returns a
  descriptive message (`"No outfit suggestion to build a fit card from."`)
  without calling the model. It never raises.

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

**Branch rule:** If `search_listings` returns an empty list, put a message in
`session["error"]` telling the user what to loosen (raise the price, drop the
size, or use broader keywords) and return the session without calling
`suggest_outfit` or `create_fit_card`. Otherwise, take the first (best-scoring)
result as `session["selected_item"]`, pass it to `suggest_outfit`, then pass that
outfit and the same item to `create_fit_card`.

**Where it lives:** `agent.py::run_agent`

**How the query is parsed:** Regex extracts `size` and `max_price` from the query. Those phrases are removed, and the remaining text becomes the search description.
`max_price`, `size X` becomes `size`, and whatever is left after removing those
two phrases is the `description`. A query with neither phrase is searched as-is. 

**What moves through the session:** In order: `query` → `parsed`
(`description`, `size`, `max_price`) → `search_results` → `selected_item` →
`outfit_suggestion` → `fit_card`. `wardrobe` is set at the start and read by
`suggest_outfit`. `error` is set only when the run stops early, and then
`selected_item`, `outfit_suggestion`, and `fit_card` stay `None`.

---

# Sample Run

<!-- Two things go here.

     1. One FULL query and its output, pasted as text.
     2. Your three per-tool terminal tests — the command and what it printed. -->

**One full query**

```text
$ python app.py ask '...'

[Fill this in after Milestone 5, once the planning loop is wired together.]
```
(.venv) georgehoverman@Georges-MacBook-Pro ai201-project2-fitfindr-starter-v2026 % python agent.py
=== A query the data can match ===
  found:    Y2K Baby Tee — Butterfly Print — $18.0 on depop
  outfit:   **Outfit 1 (Streetwear Y2K):**
Pair the Y2K Baby Tee — Butterfly Print with the baggy straight-leg jeans, dark wash. Layer the vintage black denim jacket on top. Finish with the chunky white sneakers and the black crossbody bag for an effortless, high-contrast look that balances the cropped fit with relaxed denim.

**Outfit 2 (Edgy Grunge Mix):**
Tuck the Y2K Baby Tee — Butterfly Print into the wide-leg khaki trousers, secured with the brown leather belt. Throw the oversized grey crewneck sweatshirt over your shoulders or wear it unzipped with the black cropped zip hoodie underneath, and ground the outfit using the black combat boots for a cool, textural contrast.
  fit card: Found this little butterfly print baby tee scrolling on depop for just eighteen bucks and I am obsessed. It’s got that tiny early-2000s crop that looks so good thrown on with baggy jeans and chunky sneakers. Total main character energy for under twenty dollars.

=== A query it can't ===
  stopped: No listings matched. Try raising your maximum price, removing the size filter, or using broader search terms.
  fit_card is None — it should still be None here

The second one should stop before the fit card. If both paths look the same,
the branch isn't doing anything yet.
(.venv) georgehoverman@Georges-MacBook-Pro ai201-project2-fitfindr-starter-v2026 % python app.py ask 'vintage graphic tee under $30'

  Found:    Y2K Baby Tee — Butterfly Print — $18.0 on depop

  Outfit:   **Outfit 1 (Streetwear Y2K):**
Pair the Y2K Baby Tee — Butterfly Print with the baggy straight-leg jeans, dark wash. Layer the vintage black denim jacket on top. Finish with the chunky white sneakers and the black crossbody bag for an effortless, high-contrast look that balances the cropped fit with relaxed denim.

**Outfit 2 (Edgy Grunge Mix):**
Tuck the Y2K Baby Tee — Butterfly Print into the wide-leg khaki trousers, secured with the brown leather belt. Throw the oversized grey crewneck sweatshirt over your shoulders or wear it unzipped with the black cropped zip hoodie underneath, and ground the outfit using the black combat boots for a cool, textural contrast.

  Fit card: Found this little butterfly print baby tee scrolling on depop for just eighteen bucks and I am obsessed. It’s got that tiny early-2000s crop that looks so good thrown on with baggy jeans and chunky sneakers. Total main character energy for under twenty dollars.

0 model calls this session, 2 served from cache


**The three tools, tested one at a time**

```text
$ python -c "from tools import search_listings; print(search_listings('graphic tee', max_price=30))"

[{'id': 'lst_002', 'title': 'Y2K Baby Tee — Butterfly Print', 'description': 'Super cute early 2000s baby tee with butterfly graphic. Fitted crop length. Tag says medium but fits like a small.', 'category': 'tops', 'style_tags': ['y2k', 'vintage', 'graphic tee', 'cottagecore'], 'size': 'S/M', 'condition': 'excellent', 'price': 18.0, 'colors': ['white', 'pink', 'purple'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_006', 'title': 'Graphic Tee — 2003 Tour Bootleg Style', 'description': 'Vintage-style bootleg tee with faded graphic. Slightly boxy fit. 100% cotton, soft and worn-in.', 'category': 'tops', 'style_tags': ['graphic tee', 'vintage', 'grunge', 'streetwear', 'band tee'], 'size': 'L', 'condition': 'good', 'price': 24.0, 'colors': ['black'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_017', 'title': 'Mesh Long-Sleeve Top — Black', 'description': 'Sheer black mesh long-sleeve. Great for layering under a graphic tee or over a bralette. Stretchy material, fits true to size.', 'category': 'tops', 'style_tags': ['y2k', 'grunge', 'goth', 'layering'], 'size': 'S/M', 'condition': 'excellent', 'price': 15.0, 'colors': ['black'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_033', 'title': 'Vintage Band Tee — Faded Grey', 'description': 'Faded grey band-style tee with distressed graphic. Crew neck. Fits boxy. Well-loved but no holes or major damage.', 'category': 'tops', 'style_tags': ['vintage', 'grunge', 'band tee', 'graphic tee', 'streetwear'], 'size': 'L', 'condition': 'fair', 'price': 19.0, 'colors': ['grey', 'charcoal'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_011', 'title': 'Low-Rise Cargo Pants — Khaki', 'description': 'Y2K era low-rise cargo pants. Lots of pockets. Khaki color, slightly distressed at the hems. Great for layering with a long tee.', 'category': 'bottoms', 'style_tags': ['y2k', 'cargo', '2000s', 'streetwear'], 'size': 'W29', 'condition': 'fair', 'price': 27.0, 'colors': ['khaki', 'tan'], 'brand': None, 'platform': 'poshmark'}, {'id': 'lst_015', 'title': 'Vintage Graphic Hoodie — Faded Black', 'description': 'Faded black pullover hoodie with barely-visible vintage graphic on the chest. Cozy interior. Some pilling but adds to the worn-in look.', 'category': 'tops', 'style_tags': ['vintage', 'grunge', 'graphic', 'streetwear'], 'size': 'L', 'condition': 'fair', 'price': 26.0, 'colors': ['black', 'charcoal'], 'brand': None, 'platform': 'depop'}]
```

```text
$ python -c "from tools import suggest_outfit; from utils.data_loader import get_example_wardrobe, load_listings; print(suggest_outfit(load_listings()[0], get_example_wardrobe()))"

**Look 1: Casual Streetwear**
Pair the vintage Levi's 501s with the white ribbed tank top tucked in. Layer the oversized grey crewneck sweatshirt on top. Finish with chunky white sneakers and the black crossbody bag.

**Look 2: Edgy Layers**
Wear the vintage Levi's 501s with the brown leather belt. Add the black cropped zip hoodie, topped with the vintage black denim jacket. Complete the outfit with the black combat boots and the black crossbody bag.
```

```text
$ python -c "from tools import create_fit_card; from utils.data_loader import load_listings; print(create_fit_card('jeans and white sneakers', load_listings()[0]))"

Scored these vintage Levi's 501 Jeans on depop today for just $38.00 and they honestly fit like a dream. The wash is that exact shade of blue I've been hunting for, complete with that worn-in knee fading. Just threw them on with beat-up white sneakers and called it a day.
```

---

## How I Used AI

<!-- Two specific moments. What you asked, what came back, what you changed.

     "I used Claude to help me code" is not enough.

     "I gave Claude my search_listings spec. It returned None on no match
     instead of an empty list, so I changed it" is the level we want. -->

**Moment 1**

- **What I asked for:** I wrote the Tool Inventory and pseudocode for the three tools first, then asked Claude to help turn that structure into working Python in `tools.py`.
- **What came back:** It produced implementations for `search_listings`, `suggest_outfit`, and `create_fit_card`, including keyword scoring, size matching, model prompts, and empty-case handling.
- **What I changed:** I checked the code against my own Tool Inventory, kept the return shapes and branch behavior I had specified, and tested each tool separately from the terminal, including the empty-wardrobe case.

**Moment 2**

- **What I asked for:** I asked ChatGPT to help me reason through how the planning loop and session state should work in `agent.py`.
- **What came back:** It explained the branch structure and why each tool result should be stored in the session and then read back out by the next step instead of being passed only through temporary variables.
- **What I changed:** I implemented the loop so an empty search stops early with a useful message, while a successful search stores the selected item in the session and carries it through `suggest_outfit` and `create_fit_card`. I then tested both the successful and no-results paths.

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

[1] search_listings (via MCP)
      in:  description='vintage graphic tee', size=None, max_price=30.0
      out: 10 items: Y2K Baby Tee — Butterfly Print, Graphic Tee — 2003 Tour Bootleg Style, Vintage Band Tee — Faded Grey … +7 more
      →    branch: results found, continuing
[2] suggest_outfit
      in:  new_item=lst_002, wardrobe_items=10
      out: **Outfit 1 (Streetwear Y2K):** Pair the Y2K Baby Tee — Butterfly Print with the baggy straight-leg jeans, dark…
[3] create_fit_card
      in:  new_item=lst_002
      out: Found this little butterfly print baby tee scrolling on depop for just eighteen bucks and I am obsessed. It’s …

**Empty search**

```
[1] search_listings (via MCP)
      in:  description='designer ballgown', size='XXS', max_price=5.0
      out: [] (empty)
      →    branch: empty, stopping

  No listings matched. Try raising your maximum price, removing the size filter, or using broader search terms.

```

**On the MCP move:** <!-- what changed in your code, and whether anything
behaved differently afterwards. If the rewire didn't work, say exactly where it
broke — the error text and the last thing that worked. That earns the point in
full. -->
I moved `search_listings` from a direct Python call to an MCP tool call through `mcp_client.call_tool`. The inputs and returned listings stayed the same, and both the normal query and empty-search branch behaved the same after the change.


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
