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
```
## Run Log — Before

<!-- Five criteria, five tries each, in this exact format.

     Five, because your criteria are written out of five. Mark each try PASS
     or FAIL, count the passes, and read that count against your target — a
     row targeting 4 of 5 with three PASS cells is MISSED (3/5).

     `python run_eval.py --label before` runs everything and writes the table
     into results/. Paste it here and fill in the verdicts. -->

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1. Matching query completes all three tools | At least 4 of 5 | PASS | PASS | PASS | PASS | PASS | MET |
| 2. Impossible query stops before the second tool | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET |
| 3. Selected listing ID matches the ID received by suggest_outfit | 5 of 5 | FAIL | FAIL | FAIL | FAIL | FAIL | MISSED |
| 4. Fit card includes the correct price and platform | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET |
| 5. Search results respect size and maximum price | At least 4 of 5 | FAIL | FAIL | FAIL | FAIL | FAIL | MISSED |



**Real output from one try**, pasted as text, naming the file and function
that produced it:


### Criterion 1 — Matching query completes all three tools
Source: results/run_2026-10-11_1059_before.md
Produced by: run_eval.py::main
Agent loop: agent.py::run_agent

Try 1
- stopped early: no
- selected_item: Y2K Baby Tee — Butterfly Print ($18.0, depop)
- search_results: 10

Outfit suggestion:

```
Here are two outfit combinations built around your new Y2K butterfly baby tee, utilizing pieces straight from your wardrobe:

### Outfit 1: Streetwear Contrast (Y2K Meets Grunge)
*This look leans into the classic 2000s streetwear aesthetic by balancing the ultra-feminine, pastel butterfly print with dark, baggy denim and chunky footwear.*

* **Top:** Y2K Butterfly Baby Tee (New Item)
* **Bottoms:** Baggy straight-leg jeans, dark wash
* **Outerwear:** Vintage black denim jacket (worn casually off the shoulders or open)
* **Shoes:** Chunky white sneakers
* **Accessories:** Black crossbody bag

**Why it works:** The fitted, cropped silhouette of the baby tee creates a great proportion contrast with the low-slung, baggy dark-wash jeans. Tying in the chunky white sneakers pulls out the white base of the shirt, while the black denim jacket and crossbody bag ground the pastel pink and purple butterfly graphic with a cool, edgy finish.

---

### Outfit 2: Elevated Soft-Earth (Cottagecore & Minimalist Mix)
*This outfit plays on the "cottagecore" side of the tee's style tags by pairing the soft pink and purple butterfly print with warm earth tones for a relaxed, daytime look.*

* **Top:** Y2K Butterfly Baby Tee (New Item)
* **Layering option:** White ribbed tank top worn underneath (slightly peeking out for dimension, optional) *OR* the Oversized grey crewneck sweatshirt draped over the shoulders.
* **Bottoms:** Wide-leg khaki trousers secured with the Brown leather belt
* **Shoes:** Chunky white sneakers (for a modern, sporty touch) *OR* Black combat boots (to add a grunge-lite juxtaposition)
* **Accessories:** Black crossbody bag

**Why it works:** Khaki and tan are the ultimate neutral backdrop to make the pink and purple tones of the butterfly graphic pop without feeling overwhelming. Tucking the fitted baby tee into the high-waisted wide-leg trousers accented by the brown leather belt creates an effortless, flattering silhouette that bridges early-2000s pop culture with clean, modern minimalism.
```

Fit card:

```
Just scored this adorable Y2K butterfly baby tee on Depop for only $18, and I am obsessed! I styled it with baggy dark denim and chunky sneakers for the ultimate Y2K-meets-grunge streetwear moment. ✨🦋
```

Trace:

```
[1] parse_query
      in:  vintage graphic tee under $30
      out: dict with keys: description, size, max_price
[2] search_listings (via MCP)
      in:  dict with keys: description, size, max_price
      out: 10 items: Y2K Baby Tee — Butterfly Print, Graphic Tee — 2003 Tour Bootleg Style, Vintage Band Tee — Faded Grey … +7 more
[3] select_item
      in:  10 items: Y2K Baby Tee — Butterfly Print, Graphic Tee — 2003 Tour Bootleg Style, Vintage Band Tee — Faded Grey … +7 more
      out: Y2K Baby Tee — Butterfly Print ($18.0, depop)
[4] suggest_outfit
      in:  dict with keys: selected_item, wardrobe
      out: Here are two outfit combinations built around your new Y2K butterfly baby tee, utilizing pieces straight from …
[5] create_fit_card
      in:  dict with keys: outfit, selected_item
      out: Just scored this adorable Y2K butterfly baby tee on Depop for only $18, and I am obsessed! I styled it with ba…

```
### Criterion 2 — Impossible query stops early

Source file: `results/run_2026-10-11_1059_before.md`
Produced by: `run_eval.py::main`
Agent function: `agent.py::run_agent`

**Try 1**

- stopped early: yes — I couldn't find a matching listing. Try changing the description, size, or maximum price.
- selected_item: (none)
- search_results: 0

Trace:

[1] parse_query
      in: designer ballgown size XXS under $5
      out: dict with keys: description, size, max_price
[2] search_listings (via MCP)
      in: dict with keys: description, size, max_price
      out: [] (empty)
```
```
### Criterion 3 — Selected item state

Source file: `results/run_2026-10-11_1059_before.md`
Produced by: `run_eval.py::main`
Agent function: `agent.py::run_agent`

**Try 1**
- stopped early: no
- selected_item: Y2K Baby Tee — Butterfly Print ($18.0, depop)
- search_results: 10

Outfit suggestion:

```
Here are two outfit combinations built around your new Y2K butterfly baby tee, using pieces straight from your wardrobe:

### Outfit 1: Streetwear Contrast (Y2K Meets Grunge)
This look balances the hyper-feminine, nostalgic energy of the baby tee with heavier, edgier streetwear pieces for a cool, effortless contrast.

*   **Top:** Y2K Butterfly Baby Tee (New Item)
*   **Bottoms:** Baggy straight-leg jeans (dark wash)
*   **Outerwear:** Vintage black denim jacket
*   **Shoes:** Chunky white sneakers
*   **Accessories:** Black crossbody bag

**Why it works:** The fitted, cropped cut of the baby tee balances the oversized, low-slung feel of the dark wash baggy jeans, nailing that classic 2000s silhouette. Throwing the vintage black denim jacket over top keeps the color palette grounded, while the chunky white sneakers tie into the white details of the butterfly print and add a fresh, modern streetwear finish. 

---

### Outfit 2: Casual Earthy Minimal (Soft & Grounded)
This look leans into the softer cottagecore and pastel undertones of the baby tee's pink and purple graphic, pairing them with warm neutrals for an easy everyday outfit.

*   **Top:** Y2K Butterfly Baby Tee (New Item)
*   **Bottoms:** Wide-leg khaki trousers
*   **Layering (Optional):** Oversized grey crewneck sweatshirt (carried or worn loosely over the shoulders)
*   **Shoes:** Chunky white sneakers
*   **Accessories:** Brown leather belt and Black crossbody bag

**Why it works:** Khaki and tan act as a great neutral base that lets the pink and purple tones of the butterfly graphic pop without feeling overwhelming. Tucking the baby tee into the wide-leg trousers (accented with the brown leather belt) creates a flattering high-waisted shape. If it gets chilly, tossing the oversized grey crewneck over your shoulders adds a cozy, textured dimension to the clean lines of the trousers.
```

Fit card:

```
Obsessed with this $18 Y2K butterfly baby tee I just scored on Depop! 🦋 Whether I’m styling it with baggy denim for that ultimate grunge-meets-sweet streetwear contrast or dressing it down with khaki trousers for an earthy casual fit, this little top brings all the nostalgic vibes. Run, don't walk! ✨
```

Trace:

```
[1] parse_query
      in:  vintage graphic tee under $30
      out: dict with keys: description, size, max_price
[2] search_listings (via MCP)
      in:  dict with keys: description, size, max_price
      out: 10 items: Y2K Baby Tee — Butterfly Print, Graphic Tee — 2003 Tour Bootleg Style, Vintage Band Tee — Faded Grey … +7 more
[3] select_item
      in:  10 items: Y2K Baby Tee — Butterfly Print, Graphic Tee — 2003 Tour Bootleg Style, Vintage Band Tee — Faded Grey … +7 more
      out: Y2K Baby Tee — Butterfly Print ($18.0, depop)
[4] suggest_outfit
      in:  dict with keys: selected_item, wardrobe
      out: Here are two outfit combinations built around your new Y2K butterfly baby tee, using pieces straight from your…
[5] create_fit_card
      in:  dict with keys: outfit, selected_item
      out: Obsessed with this $18 Y2K butterfly baby tee I just scored on Depop! 🦋 Whether I’m styling it with baggy deni…
```

### Criterion 4 — Fit card facts
Source file: results/run_2026-10-11_1059_before.md
Produced by: run_eval.py::main
Agent function: agent.py::run_agent

**Try 1**

- stopped early: no
- selected_item: Y2K Baby Tee — Butterfly Print ($18.0, depop)
- search_results: 10

Outfit suggestion:

```
Here are two outfit combinations built around your new Y2K butterfly baby tee, using pieces straight from your wardrobe:

### Outfit 1: Streetwear Contrast (Y2K Meets Grunge)
*This look leans into the classic 2000s streetwear aesthetic by balancing the hyper-feminine, fitted butterfly tee with ultra-baggy denim and edgy accessories.*

*   **Top:** Y2K Butterfly Baby Tee
*   **Bottoms:** Baggy straight-leg jeans (dark wash)
*   **Outerwear:** Black cropped zip hoodie (worn open or carried)
*   **Shoes:** Chunky white sneakers
*   **Accessories:** Black crossbody bag

**Why it works:** The fitted crop of the baby tee creates a great silhouette contrast against the low-slung, baggy dark-wash jeans. Tying the black cropped zip hoodie around your waist or throwing it on adds that effortless Y2K streetwear edge, while the chunky white sneakers tie the whole casual, retro vibe together.

---

### Outfit 2: Elevated Retro-Casual (Soft Earth Tones)
*This look blends the playful cottagecore and pastel elements of the butterfly tee with neutral earth tones for a balanced, everyday outfit.*

*   **Top:** Y2K Butterfly Baby Tee
*   **Bottoms:** Wide-leg khaki trousers
*   **Outerwear:** Vintage black denim jacket
*   **Shoes:** Black combat boots
*   **Accessories:** Brown leather belt, Black crossbody bag

**Why it works:** Pairing the pastel pinks and purples of the butterfly graphic with neutral khaki wide-leg trousers grounds the outfit and makes it feel a bit more mature and wearable for everyday. Tucking the baby tee in with the brown leather belt defines your waist, and throwing on the vintage black denim jacket and black combat boots adds a touch of grunge to offset the sweetness of the tee.
```

Fit card:

```
Obsessed with this Y2K butterfly baby tee I just scored for only $18.0! I'm styling it two ways—either leaning into a gritty streetwear contrast with baggy denim, or keeping it retro-casual with wide-leg trousers. Catch it on my Depop before I change my mind and keep it! 🦋✨
```

Trace:

```
[1] parse_query
      in:  vintage graphic tee under $30
      out: dict with keys: description, size, max_price
[2] search_listings (via MCP)
      in:  dict with keys: description, size, max_price
      out: 10 items: Y2K Baby Tee — Butterfly Print, Graphic Tee — 2003 Tour Bootleg Style, Vintage Band Tee — Faded Grey … +7 more
[3] select_item
      in:  10 items: Y2K Baby Tee — Butterfly Print, Graphic Tee — 2003 Tour Bootleg Style, Vintage Band Tee — Faded Grey … +7 more
      out: Y2K Baby Tee — Butterfly Print ($18.0, depop)
[4] suggest_outfit
      in:  dict with keys: selected_item, wardrobe
      out: Here are two outfit combinations built around your new Y2K butterfly baby tee, using pieces straight from your…
[5] create_fit_card
      in:  dict with keys: outfit, selected_item
      out: Obsessed with this Y2K butterfly baby tee I just scored for only $18.0! I'm styling it two ways—either leaning…

```
### Criterion 5 — Size and price filters

Source file: `results/run_2026-10-11_1059_before.md`
Produced by: `run_eval.py::main`
Agent function: `agent.py::run_agent`

**Try 1**

- stopped early: no
- selected_item: Y2K Baby Tee — Butterfly Print ($18.0, depop)
- search_results: 2

Outfit suggestion:

```
Here are two distinct outfit combinations built around your new Y2K butterfly baby tee, mixing the vintage-cute aesthetic of the top with pieces from your existing wardrobe:

### Outfit 1: Streetwear Contrast
*This look leans into the classic Y2K juxtaposition of a super-feminine, fitted top paired with baggy, masculine bottoms and chunky footwear.*

*   **Top:** New Y2K Butterfly Baby Tee
*   **Bottoms:** Baggy straight-leg jeans (dark wash)
*   **Outerwear:** Black cropped zip hoodie (worn open)
*   **Shoes:** Chunky white sneakers
*   **Accessories:** Black crossbody bag

**Why it works:** The dark wash and relaxed fit of the straight-leg jeans balance out the sweet, cropped, and colorful nature of the baby tee. Layering the black cropped zip hoodie on top keeps the silhouette sharp and waist-defining, while the chunky white sneakers and crossbody bag tie the streetwear elements together.

---

### Outfit 2: Casual Earthy Soft-Grunge
*This look blends the vintage Y2K vibe with earthy minimalism, creating an effortless, everyday look with a touch of edge.*

*   **Top:** New Y2K Butterfly Baby Tee
*   **Bottoms:** Wide-leg khaki trousers
*   **Outerwear:** Vintage black denim jacket
*   **Shoes:** Black combat boots
*   **Accessories:** Brown leather belt, Black crossbody bag

**Why it works:** The tan wide-leg trousers ground the pink and purple tones of the butterfly print, making the outfit feel a bit more grounded and mature. Tucking the baby tee in with the brown leather belt adds polish, and throwing on the vintage black denim jacket and black combat boots introduces just the right amount of grunge contrast to the cottagecore-adjacent tee.
```

Fit card:

```
Scored this dreamy Y2K butterfly baby tee for just $18 on Depop and I am obsessed! I styled it with baggy dark denim and chunky sneakers for the ultimate Y2K streetwear contrast. ✨🦋
```

Trace:

```
[1] parse_query
      in:  graphic tee size M under $30
      out: dict with keys: description, size, max_price
[2] search_listings (via MCP)
      in:  dict with keys: description, size, max_price
      out: 2 items: Y2K Baby Tee — Butterfly Print, Mesh Long-Sleeve Top — Black
[3] select_item
      in:  2 items: Y2K Baby Tee — Butterfly Print, Mesh Long-Sleeve Top — Black
      out: Y2K Baby Tee — Butterfly Print ($18.0, depop)
[4] suggest_outfit
      in:  dict with keys: selected_item, wardrobe
      out: Here are two distinct outfit combinations built around your new Y2K butterfly baby tee, mixing the vintage-cut…
[5] create_fit_card
      in:  dict with keys: outfit, selected_item
      out: Scored this dreamy Y2K butterfly baby tee for just $18 on Depop and I am obsessed! I styled it with baggy dark…
```


---
```
```
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
| 1 | Matching query completes all three tools and returns a fit card | ≥4/5 | MET (5/5) | All five tries completed `search_listings`, `suggest_outfit`, and `create_fit_card` and returned a fit card. |
| 2 | Empty search stops before `suggest_outfit` and tells the user what to change | 5/5 | MET (5/5) | All five tries stopped after the empty search and returned a message telling the user what to change. |
| 3 | Selected listing `id` matches the `id` received by `suggest_outfit` | 5/5 | MISSED (NOT MEASURABLE) | The selected item stayed consistent across all five tries, but the trace does not show the listing `id` at both points, so I could not verify the ID match as written. |
| 4 | Fit card includes the correct price and platform | 5/5 | MET (5/5) | All five tries kept the correct price and platform in the fit card. |
| 5 | Search results respect the requested size and maximum price | ≥4/5 | MISSED (NOT MEASURABLE) | The search returned two results in all five tries, but the log does not show the size and price of every returned listing, so I could not verify the criterion as written. 

**Diagnoses**

**Criterion 3 — MISSED (UNPROVEN).**  
The issue is in the session/trace visibility, not an observed wrong item. The criterion compares the `id` of the selected listing with the `id` received by `suggest_outfit`, but the trace does not show those values. It only shows the selected item by title, price, and platform, and summarizes the `suggest_outfit` input as `dict with keys: selected_item, wardrobe`. Because the IDs are not visible at both steps, I could not verify the criterion as written.

**Criterion 5 — MISSED (UNPROVEN).**  
The issue is in the `search_listings` trace output. The query requested size M and a maximum price of $30, and two listings were returned, but the trace only shows their titles. It does not show the size and price of every returned listing. Because those fields are not visible, I could not verify that every result met both filters.

**Pattern:** Both misses are observability problems rather than observed incorrect behavior from the agent. The test ran successfully, but the trace does not expose all the fields needed to verify Criteria 3 and 5 directly. A useful improvement would be to include the listing `id`, size, and price in the trace so these criteria can be measured from the test output.

---
```
```
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
$python app.py ask 'vintage graphic tee under $30' --trace
[1] parse_query
      in:  vintage graphic tee under $30
      out: dict with keys: description, size, max_price
[2] search_listings (via MCP)
      in:  dict with keys: description, size, max_price
      out: 10 items: Y2K Baby Tee — Butterfly Print, Graphic Tee — 2003 Tour Bootleg Style, Vintage Band Tee — Faded Grey … +7 more
[3] select_item
      in:  10 items: Y2K Baby Tee — Butterfly Print, Graphic Tee — 2003 Tour Bootleg Style, Vintage Band Tee — Faded Grey … +7 more
      out: Y2K Baby Tee — Butterfly Print ($18.0, depop)
[4] suggest_outfit
      in:  dict with keys: selected_item, wardrobe
      out: Here are two distinct outfit combinations built around your new Y2K butterfly baby tee, using pieces straight …
[5] create_fit_card
      in:  dict with keys: outfit, selected_item
      out: Just scored this adorable butterfly baby tee for only $18 on depop, and I’m so obsessed! I styled it two ways—…

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

**Empty search**

```
$python app.py ask 'gold astronaut wedding dress size XXL under $2' --trace
[1] parse_query
      in:  gold astronaut wedding dress size XXL under $2
      out: dict with keys: description, size, max_price
[2] search_listings (via MCP)
      in:  dict with keys: description, size, max_price
      out: [] (empty)

  I couldn't find a matching listing. Try changing the description, size, or maximum price.

0 model calls this session
```

**On the MCP move:** <!-- what changed in your code, and whether anything
behaved differently afterwards. If the rewire didn't work, say exactly where it
broke — the error text and the last thing that worked. That earns the point in
full. -->

I moved `search_listings` behind MCP and changed `run_agent()` to call it through `mcp_client.call_tool` instead of calling the function directly. The search results still came back in the same format, so the rest of the agent continued to work without changes. I tested a normal query after the move and the agent still completed the full path through `suggest_outfit` and `create_fit_card`.

---

## The Improvement

**What I changed:**

I updated the `_short()` function in `trace.py` so the trace shows the listing `id`, size, and price instead of only the listing title. I also made the selected item's `id` visible when it is passed to `suggest_outfit`.

**Which failure it was meant to fix:**

This change was meant to fix the observability problem I found in Criteria 3 and 5. In the before run, I could see that the agent was selecting and returning listings, but the trace did not show enough information to verify that the same listing `id` reached `suggest_outfit` or that every search result matched the requested size and maximum price.

### Run Log — After

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1. Matching query completes all three tools and returns a fit card | ≥4/5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 2. Empty search stops before `suggest_outfit` and tells the user what to change | 5/5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 3. Selected listing `id` matches the `id` received by `suggest_outfit` | 5/5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 4. Fit card includes the correct price and platform | 5/5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 5. Search results respect the requested size and maximum price | ≥4/5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |

**Evidence from the After Run**
The following trace output shows the fields that were missing in the before run.

**Criterion 3 — Selected item state**
Source file: `results/run_2026-10-11_1509_after.md`  
Agent function: `agent.py::run_agent`
**Try 1**

- stopped early: no
- selected_item: Y2K Baby Tee — Butterfly Print ($18.0, depop)
- search_results: 10

Outfit suggestion:

```
Here are two outfit combinations built around your new Y2K butterfly baby tee, using pieces straight from your wardrobe:

### Outfit 1: Streetwear Contrast
*This look leans into the Y2K streetwear vibe by pairing the ultra-feminine, fitted baby tee with heavy-duty denim and chunky sneakers for that classic 2000s contrast.*

*   **Top:** Y2K Butterfly Baby Tee (New item)
*   **Bottoms:** Baggy straight-leg jeans, dark wash
*   **Outerwear:** Black cropped zip hoodie (worn open or carried)
*   **Shoes:** Chunky white sneakers
*   **Accessories:** Black crossbody bag

**Why it works:** The fitted crop of the baby tee balances out the baggy, low-slung look of the dark wash jeans. Tying the black cropped zip hoodie around your waist or layering it over top adds depth, while the chunky white sneakers tie in the white from the tee's base color. 

***

### Outfit 2: Casual Earth-Tone Mix
*This outfit blends the playful, cottagecore-leaning butterfly print with minimalist earth tones for an easy, everyday look that feels grounded yet vintage.*

*   **Top:** Y2K Butterfly Baby Tee (New item)
*   **Bottoms:** Wide-leg khaki trousers 
*   **Outerwear:** Vintage black denim jacket
*   **Shoes:** Black combat boots
*   **Accessories:** Brown leather belt, Black crossbody bag

**Why it works:** Pairing the pink and purple butterfly tee with tan wide-leg trousers creates a nice contrast between soft pastel graphics and neutral earth tones. Tucking in the baby tee and wearing the brown leather belt pulls the waist in, while the vintage black denim jacket and black combat boots add a bit of edgy, grunge-inspired texture to the outfit.
```

Fit card:

```
Just scored this Y2K butterfly baby tee on Depop for only $18, and I am obsessed with how it fits! I styled it with baggy denim and chunky sneaks for the ultimate Y2K streetwear contrast, but it also looks so good dressed down with earth tones. Run, don't walk, to grab your next favorite vintage piece! ✨🦋
```

Trace:

```
[1] parse_query
      in:  vintage graphic tee under $30
      out: dict with keys: description, size, max_price
[2] search_listings (via MCP)
      in:  dict with keys: description, size, max_price
      out: 10 items: Y2K Baby Tee — Butterfly Print [id=lst_002, size=S/M, price=$18.0], Graphic Tee — 2003 Tour Bootleg Style [id=lst_006, size=L, price=$24.0], Vintage Band Tee — Faded Grey [id=lst_033, size=L, price=$19.0] … +7 more
[3] select_item
      in:  10 items: Y2K Baby Tee — Butterfly Print [id=lst_002, size=S/M, price=$18.0], Graphic Tee — 2003 Tour Bootleg Style [id=lst_006, size=L, price=$24.0], Vintage Band Tee — Faded Grey [id=lst_033, size=L, price=$19.0] … +7 more
      out: Y2K Baby Tee — Butterfly Print [id=lst_002, size=S/M, price=$18.0, platform=depop]
[4] suggest_outfit
      in:  selected_item=Y2K Baby Tee — Butterfly Print [id=lst_002, size=S/M, price=$18.0, platform=depop]
      out: Here are two outfit combinations built around your new Y2K butterfly baby tee, using pieces straight from your…
[5] create_fit_card
      in:  selected_item=Y2K Baby Tee — Butterfly Print [id=lst_002, size=S/M, price=$18.0, platform=depop]
      out: Just scored this Y2K butterfly baby tee on Depop for only $18, and I am obsessed with how it fits! I styled it…
```
**Criterion 5 — Size and price filters**
Source file: results/run_2026-10-11_1509_after.md
Agent function: agent.py::run_agent
**Try 1**

- stopped early: no
- selected_item: Y2K Baby Tee — Butterfly Print ($18.0, depop)
- search_results: 2

Outfit suggestion:

```
Here are two distinct outfit combinations built around your new Y2K butterfly baby tee, using pieces straight from your wardrobe:

### Outfit 1: Streetwear Contrast (Y2K Meets Edgy Grunge)
*This look leans into the classic 2000s contrast of a super-feminine top paired with baggy, relaxed bottoms and edgy footwear.*

*   **Top:** Y2K Baby Tee — Butterfly Print (layered underneath your black cropped zip hoodie left partially unzipped)
*   **Bottoms:** Baggy straight-leg jeans, dark wash (secured with the Brown leather belt)
*   **Shoes:** Chunky white sneakers
*   **Accessories:** Black crossbody bag

**Why it works:** The fitted, pastel butterfly graphic pops against the dark wash denim and the black zip-up. Wearing the hoodie open highlights the baby tee's cropped length, while the chunky sneakers and baggy jeans pull the whole Y2K streetwear vibe together.

---

### Outfit 2: Casual Earth-Tone Mix (Soft Vintage Cottagecore)
*This look pulls out the softer, vintage side of the baby tee by pairing it with warm neutrals and classic outerwear.*

*   **Top:** Y2K Baby Tee — Butterfly Print (worn solo as the statement piece)
*   **Bottoms:** Wide-leg khaki trousers (with the Brown leather belt)
*   **Outerwear:** Vintage black denim jacket (worn casually over the shoulders or on)
*   **Shoes:** Chunky white sneakers (or switch to Black combat boots if you want a grungier finish)
*   **Accessories:** Black crossbody bag

**Why it works:** Pairing the white, pink, and purple tee with the tan wide-leg trousers grounds the outfit in effortless neutrals, letting the butterfly graphic stand out. Throwing on the vintage black denim jacket adds a cool-girl textural contrast that balances the sweetness of the top.
```

Fit card:

```
Just scored this dreamy butterfly Y2K baby tee on Depop for just $18 and I am obsessed! Styled it today with baggy dark-wash jeans and a zip hoodie for the ultimate edgy streetwear look. ✨🦋 #thrifted #y2kstyle
```

Trace:

```
[1] parse_query
      in:  graphic tee size M under $30
      out: dict with keys: description, size, max_price
[2] search_listings (via MCP)
      in:  dict with keys: description, size, max_price
      out: 2 items: Y2K Baby Tee — Butterfly Print [id=lst_002, size=S/M, price=$18.0], Mesh Long-Sleeve Top — Black [id=lst_017, size=S/M, price=$15.0]
[3] select_item
      in:  2 items: Y2K Baby Tee — Butterfly Print [id=lst_002, size=S/M, price=$18.0], Mesh Long-Sleeve Top — Black [id=lst_017, size=S/M, price=$15.0]
      out: Y2K Baby Tee — Butterfly Print [id=lst_002, size=S/M, price=$18.0, platform=depop]
[4] suggest_outfit
      in:  selected_item=Y2K Baby Tee — Butterfly Print [id=lst_002, size=S/M, price=$18.0, platform=depop]
      out: Here are two distinct outfit combinations built around your new Y2K butterfly baby tee, using pieces straight …
[5] create_fit_card
      in:  selected_item=Y2K Baby Tee — Butterfly Print [id=lst_002, size=S/M, price=$18.0, platform=depop]
      out: Just scored this dreamy butterfly Y2K baby tee on Depop for just $18 and I am obsessed! Styled it today with b…
```



**Did it help, and how do I know:**

Yes. The change made Criteria 3 and 5 directly measurable from the trace. For Criterion 3, all five tries showed the same listing `id` at `select_item` and when the item was passed to `suggest_outfit`. For Criterion 5, all five tries returned two listings for the size M, under $30 query, and the trace showed that both were size S/M and priced at $18 and $15. The agent's behavior did not change; the improvement made the existing behavior visible enough to test against my criteria.



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
