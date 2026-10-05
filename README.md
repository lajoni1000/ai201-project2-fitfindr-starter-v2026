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

FitFindr is an agent that helps a user find a clothing item from a set of secondhand listings and decide how to style it. A user can describe what they are looking for and optionally include a size and maximum price. The agent searches the listings, selects a matching item, suggests two outfits using pieces from the user's wardrobe, and creates a short fit card with information about the selected item. If there are no matching listings, the agent stops and tells the user what they could change in their search.

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

- **What it does:** Searches the local clothing listings for items that match the description, with optional size and maximum price filters.
- **Inputs:** `description` (str), `size` (str | None), `max_price` (float | None). Size matching is case-insensitive, and `M` can match a combined size like `S/M`.
- **Returns:** A list of complete matching listing dictionaries. Each dictionary includes `id`, `title`, `description`, `category`, `style_tags`, `size`, `condition`, `price`, `colors`, `brand`, and `platform`.
- **When it has nothing:** Returns an empty list `[]` when no listings match the search.

### `suggest_outfit`

- **What it does:** Suggests two outfits around the selected listing, using items from the user's wardrobe when available.
- **Inputs:** `new_item` (dict), `wardrobe` (dict).
- **Returns:** A non-empty string containing two outfit suggestions. When wardrobe items are available, the suggestions name pieces from the wardrobe.
- **When it has nothing:** If the wardrobe is empty, returns general outfit ideas and explains that the suggestions are general because no wardrobe items are saved.

### `create_fit_card`

- **What it does:** Creates a short social-media-style caption about the selected second-hand item and how it can be worn.
- **Inputs:** `outfit` (str), `new_item` (dict).
- **Returns:** A string containing a 2–4 sentence caption that includes the item's price in digits and the selling platform.
- **When it has nothing:** If `outfit` is empty or contains only whitespace, returns a helpful fallback message without calling the model or raising an exception.


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

**Branch rule:** If `search_listings` returns an empty list, put a message in the session and stop. Otherwise, take the first result and go to `suggest_outfit`.
**Where it lives:** `agent.py::run_agent`

**How the query is parsed:** <!-- regex, string splitting, or asking the model — say which -->
The query is parsed with regular expressions and string cleanup. The parser looks for a maximum price written as “under $X” or “under X” and for a size written as “size X.” Those parts are removed from the query, along with common introductory words such as “looking for,” and the remaining text becomes the description used by search_listings.

**What moves through the session:** <!-- which fields, in what order -->
The session starts with the original query and wardrobe. The parsed description, size, and maximum price are stored in session["parsed"]. Search results are stored in session["search_results"], and the first result is stored as session["selected_item"]. That selected item is read from the session by suggest_outfit, and its result is stored in session["outfit_suggestion"]. Finally, create_fit_card reads the outfit suggestion and selected item from the session, and the result is stored in session["fit_card"]. If no listings are found, session["error"] is set and the agent stops before the remaining tools are called.
---

## Sample Run

<!-- Two things go here.

     1. One FULL query and its output, pasted as text.
     2. Your three per-tool terminal tests — the command and what it printed. -->

**One full query**

```
python app.py ask "looking for a vintage graphic tee under `$30"

  Found:    Y2K Baby Tee — Butterfly Print — $18.0 on depop

  Outfit:   Here are two distinct outfit combinations built around your new Y2K butterfly baby tee, using pieces straight from your wardrobe:

### Outfit 1: Streetwear Contrast (Y2K Meets Grunge)
This look plays on the contrast between the fitted, feminine butterfly tee and relaxed, edgy streetwear staples. 

*   **Top:** Y2K Butterfly Baby Tee (layered over or under your white ribbed tank top for a textured neckline, if you like)
*   **Bottoms:** Baggy straight-leg jeans (dark wash)
*   **Outerwear:** Black cropped zip hoodie (worn open to show off the graphic)
*   **Shoes:** Chunky white sneakers
*   **Accessories:** Black crossbody bag

**Why it works:** The dark, baggy jeans and chunky sneakers ground the sugary-sweet pink and purple butterfly print, giving it an authentic early-2000s street style feel. Adding the black cropped zip hoodie ties the dark accents of the shoes and bag together while keeping the cropped silhouette balanced.

---

### Outfit 2: Elevated Earth Tones (Soft Cottagecore & Minimal)
This look leans into the softer, cottagecore-adjacent side of the tee by pairing it with warm neutrals for a more polished, everyday outfit.

*   **Top:** Y2K Butterfly Baby Tee
*   **Bottoms:** Wide-leg khaki trousers 
*   **Accessories (Belt):** Brown leather belt (threaded through the trousers to define the waist)
*   **Outerwear:** Vintage black denim jacket (draped over the shoulders or worn casually)
*   **Shoes:** Chunky white sneakers (or swap for black combat boots to add a tougher edge)
*   **Accessories (Bag):** Black crossbody bag

**Why it works:** Khaki and white make a classic, clean base that lets the pink and purple tones of the butterfly graphic pop. Tucking the fitted baby tee into the high-waisted wide-leg trousers creates a flattering proportion, and the brown leather belt adds a touch of vintage warmth that bridges the gap between the white tee and tan bottoms.

  Fit card: Just scored this adorable butterfly baby tee for only $18 on depop, and I’m so obsessed! I styled it two ways—first with baggy denim for that ultimate Y2K grunge look, and again with wide-leg trousers for a softer, elevated vibe. Which fit is your favorite? 🦋✨

0 model calls this session, 2 served from cache

```

**The three tools, tested one at a time**

```

python -c "from tools import search_listings; print([(x['id'], x['title'], x['size'], x['price']) for x in search_listings('graphic tee', size='M', max_price=30)])"
[('lst_002', 'Y2K Baby Tee — Butterfly Print', 'S/M', 18.0), ('lst_017', 'Mesh Long-Sleeve Top — Black', 'S/M', 15.0)]

```

```
python -c "from tools import suggest_outfit; from utils.data_loader import get_example_wardrobe, load_listings; print(suggest_outfit(load_listings()[0], get_example_wardrobe()))"
Here are two distinct outfit combinations built around your new vintage Levi's 501s, using pieces you already own:

### Outfit 1: Casual Streetwear Sporty
This look leans into the vintage streetwear vibe of the 501s by pairing them with sporty, contrasting layers and chunky sneakers. 

*   **Bottoms:** Vintage Levi's 501 Jeans (Medium Wash)
*   **Top:** White ribbed tank top (tucked in to define the waist)
*   **Outerwear:** Black cropped zip hoodie (layered open over the tank)
*   **Shoes:** Chunky white sneakers
*   **Accessories:** Black crossbody bag

**Why it works:** The fitted white tank creates a clean, minimal base that balances the relaxed straight leg of the Levi's. Throwing the black cropped zip hoodie on top plays with proportions (fitted vs. cropped), and the chunky white sneakers tie the whole streetwear aesthetic together with the denim.

---

### Outfit 2: Grunge-Infused Vintage Classic
This look highlights the vintage, slightly worn-in character of the medium wash denim by pairing it with darker, textured layers and rugged boots.

*   **Bottoms:** Vintage Levi's 501 Jeans (Medium Wash)
*   **Top:** Oversized grey crewneck sweatshirt 
*   **Outerwear:** Vintage black denim jacket (worn over the sweatshirt)
*   **Shoes:** Black combat boots
*   **Accessories:** Brown leather belt

**Why it works:** Double denim is a classic vintage move, and pairing a medium wash with a black denim jacket creates a cool contrast without being too matchy. The oversized grey crewneck adds a cozy, effortless texture underneath, while the brown leather belt and black combat boots ground the outfit with a touch of grunge edge.

```
python -c "from tools import create_fit_card; from utils.data_loader import load_listings; print(create_fit_card('jeans and white sneakers', load_listings()[0]))"
Nothing beats the effortless cool of these vintage Levi's 501 jeans, complete with that perfectly worn-in knee fading. I styled them with crisp white sneakers for the ultimate off-duty streetwear look that screams timeless casual. Grab this medium wash staple for just $38.0 over on my depop before someone else snatches them up!

```

---

## How I Used AI

<!-- Two specific moments. What you asked, what came back, what you changed.

     "I used Claude to help me code" is not enough.

     "I gave Claude my search_listings spec. It returned None on no match
     instead of an empty list, so I changed it" is the level we want. -->

**Moment 1**

- *What I asked for:* I asked ChatGPT to help me review my `search_listings` implementation and test whether the size filter worked with the listings data.
- *What came back:* We found that sizes in the data are not always stored as a single value. For example, a listing can have `S/M`, so an exact comparison with `M` would miss an item that should match.
- *What I changed:* I changed the size comparison so it is case-insensitive and splits combined sizes such as `S/M`. I then tested `search_listings` from the terminal and confirmed that a search for size `M` could return the `S/M` listing.

**Moment 2**

- *What I asked for:* I asked ChatGPT to coach me through connecting the three tools in `run_agent` while keeping the session as the source of state between the tools.
- *What came back:* We identified that after `search_listings`, the agent needed an explicit branch for an empty result instead of continuing to the outfit and fit-card tools.
- *What I changed:* I added the branch so that an empty search stores an actionable error message in the session and returns immediately. For a successful search, I store the first result in `session["selected_item"]` and use the session values for the next tool calls. I tested both paths and confirmed that the no-match path stops with `fit_card` still set to `None`.

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
