---
name: add-recipe
description: Use when the user wants to add a recipe to this repo, from a Word doc, a photo (of a recipe card, handwritten note, or the finished dish), or a web link. Converts the source into the repo's standard recipe format and saves it in the right place.
---

# Add recipe

Adds one recipe to this repo in the repo's standard format. Recipes should be
as few clicks away as possible: loose category folders sit at the top level
(e.g. `soup/`, `seafood/`), each recipe is one markdown file directly inside a
category folder, and a recipe's photo sits next to it in the same folder.
Recipes not cooked yet go in `to_try/`.

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
     cruft (ads, story preamble, nutrition widgets).

2. **Normalize into the repo format:**
   ```markdown
   # Recipe Title
   > Source: <where this came from>

   Optional short notes (substitutions, provenance, caveats).

   ## Ingredients

   - ingredient (quantity)

   ## Method

   1. Step one.
   ```
   The `> Source:` line goes directly under the title, no blank line before
   it. Include it whenever the source is a URL (use the actual link), and
   otherwise only when it's obvious from context (e.g. a named shared doc)
   — don't invent or guess one if it isn't clear. Keep the rest plain — no
   calorie counts, prep-time badges, or other cruft from the source.
   Preserve the source's own idiosyncrasies (approximate quantities, "??"
   uncertainty, etc.) rather than tidying them into false precision.

3. **Pick the destination folder.** If the user hasn't cooked it yet, it goes
   in `to_try/`. Otherwise it goes in a category folder at the top level.
   Categories are loose and made up as needed (`chinese/`, `baking/`, etc.) —
   check existing folders for a fit first; if the user hasn't said which one,
   ask rather than guessing a new category into existence.

4. **Title and filename:** keep titles short, a few words (e.g. "Aubergine
   Miso Roast", not "Miso Aubergines with Tofu, Sesame and Chilli"). The
   filename is snake_case of the short title (md files use underscores, not
   dashes), e.g. `fake_mapo_tofu.md`, saved at `<category>/<name>.md`.

5. **Photos:** save the photo in the same folder as its recipe, named to
   match it (`fake_mapo_tofu.jpg`, or `fake_mapo_tofu_1.jpg`, `_2.jpg` for
   multiples). Reference it from the recipe file with a plain relative path:
   `![](fake_mapo_tofu.jpg)`. Only use photos the user took or supplied, not
   a publisher's photos.

6. **Show the user the new file(s) before considering the task done.** Don't
   commit unless asked.
