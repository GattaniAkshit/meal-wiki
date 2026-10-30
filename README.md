# meal-wiki

A personal nutrition assistant I am building, to run in an end-to-end encrypted Matrix room.
Nothing is built yet (see Status).

The design: I send it a photo or a line of text about what I ate. It works out the components and
amounts, looks up their nutrients, keeps running totals against my goals, and asks me when it is
unsure. What it learns about me (goals, preferences, pantry, the dishes I cook) is kept as dated
facts and rendered into an Obsidian vault that I can read and edit.

It is meant as the first domain of a personal knowledge system. Training and calendar data come
later, joined to this one on the time each thing happened.

## How it is meant to work

The language model parses and does nothing else. A photo or a message becomes a list of
components, each with an amount in grams if one is known, whether it was raw or cooked, and my
name for it. Code does the rest: the lookup, the logging, and the decision to ask.

Once the components are known, nutrients per gram come from the first of these that exists. The
first four are lookups and the fifth is an estimate, and the stored value records which one it was:

1. a value I stated
2. a nutrition label I photographed
3. one of my saved recipes
4. BLS 4.0 (Bundeslebensmittelschlüssel), the German national food composition database
5. the model's own estimate

The error left after the lookup sits in five places: fats I did not state, such as the oil in the
pan; my share of a batch I cooked; whether a weight was raw or cooked; a name matched to the wrong
database entry; and the gap between a generic entry and my actual product. The last is a floor.
The bot's questions target the other four. It asks about an item when the likely error is over 20%
of the item's energy and worth more than about 30 kcal, both starting values. Which items need
asking is learned from the corrections I make.

- **Chat:** Matrix with end-to-end encryption. The homeserver stores ciphertext and sees metadata
  (who, when, message sizes). The bot decrypts on its server. The GPU provider also sees each photo
  while the model reads it; whether it keeps inputs depends on its data terms, which are checked
  before any real photo is sent.
- **Model:** one open-weight vision-language model and bge-m3 embeddings on a serverless GPU. There
  is no hosted fallback. If the GPU is down, work waits in a queue and the bot says so.
- **Storage:** DuckDB for meals, foods, recipes, corrections and facts. A fact has a validity
  window. When it stops being true it is closed, and the old version stays.
- **Vault:** Obsidian pages generated from the facts by a fixed template. An edit I make in
  Obsidian is read back as a new fact, and the bot asks me when an edit would otherwise be lost.
- **Hosting:** a small VPS for the bot. The budget is €10-12 a month for the VPS and the GPU
  together; what a logged meal costs in GPU time is measured in the first milestone.

## Status

Started 30 October 2026. Nothing is built yet.

The first milestone is a gate: does one open-weight model find all the major components, fats
included, in at least 8 of 10 of my real meal photos, and what does a logged meal cost in GPU
time? 8 of 10 is consistent with a true rate as low as 0.44 (Clopper-Pearson 95% lower bound), so
the result will be reported as a count with that bound. This README grows with each milestone and
says what has been measured and what has not.
