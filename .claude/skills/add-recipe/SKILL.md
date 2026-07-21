---
name: add-recipe
description: Use when the user wants to add a recipe to this repo, from a Word doc, a photo (of a recipe card, handwritten note, or the finished dish), or a web link. Converts the source into the repo's standard recipe format and saves it in the right place.
---

# Add recipe

Adds one recipe to this repo in the repo's standard format: loose category
folders (e.g. `chinese/`, `baking/`), one markdown file per recipe, and a
single flat top-level `photos/` folder for images.

## Steps

1. **Identify the source and extract the raw content.**
   - **Word doc (.doc/.docx):** these aren't plain text — don't try to Read
     them directly. Convert first, e.g. `textutil -convert txt -stdout
     file.docx` (macOS) or `pandoc file.docx -t markdown`, then read the
     converted output. A single doc may contain multiple recipes — split
     them out one at a time.
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

   Optional short notes (substitutions, provenance, caveats).

   ## Ingredients

   - ingredient (quantity)

   ## Method

   1. Step one.
   ```
   Keep it plain — no calorie counts, prep-time badges, or other cruft from
   the source. Preserve the source's own idiosyncrasies (approximate
   quantities, "??" uncertainty, etc.) rather than tidying them into false
   precision.

3. **Pick the destination folder.** Categories are loose and made up as
   needed (`chinese/`, `baking/`, etc.) — check existing folders for a fit
   first; if the user hasn't said which one, ask rather than guessing a new
   category into existence.

4. **Filename:** kebab-case of the title, e.g. `fake-mapo-tofu.md`, saved at
   `<category>/<name>.md`.

5. **Photos:** save to the top-level `photos/` folder (flat, no
   subfolders), named to match the recipe (`fake-mapo-tofu.jpg`, or
   `fake-mapo-tofu-1.jpg`, `-2.jpg` for multiples). Reference them from the
   recipe file with a relative path: `![](../photos/fake-mapo-tofu.jpg)`.

6. **Show the user the new file(s) before considering the task done.** Don't
   commit unless asked.
