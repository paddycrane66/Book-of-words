# Build brief: "Book of Words" (Elden Ring x Scribblenauts Unlimited)

Paste this into Claude Code or Codex on the PC that has the games, together with the Publish prompt from Melty (it carries the connection details; keep the token out of files, commits and replies).

The user has already approved this plan. Build it, test it and prepare the listing. Do not publish until the user has tested it in Melty and says so.

## The idea
Elden Ring is the host game (solo, offline). Scribblenauts Unlimited contributes the idea only: the player writes any word and something happens. The player starts with a **Book of Words**. They open an in-game text box, type a word, and it is matched to the closest spell. Unknown words fall back to a small default effect, so typing never fails.

## Decisions already made
- Host game: Elden Ring, run offline through ModEngine2 (Melty installs it and starts the game with Easy Anti-Cheat off). Never start the game any other way and never work around the anti-cheat.
- Solo only. No multiplayer, so leave `recipe.multiplayer` out.
- First minute: the player starts with the Book and can type a word and cast right away.
- Free typing, matched to a fixed spell set. No online service, no API key, works offline.
- First playable version: about 100 words in categories (elements, animals, objects) with synonyms.
- Scribblenauts Unlimited is not in Melty's catalog and its files are not needed. Check with `game_info`/`search_games` whether Melty wants it as a required game.

## Step 0: read up (before building)
1. Connect to Melty, call `list_my_mods` (reuse a draft only if it is clearly this project).
2. Call `game_info` for Elden Ring (what stops players of other mods, camera/key bindings, safety) and `search_games` for Scribblenauts Unlimited. Call `search_mashups` for both games and tell the user if something close already exists (remixing may be better). Run `mashup_info` on the closest launched one as a recipe example.
3. Set up universal-modder (github.com/rehan-remade/universal-modder), follow `skills/mod-any-game/SKILL.md`. Ask the user before anything that costs money.

## Biggest risk: the in-game text box
ModEngine2 mostly swaps game data. It does not draw interface or read typed text by itself. Work out first whether the text box needs a small DLL or helper loaded through ModEngine2.
- It must be loadable by ModEngine2 so a player presses Play once.
- Anything bundled must have a license that allows sharing; credit its author in the listing.
- If it cannot work in one click, tell the user exactly why before building further. Do not continue silently.
- Fallback to propose if typing is impossible: a pick-from-a-list Book of Words (the user's earlier option 1).

## Design sheets (source of truth, change the sheet before the code)
Make JSON sheets. Each row is one thing, each column a property, and each row generates one piece of code.
- `words.json`: id, word, category, spellId
- `synonyms.json`: synonym, wordId (many synonyms per word)
- `categories.json`: id, name, defaultSpellId (fallback for an unknown word in a category)
- `spells.json`: id, name, base Elden Ring spell/effect it reuses (or the effect hook), FP cost, cooldown, damage/effect values, VFX/SFX reference
- `hooks.json`: every point where the mod touches the game (give the Book at start, open text box, read input, grant/cast spell)
- `book.json`: the Book item (where it is granted, how it opens)

Treat every cell as a checkbox. Reuse existing game spells and effects where possible; only make new assets if needed (fal.ai art/sound needs a key, so ask first).

**Preflight before every build:** lay all sheets over each other and list every empty cell and every cross-sheet reference that does not resolve (word -> spell, synonym -> word, spell -> game effect, hook -> game). Fix them all, build only when clean.

## Build and test
- Prefer the established route: ModEngine2's mod folder (`{managed}/modengine2/mod`) rather than patching game files.
- Build, fix errors, and test in the running game: open the Book, type words from each category, an unknown word, an empty entry and an odd one (spaces, numbers, long text). Report exactly what you tested; mark anything untested as untested.
- Check that normal play is unaffected when the Book is not opened.

## Package and publish (Melty's tools only)
1. Package the release. Run `inspect_package` on every archive entry, fix its findings, then `validate_recipe` and `one_click_check` (include `says`: the feature list from the README). Fix everything it lists. Ask for ModEngine2 in the recipe rather than bundling it.
2. Suggest a title, one-line tagline and short player-friendly description based on the real build; let the user choose. Ask for credits, content license and remix choice (do not guess permissions).
3. `create_mod` (or `update_mod` for the chosen existing listing), record the modId.
4. Upload with `start_upload` (real byte size and SHA-256), PUT the bytes with the returned headers, `finish_upload`, then `submit_release` with the version, uploadIds and validated recipe.
5. Capture a real screenshot or clip of this build running, showing the distinctive behavior (typing a word and the spell casting). Game only, no tokens or private windows. Upload with `add_screenshot`, PUT, `finish_screenshot`. No mockups.
6. Show the user a brief summary (what players get, required games, credits/remix choice, test result). Ask them to try it in the Melty app (Test), and publish only once it is tested and they authorize it. Then `mod_status` to confirm; say "live" only when the server does. Say "plays in one click" only when `one_click_check` says yes and the install was tested.

If a build, a working install recipe or real gameplay media is missing, keep the listing as a draft and tell the user exactly what remains.
