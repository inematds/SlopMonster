# SlopMonster

**🇧🇷 [Português](README.md) · 🇺🇸 [English](README.en.md) · 🇪🇸 [Español](README.es.md)**

![The five SlopMonster mascots lined up, one for each rule it scores](docs/img/hero.png)

**Turn AI-written text into text a person would publish.**

## 📖 User guide

Complete guide (landing page + walkthrough): **https://inematds.github.io/SlopMonster/guia/en/**

AI text has a smell. `delve`, `seamless`, `unlock`, `it's not just a tool, it's a
journey`. Readers already notice, and a page with that smell is a page they stop
trusting.

Most tools that fix this come from the same public research. This one adds the two things
the others skip.

**It gives your text a score from 0 to 5 and can fail your build.** No opinions, no
guesswork, just patterns. Developers call this a linter. Everyone else can call it a
checker that won't let you publish.

**A rival model does the cleanup.** If Claude wrote the draft, GPT cleans it up. A model
can't hear its own accent, just as you can't hear yours.

It works on landing pages, READMEs, emails, and scripts. Anything a person will read and
judge.

## The cycle

```
1. SCORE      tools/deslop.py     scores from 0 to 5, exits red below 5. Patterns, no opinions.
2. REWRITE    three passes        kills the vocabulary → kills the forms → puts a person back in
3. CLEANSE    tools/cleanse.sh    a different model family strips the marks the first one left
4. RESCORE    tools/deslop.py     only publishes at 5/5
```

The scorer has the first and last word, because the scorer is honest and the model is
persuasive. "Almost clean" is how a page ends up sounding like every other AI page on the
internet.

## Receipts, not promises

A real run on seven verbatim lines from the Jasper.ai homepage (Aug 29, 2026):

| | score |
|---|---|
| Their text, as downloaded | **3/5**, with `unlock`, `empower`, and four lists of three |
| After one pass through this cycle | **5/5**, meaning intact, length within 10%, nothing invented |

Every command and its exact output: [`examples/jasper-live-run.md`](examples/jasper-live-run.md).
Their text is cited for criticism and remains theirs. The MIT license below
covers the code and prose in this repository.

## Quick start

```bash
git clone https://github.com/inematds/SlopMonster && cd SlopMonster

# score anything
python3 tools/deslop.py --text "It's not just a tool, it's a game-changing journey."
# → score 3/5, flags both marks, exits with code 1

# score a finished page (reads only what the visitor SEES)
python3 tools/deslop.py index.html

# score a markdown file (skips code spans, fenced blocks, and struck-through text)
python3 tools/deslop.py README.md

# cleanse a draft with a rival model and rescore it
tools/cleanse.sh rascunho.md > limpo.md
python3 tools/deslop.py --text "$(cat limpo.md)"

# your numbers are real and you can prove it? prevent the proof rule from failing the build
python3 tools/deslop.py index.html --allow-proof

# changed a regex? this catches a catalog that silently went half-blind
python3 tools/test_deslop.py
```

**The catalog is English-only.** Text in another language, including Portuguese, gets 5/5
because the scorer can't read it, not because it's clean. The rival-model cleanup step
works in any language, but the score only applies to English text.

No dependencies. The scorer is pure Python, standard library. The cleanup script needs
an AI CLI (`codex` or `claude`), or none at all, in which case it prints the prompt
for you to paste. The `.github/workflows/slop.yml` file is the build gate, ready to
copy into your own repository.

### Install as an agent skill

**Claude Code:** copy this folder to `~/.claude/skills/slopmonster/` and say `/slopmonster`,
"clean the slop out of this" or "de-slop this." **Codex and other agents:** point the agent to
`SKILL.md`. They're plain Markdown instructions, nothing specific to Claude.

## Cleanup knows which model wrote it

The rule: cleanup runs on a **different model family** from the one that wrote the draft.
Different families have different accents, and a model is bad at hearing its own.

| You work in | Draft's accent | What `cleanse.sh` does |
|---|---|---|
| Claude Code | Anthropic | calls **GPT-5.6** through your `codex` CLI, with a timeout and a read-only sandbox |
| Codex / ChatGPT | OpenAI | with `DESLOP_WRITER=gpt`, it calls **Claude** via `claude -p` |
| Gemini CLI | Google | whichever rival CLI is installed |
| no rival CLI | n/a | prints the full prompt for you to paste into the other model family’s chat |

It refuses to send a draft back to its own family. A model correcting its own work is
exactly what this step exists to prevent.

Then it always rescores: a top-tier model is very good at removing marks and quite capable of
adding new ones in the process.

## What the scorer hunts for

Five groups. Trip one and you lose a point. Below 5/5, the command exits red, so
a build can stop there.

Four groups strip away the AI accent. The fifth asks whether the line sells anything.

Each before and after below is a real line from the Ridgeline Roofing site. Strikethrough is
what the first draft said. Bold is what was published. The full record:
[`examples/ridgeline-roofing.md`](examples/ridgeline-roofing.md).

Every example on this page is in `code format` or struck through. That's not decoration. A
literal isn't sales copy, so `deslop.py` skips both when it reads a `.md` file.

![Rule 1, AI vocabulary: delve, leverage, seamless, unlock](docs/img/rule-1-vocab.png)

**1. AI vocabulary.** Words that appear much more often in AI text than in human text.

There are two lists, and they work in different ways.

The first list matches by stem. So `elevate` also catches `elevates`, `elevated`, and
`elevating`. That matters more than it seems. Sales pages are written in the third person.
`Acme elevates your workflow` is the most common form of the word, and an exact search would
miss it entirely.

The second list has words that also have an honest everyday meaning. `crafted`,
`harness`, `landscape`, `journey`. These are matched word by word. So "we craft
furniture by hand" stays clean, and only the marketing usage gets caught.

The fix is a simpler word. Not a fancier synonym for the same idea.

> ~~We leverage industry-leading materials to deliver unparalleled protection.~~
> **We source materials from manufacturers who test for wind, hail and sun.**
> `leverage`, `deliver`, `unparalleled`. None of them says anything, and each costs a line.

![Rule 2, AI constructions: "not just a tool, it's a journey"](docs/img/rule-2-phrases.png)

**2. AI constructions.** This group catches sentence forms, not individual words.

A form is a mold you can fill with anything. `not just X, but Y` is the loudest one in
English today. Once you see it, you can't stop seeing it.

There are 17 forms in the list. Things like `that's where X comes in`, `say goodbye to` and
`whether you're X or Y`, stacks of hedging like `could potentially`, and questions the
author answers themselves.

Both the short and long forms are checked, "it's" and "it is." Formal writing isn't a clever
disguise. It's the default for what a model writes.

Forms matter more than words, because a page can pass a vocabulary test and
still sound machine-written.

> ~~Not just a roof, but peace of mind.~~
> **A written scope and a fixed number before anyone climbs a ladder.**
> The form promises a revelation and delivers an abstraction.

![Rule 3, punctuation cadence: two em dashes in one sentence](docs/img/rule-3-punctuation.png)

**3. Punctuation cadence.** Two em dashes inside the same sentence.

One em dash in a paragraph is punctuation. Three is a tic. Models use em dashes at a rate
three to five times higher than humans.

The check only looks within a 220-character window, and that limit does real work. UI text
has no periods. Menu items, buttons, and labels run together, so a naive sentence break treats
the whole page as one sentence and the rule fires everywhere. A scorer that cries wolf gets
turned off, so the window stays.

Semicolons count too, but only above a floor of three, proportional to page length. Two
semicolons in a long technical document are style, not a mark.

> ~~Our team — trained, certified and local — is ready to help.~~
> **Thirty-eight on the crew, factory-trained for every material we install.**
> The em dashes hid the fact that the sentence had no information.

![Rule 4, rule of three: "faster, smarter, and better"](docs/img/rule-4-rhythm.png)

**4. Rule of three.** Three items in a row. `faster, smarter, and better`.

Three adjectives are a rhythm, not an argument. A tricolon is rhetoric. Three of them on a
page is a machine. Tricolon is just the fancy name for a list of three items.

This check is narrow on purpose, and only two forms trigger it. With the Oxford comma,
it needs three standalone words. Without it, the third item has to be a short phrase that
closes the clause.

The narrowness is the point. "Inspection, repair and replacement for homes and commercial
buildings" are three real things a roofer does, and it passes clean. Flagging that would be
crying wolf, and the next person would turn off the scorer.

> ~~Trusted, reliable and built to last.~~
> **Six nails per shingle, every shingle.**
> A specification beats three adjectives every time. Nobody invents a line like that,
> because invented text doesn't know it.

![Rule 5, sales and marketing: the rewrite based on Krug, Priestley, and Hormozi](docs/img/rule-5-conversion.png)

**5. Sales and marketing.** The first four rules remove the robot. This one asks the
harder question. Does the line sell anything?

Clean text that says nothing is still a dead page. Two things run here.

**The hard rule: never invent proof.** No customer counts, testimonials, or
reviews the business hasn't earned. The scorer flags any number next to a people noun,
such as `10,000+ happy users`. It triggers easily on purpose. A false alarm costs ten seconds.
A mistake leaves a claim on your site you can't back up. If the number is real and you can
prove it, `--allow-proof` downgrades it to a warning and still prints the matches.

Fake proof is a sales failure before it's a writing failure. Nobody buys from a
page caught lying.

> ~~Loved by 10,000+ happy homeowners.~~
> **Project names and photography are placeholders. Swap in your own jobs before this goes live.**
> Say that the space is empty. It sounds like confidence, not weakness.

> ~~The area's most trusted roofing experts.~~
> **Roofing, and only roofing, since 2001.**
> "Most trusted" can't be checked, so the reader discounts it. A date can't be argued with.

**Where the lines come from.** Four sources. If a sentence can't name its source, it
doesn't go on the page.

1. **What the trade actually does.** By far the strongest source. "Six nails per shingle"
   is a real specification with a real failure mode behind it.
2. **What the customer already fears.** That the price will change. That the yard will be
   wrecked. That they're selling a whole new roof to fix a flashing problem.
3. **What the competition won't say.** Refusal travels farther than a promise. "We don't do
   overlays" positions you and disqualifies the wrong customer in one line.
4. **The lines that were already good.** "From first call to final nail" came in written on
   the wireframe and beat every rewrite. It stayed.

**The work named behind the rewrite:**

| Who | What it asks for |
|---|---|
| **Steve Krug**, *Don't Make Me Think* (2000) | every line the reader has to decipher is a line they skip |
| **Daniel Priestley**, pitch order | opens with the problem and the insight, never the product |
| **Alex Hormozi**, the offer side | named pain, checkable specificity, proof you actually have |

Those three plus the category benchmark become five working principles, each with a real
before and after: [`references/principles.md`](references/principles.md).

## What replaces the marks

Clean isn't the same as good. Five principles decide what the line says instead: Krug's
*Don't Make Me Think*, Priestley's pitch order that opens with the problem, and Hormozi's
argument, from the offer side, that specificity beats superlatives. Each with a real
before and after: [`references/principles.md`](references/principles.md).

## A complete example

The Ridgeline Roofing site: from a Lorem ipsum wireframe to a published site, with the
before → after for every headline, the six marks caught in the first drafts, and the
verify-or-flag pass on every number. This is the teaching file:
[`examples/ridgeline-roofing.md`](examples/ridgeline-roofing.md).

> ~~The area's most trusted roofing experts.~~
> **Roofing, and only roofing, since 2001.**
> "Most trusted" can't be falsified, so the reader discounts it entirely. A date can't be argued with.

## The one hard rule

**Never invent proof.** No user counts, testimonials, or reviews you haven't earned. If a
claim needs a number you don't have, write `[needs number]` and move on. The gain from an
invented number is smaller than the gain from real specificity, and it's the only irreversible mistake.

And this skill will never promise to "beat AI detectors." Detectors are noise. The target
is a human reader's instinct.

About this file: `python3 tools/deslop.py README.md` scores **5/5**, but with one honest caveat.
The English examples are marked as literals, in `code` or struck through, and the scorer skips both.
The surrounding prose is in Portuguese, and the catalog doesn't read Portuguese.
The original English version of this README passed its own scorer with nothing softened.

## What it's based on

All sources are in [`references/sources.md`](references/sources.md):
Wikipedia's *Signs of AI writing* (WikiProject AI Cleanup) as the canonical catalog,
plus four open-source humanizers under MIT: `blader/humanizer`,
`harshaneel/humanize`, `lguz/humanize-writing-skill`, `haidrrrry/humanize-ai-writing`.
The rewrite principles come from Krug, Priestley, and Hormozi. Detector evasion
repositories are deliberately excluded.

## Repository map

```
SKILL.md                            the agent skill, the whole cycle as instructions
tools/deslop.py                     the scorer. standard library, no deps, exits red below 5/5
tools/test_deslop.py                regression suite. run after changing any regex
tools/cleanse.sh                    rival-model cleanup, routed automatically, with a timeout
.github/workflows/slop.yml          the build gate, ready to copy
prompts/cleanse.txt                 the exact instruction the cleanup model receives
references/signs-of-ai-writing.md   the full catalog: 2 vocab tiers, 8 forms, cadence, rhythm, proof
references/principles.md            the five rewrite principles, each with a real pair
references/sources.md               every source this is based on
examples/ridgeline-roofing.md       full site, every line before → after
examples/jasper-live-run.md         real unedited run: 3/5 → 5/5 on a real page
```

MIT. Same as the humanizers this is based on.
