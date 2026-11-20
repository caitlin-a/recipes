---
name: add-recipe
description: Use when the user wants to add a recipe to this repo, from a Word doc, a photo (of a recipe card, handwritten note, or the finished dish), or a web link. Converts the source into the repo's standard recipe format and saves it in the right place.
---

# Add recipe

Adds one recipe to this repo in the repo's standard format. Recipes should be
as few clicks away as possible: there are no category folders. Every recipe
the user has cooked is one markdown file directly in the repo root, and its
photo goes in `assets/`. Recipes she hasn't cooked yet (`status: to_try`) live
flat in `untested/`, so nobody browsing the repo mistakes them for recipes she
recommends. When she has cooked one, change its `status` to `tried` and move
it into the repo root (its photo stays in `assets/`). Finding things is done with the YAML
tags (see step 2), not folders.

## Steps

1. **Identify the source and extract the raw content.**
   - **Word doc (.doc/.docx), text:** these aren't plain text — don't try to
     Read them directly. Convert first, e.g. `textutil -convert txt -stdout
     file.docx` (macOS) or `pandoc file.docx -t markdown`, then read the
     converted output. A single doc may contain multiple recipes — split
     them out one at a time.
   - **Word doc (.docx) with embedded photos:** a `.docx` is a zip archive —
     extract images directly with `unzip file.docx -d <tmpdir>`, they'll be
     under `<tmpdir>/word/media/`. Match each image to the recipe it
     appeared next to in the doc text before renaming/moving it (see step
     5).
   - **Photo of a recipe card / handwritten note:** use the Read tool
     directly on the image (it's multimodal) and transcribe the ingredients
     and method as written. Ask the user if anything is illegible rather
     than guessing at quantities.
   - **Photo of a finished dish:** this is an image to attach to a recipe,
     not a source to transcribe — pair it with whatever text source (doc,
     dictation, web link) actually has the ingredients/method.
   - **Web link:** fetch it and extract ingredients + method, dropping site
     cruft (ads, story preamble, nutrition widgets). Some sites (e.g. BBC
     Good Food) can't be fetched: ask the user to upload the page saved as an
     `.mht` file. It's MIME-encoded HTML, so parse it with Python's `email`
     module (`email.message_from_binary_file`, take the `text/html` part),
     strip the tags, and read the ingredients and method from the text.

2. **Normalize into the repo format:**
   ```markdown
   ---
   status: tried
   diet: vegetarian
   type: main
   contains: [onion, garlic]
   ---

   # Recipe Title
   > Source: <where this came from>

   Optional short notes (substitutions, provenance, caveats).

   ## Ingredients

   - ingredient (quantity)

   ## Method

   1. Step one.
   ```
   The YAML block at the very top has four required fields (plus the
   optional `adaptable`):
   - `status`: `tried` (the user has cooked it) or `to_try`. Ask if unclear.
   - `diet`: `vegan`, `vegetarian` or `omnivore`. Only say vegan or
     vegetarian if sure (e.g. marshmallows and some cheeses are not
     vegetarian); when in doubt use `omnivore`.
   - `adaptable` (optional, only when true): `[vegan]` when a vegetarian
     recipe becomes vegan with simple swaps (oil for butter, plant milk for
     milk, rice noodles for egg noodles). Put it on its own line straight
     after `diet`. Leave the line out if there's no easy swap.
   - `type`: `main`, `side`, `dessert`, `bake` (oven-baked bread, cakes,
     biscuits) or `snack`. Soups and noodle salads count as `main`.
   - `contains`: a list using only these words: `onion` (also shallot, leek,
     spring onion), `garlic`, `chickpeas` (also hummus), `wheat` (flour,
     pasta, bread, pitta, biscuits, pastry, noodles, soy sauce, and things
     that typically contain it such as veggie sausages). Tag from the listed
     ingredients, and when in doubt include the word. Use `[]` if none.

   The `> Source:` line goes directly under the title, no blank line before
   it. Include it whenever the source is a URL (use the actual link), and
   otherwise only when it's obvious from context (e.g. a named shared doc)
   — don't invent or guess one if it isn't clear. Keep the rest plain — no
   calorie counts, prep-time badges, or other cruft from the source.
   Preserve the source's own idiosyncrasies (approximate quantities, "??"
   uncertainty, etc.) rather than tidying them into false precision. If a
   quantity is in ounces or pounds, keep it and add grams after it, rounded
   to the nearest 5g, e.g. `(9oz, 255g)`. Do the same for cups (grams for dry
   ingredients using King Arthur's ingredient weight chart, e.g. 1 cup bread
   flour 120g, packed brown sugar 213g, icing sugar 113g, granulated sugar
   200g, butter 227g; millilitres for liquids, using a US cup of 240ml) and
   pints (a UK pint is 568ml).
   Don't write the user's remarks about the recipe into the file ("I've made
   this", "I really like it", "not tried yet", "it was good"): the `status`
   tag already records that. Only put in things that change how to cook it,
   such as substitutions or timing notes the user asked to have added.
   Don't repeat the source (author, book, magazine, site) in the blurb: the
   Source line already covers it. If the recipe came
   from Roger's Recipes (the shared Google Doc), the Source line must say so
   even when it has a link, e.g. `> Source: <url> (via Roger's Recipes, shared
   Google Doc)`, because it means Roger recommends it. Don't add a
   "recommended by" line to the blurb or a YAML field.

3. **Pick the destination folder.** A recipe the user has cooked goes in the
   repo root; one she hasn't cooked (`to_try`) goes in `untested/`. No
   category folders.

4. **Title and filename:** keep titles short, a few words (e.g. "Aubergine
   Miso Roast", not "Miso Aubergines with Tofu, Sesame and Chilli"). The
   filename is snake_case of the short title (md files use underscores, not
   dashes), e.g. `fake_mapo_tofu.md`, saved at `<name>.md` (or `untested/<name>.md`).

5. **Photos:** save the photo in the top-level `assets/` folder (flat, no
   subfolders), named to match the recipe (`fake_mapo_tofu.jpg`, or
   `fake_mapo_tofu_1.jpg`, `_2.jpg` for multiples). Reference it from a recipe
   in the repo root with `![](assets/fake_mapo_tofu.jpg)`, or from a recipe in
   `untested/` with `![](../assets/fake_mapo_tofu.jpg)`. Only use photos the
   user took or supplied, not a publisher's photos.

6. **Update the contents list** in `README.md`: add a link to the new recipe
   (title as the link text), in alphabetical order, under the main list or
   under "Untested". If a recipe moves out of `untested/`, move its link too.

7. **Show the user the new file(s) before considering the task done.** Don't
   commit unless asked; if they ask for it to be added to GitHub, commit and
   push to `main`.
