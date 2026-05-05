# filtering

this folder is the source-of-truth for what sodalite's safety filter
catches at each level. it's split by **level** (the button in your
privacy settings) and then by **category**.

```
filtering/
  relaxed/         words filtered at relaxed and above
    slurs/         racist / homophobic / ableist slurs
    threats/       death threats, kys, doxx threats
    doxxing/       address / phone-number / personal-info giveaways
  moderate/        added on top of relaxed at moderate and above
    profanity/     light profanity (fuck, shit, bitch, etc.)
  strict/          added on top of moderate at strict only
    sex/           sexual / explicit terms
    suggestive/    suggestive but not explicit (innuendo, kinks, etc.)
```

## how it works

- each level **stacks**: `moderate` includes everything in `relaxed`
  plus `moderate`. `strict` includes everything in `relaxed` +
  `moderate` + `strict`. `off` includes nothing.
- the file format is plain text — one word per line. blank lines and
  lines starting with `#` are ignored.
- matching is **case-insensitive at word boundaries** with leet-speak
  normalisation (`f4ck`, `sh!t`, etc). so `sex` catches `"sex"` and
  `"having sex"` but NOT `"essex"`, `"sextet"`, `"sexism"`. `cock`
  catches `"cock"` but not `"peacock"` or `"shuttlecock"`. `rape`
  catches `"rape"` but not `"drape"`, `"grape"`, or `"scrape"`.
- compound usage still gets caught when the listed word appears at
  a word boundary. `cock` catches `"cocksucker"` (boundary at the
  start). compound entries like `cumshot`, `gethighwithme`,
  `findyourhouse` are kept as their own lines so they trip cleanly
  even when broken across spaces.
- users can also add their own custom words on top of any level via
  settings → privacy → your custom blocks.

## suggesting a change

found a slur we missed? a word that shouldn't be there? a category
mistake? open a pr.

1. fork the repo
2. edit the relevant `.txt` file (or add a new one in the right folder)
3. open a pr — explain why you think it should be added / removed /
   moved
4. one of the maintainers will review

a few rules:
- **don't add joke words.** this isn't a profanity wishlist — it's a
  real filter that affects real users (including kids).
- **avoid bare words that are also legitimate english at word
  boundaries.** `of` is a preposition; `rope` is a noun; `molly` is
  a name. word-boundary matching handles substring collisions
  (essex, raccoon, drape) but it can't help when the bare word is
  innocent on its own. for those, prefer a compound form
  (`gethighwithme` instead of `high`, `swatyou` instead of `swat`).
- **slurs are never debatable.** if it's a slur, it goes in `slurs/`,
  full stop.
- **regional variations welcome.** if a word is filtered in english
  but not its spanish / french / etc. equivalent, that's a gap.

## the in-app viewer

sodalite ships a "see what's blocked" page at settings → privacy that
mirrors these files exactly. the build script in the sodalite repo
(`scripts/build-filter-words.mjs`) clones this repo on every deploy
and bakes the contents into the client bundle, so the viewer is
always in sync with what's on github.

### labels and ordering

`meta.json` at the root of this folder controls the ui labels for
each level + category, plus the display order. add a new category
folder and you should also add it to `categoryOrder` and `categories`
in `meta.json` so the viewer knows what to call it.

## why not just use a library

we want users to be able to read the actual list. opaque "ai
moderation" or imported npm packages mean nobody knows what's filtered
or why. plain text in a public folder = full transparency.
