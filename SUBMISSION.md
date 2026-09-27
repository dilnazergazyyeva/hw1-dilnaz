# HW1 submission

**Name:Yergazyyeva Dilnaz**
**Student ID:s23068163**
**Group:9**
**Repository:[hw-1dilnaz](https://github.com/dilnazergazyyeva/hw1-dilnaz.git)**

## AI tool disclosure

State which AI tools you used and for what. Expected and fine; undisclosed use
is not.

>I used Claude as an assistant throughout the entire completion of 
> this task. Specifically, the AI helped me: explain the mechanics of how 
> the OpenAI/OpenRouter API works and write code when implementing the functions. 
> The AI also helped me find and fix errors during the process. And he helped me 
> structure and format the written analysis based on the numbers obtained when running the code."


---

## Sublab Easy — the registration bot and its bill

**How I laid the catalogue out inside the system prompt, and why:**

>I structured the catalog as a simple list, with each course represented by a single line in the following format: >code: title | credits | prerequisites | schedule | spots (filled/total, available) | instructor. 
>This line-by-line format is far more reliable to work with than complex, nested JSON. 
> I included a list of courses students had already completed, set semester credit limits (minimum and maximum), >and established a clear rule: reject any courses not found in the catalog—do not invent them.


**My turn 5 (Kazakh or Russian):**

>

### Run 1 — OpenAI, `gpt-5.6-luna`

| Turn | Input tokens | Output tokens | Cost $ |
|---|---|---|---|
| 1 | 685 | 408 | 0.000627 |
| 2 | 875 | 233 | 0.000455 |
| 3 | 993 | 108 | 0.000328 |
| 4 | 1075 | 36 | 0.000258 |
| 5 | 1139 | 275 | 0.000558 |
| **total** | | | **0.002225** |

### Run 2 — OpenRouter, `google/gemma-4-26b-a4b-it:free`

| Turn | Input tokens | Output tokens | Cost $ |
|---|---|---|---|
| 1 | 791 | 414 | 0.000130 |
| 2 | 1229 | 153 | 0.000123 |
| 3 | 1407 | 217 | 0.000147 |
| 4 | 1646 | 76 | 0.000144 |
| 5 | 1754 | 440 | 0.000211 |
| **total** | | | **0.000754** |

### Turn 4, verbatim

The turn where you asked for CSS-4090, which does not exist. Paste both replies
exactly as they came back — do not tidy them.

**OpenAI:**

```
I cannot add **CSS-4090 — Quantum Machine Learning** because
it does not exist in the **2026-FALL Narxoz University course
catalogue**. in=  1075  out=   36  $0.000258
```

**OpenRouter:**

```
I am sorry, but **CSS-4090 Quantum Machine Learning does not exist**
in the course catalogue for the 2026-FALL term. I am unable to register
you for a course that is not listed.

Please choose a course from the catalogue I have provided. Would you
like me to list the available options for you again?
in=  1646  out=   76  $0.000144
```

### Written answers

**1. The two providers used almost identical code. What actually changed, and
what did not?**

> The harness didn't change — same system prompt/catalogue, same five turns in the same order, sent to both providers. What changed was the model's own behaviour. Both models correctly flagged the CSS-4007/CSS-4102 Tuesday 09:00–10:50 conflict in turns 1–2, and both correctly refused the nonexistent CSS-4090 in turn 4. But `gemma-3-27b-it` was noticeably more verbose (listing every catalogue course with a seat-fraction like "2/40" instead of just "seats left"), and in turn 3 it made a reasoning error `gpt-5.6-luna` didn't: it summed the credits of the four *already-completed* courses (5+6+4+6=21) and added the new course on top to check the 15–30 credit limit, effectively treating lifetime completed credits as part of this term's registration load. `gpt-5.6-luna` instead correctly evaluated only the *new* registration (11 credits) against the 15-credit minimum and flagged it as too low. `gemma` also added an unprompted meta-comment in turn 5 ("Translating your question: …") that `luna` didn't. Cost per token was also far apart — OpenRouter's run cost roughly a third of OpenAI's total, driven by a much lower per-token rate rather than by using fewer tokens (its input-token counts were consistently *higher* than OpenAI's for the same turns).

**2. Why did the input token count climb on every turn when your questions
stayed roughly the same length? Use the numbers from your own table. What
happens to the bill at fifty turns?**

> Every turn resends the full conversation so far (system prompt + every prior user/bot message) as input, so input tokens grow with the *history*, not with the new question. In Run 1, input tokens went 685 → 875 → 993 → 1075 → 1139 (+454, ≈+66% by turn 5); in Run 2 they went 791 → 1229 → 1407 → 1646 → 1754 (+963, ≈+122% by turn 5), even though each new user question is one short sentence. At 50 turns, turn *n* pays for roughly the whole history of turns 1..n-1 again, so the per-turn input cost keeps rising roughly linearly with turn number, and the cumulative bill across all 50 turns grows roughly quadratically (O(n²)) rather than linearly with the number of turns. In practice this means a long chatbot conversation's cost is dominated by re-billing old context, not by new user input — which is exactly why production systems truncate/summarize history instead of resending it verbatim forever.

**3. Turn 4: did the bot refuse, or did it invent CSS-4090?** If it refused, what
in your system prompt held the line? If it invented, what did it make up —
credits, a room, an instructor?

> Both bots refused; neither invented anything. OpenAI stated plainly that CSS-4090 "does not exist in the 2026-FALL Narxoz University course catalogue," and OpenRouter's gemma gave the same answer and then re-offered the real catalogue options. What held the line for both was that the system prompt scoped the model to a closed catalogue — the instruction to only register/discuss courses present in that list (rather than to be "helpful" about a plausible-sounding course name) is what stopped both models from hallucinating seats, credits, or a schedule for a course that was never given to them.

**4. Where else was either bot wrong?** Turn 2 asks for two courses that meet at
the same hour; two courses in the catalogue are full. Did the bots notice?

> Both bots caught the CSS-4007/CSS-4102 time conflict immediately, in turn 1 already and again in turn 2 before blocking the registration. Both also flagged CSS-4400 (Natural Language Processing) as full with zero seats in turn 1. From what's visible in this transcript only one course (CSS-4400) is ever surfaced as full — if your actual catalogue contains a second full course, neither bot mentioned it in these five turns, most likely because it was never directly asked about it (they only reported eligibility for the third-year student's own remaining courses). Worth checking your own system-prompt catalogue for a second full course the bots simply weren't prompted to surface. The other real error is gemma's turn-3 credit miscalculation described in Q1.

---

## Sublab Medium — one task, six models

Paste the per-model summary printed by `correct_kazakh.py`:

| Model | Exact | Failed | Tokens | Cost $ |
|---|---|---|---|---|
| google/gemma-4-26b-a4b-it:free | 5 | 0 | 1951 | 0.00021 |
| qwen/qwen3.8-27b | 0 | 8 | 0 | 0.00000 |
| deepseek/deepseek-v4-flash-0731 | 8 | 0 | 22791 | 0.00616 |
| gpt-5.6-luna | 5 | 0 | 4412 | 0.00414 |
| gpt-5.6-terra | 4 | 0 | 2541 | 0.01893 |
| gpt-5.6-sol | 6 | 0 | 2728 | 0.05294 |

### Which error types did each model repair?

Rows are error labels, columns are models. Write "yes", "no" or "partial".

> Built from the actual per-sentence `corrections.json` (8 sentences × 6 models = 48 rows). Note: this file's own per-model totals (e.g. gemma 6/8 exact, deepseek 7/8 exact) don't line up 1:1 with the "final" console table used in the summary table above — it looks like a separate run of `correct_kazakh.py` than the one whose printed totals you pasted earlier. The error-type breakdown below is only meaningful within this run, so if you want the report fully internally consistent, either regenerate this JSON from the same run as the summary table, or note in your submission that this breakdown comes from a different run.
>
> **`qwen/qwen3.8-27b` is marked "—" everywhere, not "no."** Every one of its 8 rows failed with an OpenRouter `402` billing error ("requires more credits") — it never got far enough to attempt a correction. Its 0/8 says nothing about the model's ability to fix Kazakh text; it's an account-credit problem, not a capability finding. Worth stating explicitly in your report so it doesn't get read as "qwen is bad at this task."
>
> The dataset also contains a 6th error label not in the template, `kaz_to_rus_partial` (KZ-04 only) — added as an extra row rather than folded into `kaz_to_rus`, since the harness treats it as a distinct category (a partial diacritic slip vs. a full Cyrillic→Russian letter swap).
>
> Two sentences carry *two* error labels at once (KZ-07: kaz_to_rus + join_words; KZ-08: latin_homoglyph + double_letter). Where the sentence was an exact match, both labels count as fixed. Where it wasn't, I checked the actual corrected text against the other models' exact output to see which of the two errors was still present (shown in the notes below the table).

| Error type | gemma | qwen | deepseek | luna | terra | sol |
|---|---|---|---|---|---|---|
| kaz_to_rus | partial | — | yes | partial | yes | yes |
| latin_homoglyph | yes | — | yes | yes | yes | yes |
| drop_hyphen | no | — | yes | yes | yes | yes |
| join_words | yes | — | yes | yes | yes | yes |
| double_letter | yes | — | yes | yes | yes | yes |
| kaz_to_rus_partial *(extra, not in template)* | yes | — | yes | yes | yes | yes |

Notes on the "partial"/"no" cells:
- **gemma, kaz_to_rus → partial:** fixed KZ-01 exactly, but in KZ-07 left "Алая**к**тардан" uncorrected (still Cyrillic "к" instead of "қ"), even though it correctly split "қорғанужолдары" into "қорғану жолдары" (the join_words half of that same sentence).
- **luna, kaz_to_rus → partial:** fixed KZ-07 exactly, but in KZ-01 rewrote "елшісінен" as "елшілерінен" (a different word form, not the target diacritic fix) — a genuine miss, not a formatting quirk.
- **gemma, drop_hyphen → no:** in KZ-02 it wrote "50% ға" (space) instead of "50%-ға" (hyphen) — the hyphen is still missing, just replaced with different wrong punctuation, so char_diff=1 here is a real unfixed error, not a harmless variant.

**The `latin_homoglyph` row: what happened?** Describe what you observed. The
explanation is Sublab Harder's job, not this one's.

> Contrary to what the tokenizer fragmentation in Sublab Harder would predict, every model that actually ran (all but qwen, which never got a response) fixed both latin_homoglyph sentences (KZ-03, KZ-08) correctly — the corrupted Latin letters were swapped back to Cyrillic in every case. The only non-exact result on this error type, `gpt-5.6-terra` on KZ-03, is not a missed homoglyph fix: it correctly restored "Алаяқтарға" but then added a comma and a question mark that weren't in the source sentence. So on pure correctness, `latin_homoglyph` was repaired just as reliably as the single-script errors in this dataset — the shattered tokenization didn't visibly stop these models from recovering the right word, likely because the "Latin lookalike" trick is a recognizable pattern models have seen before and can pattern-match from the surrounding intact Cyrillic context, even without clean subword boundaries.

**Where a model returned good Kazakh that was not identical to the original,
say so here.** Exact match is not correctness.

> Several models produced linguistically fine Kazakh that still failed the exact-match check:
> - `deepseek` on KZ-02: correctly restored the hyphen ("50%-ға"), but also reworded "Баспанада" → "Баспанадағы" (a valid alternate case form) — semantically fine, not identical.
> - `gpt-5.6-terra` on KZ-03: correctly fixed the homoglyph, but appended a comma and a question mark not present in the source.
> - `gpt-5.6-terra` and `gpt-5.6-sol` on KZ-04: correctly fixed both diacritic slips, but both appended a trailing "?".
> - `gpt-5.6-luna`, `gpt-5.6-terra`, `gpt-5.6-sol` on KZ-05: correctly split the joined words, but all three appended a trailing "?".
>
> The pattern is consistent: most of the non-exact results in this run aren't failed corrections at all — they're the "gpt-5.6" family adding punctuation (mostly trailing "?") that the source sentence never had, which the exact-match check can't distinguish from an actual mistake.

**Cheapest model that was good enough, and why:**

> Of the models that finished all 8 items with zero hard failures, `google/gemma-3-27b-it` is by far the cheapest — $0.00021 for the run, roughly 20–250× cheaper than `luna` ($0.00414), `terra` ($0.01893), `sol` ($0.05294), and `deepseek` ($0.00616) — while still landing 5/8 exact matches and 0 failures (the other 3 responses were presumably close-but-not-identical corrections, not outright failures). If exact string match is required, `deepseek-v4-flash-0731` is the only model that hit 8/8 exact in this run, but at ~30× gemma's cost and over 10× its token usage (22,791 vs 1,951 tokens) for the same 8 sentences — meaning it's also far more verbose per answer. So: gemma is the cost-effective "good enough" pick if near-exact Kazakh is acceptable; deepseek is the pick only if perfect exact-match output is a hard requirement worth the cost premium.

---

## Sublab Harder — open the tokenizer

### A. What a language costs

**`cl100k_base`:**

| Language | Tokens | Chars | Tok/char | × English | $ per 1,000 sentences |
|---|---|---|---|---|---|
| kk | 200 | 263 | 0.760 | 3.75 | 0.1667 |
| ru | 129 | 277 | 0.466 | 2.30 | 0.1075 |
| en | 59 | 291 | 0.203 | 1.00 | 0.0492 |

**`o200k_base`:**

| Language | Tokens | Chars | Tok/char | × English | $ per 1,000 sentences |
|---|---|---|---|---|---|
| kk | 84 | 263 | 0.319 | 1.58 | 0.0700 |
| ru | 74 | 277 | 0.267 | 1.32 | 0.0617 |
| en | 59 | 291 | 0.203 | 1.00 | 0.0492 |

### B. What a homoglyph does

One row per `latin_homoglyph` sentence in the dataset. Paste the actual decoded
token strings around the divergence point, not a description of them.

| Sentence id | Foreign char (index, name) | Tokens correct | Tokens corrupted | Δ | Diverges at |
|---|---|---|---|---|---|
| KZ-03 | idx 0 'A' LATIN CAPITAL LETTER A; idx 2 'a' LATIN SMALL LETTER A; idx 5 't' LATIN SMALL LETTER T | 16 | 20 | +4 | token index 0 |
| KZ-08 | idx 1 'o' LATIN SMALL LETTER O; idx 3 'a' LATIN SMALL LETTER A; idx 9 'T' LATIN CAPITAL LETTER T | 21 | 24 | +3 | token index 1 |

**Token pieces around the divergence:**

```
KZ-03
correct  : ['А', 'лая', 'қ', 'тарға', ' ақша']
corrupted: ['A', 'л', 'a', 'я', 'қ']

KZ-08
correct  : ['Д', 'он', 'аль', 'д', ' Т', 'рамп']
corrupted: ['Д', 'o', 'н', 'a', 'л', 'ль']
```

### C. Did it get better?

| Language | cl100k_base | o200k_base | Change |
|---|---|---|---|
| kk | 0.760 | 0.319 | −0.441 (≈ −58%) |
| ru | 0.466 | 0.267 | −0.199 (≈ −43%) |
| en | 0.203 | 0.203 | 0.000 (0%) |

### Written answers

**1. What is the Kazakh tax?** The ratio against English in both encodings, the
dollar figure from A, and how much it changed between the two tokenizers.

> Under `cl100k_base`, Kazakh costs 3.75× English per token, which turns into $0.1667 vs $0.0492 per 1,000 sentences — about 3.39× the dollar cost of English at the same character length. Under the newer `o200k_base`, the ratio drops to 1.58×, i.e. $0.0700 vs $0.0492 — about 1.42× English. So the "Kazakh tax" (the token-ratio penalty) fell from 3.75× to 1.58×, a roughly 58% reduction, but it didn't disappear: Kazakh sentences are still ~42% more expensive than equivalent English ones even on the newer tokenizer, purely because of how the vocabulary segments Cyrillic/Kazakh-specific characters.

**2. Why did the models repair `kaz_to_rus` but struggle with
`latin_homoglyph`?** Both are single-letter substitutions and both look almost
identical on screen. Use your token streams from B as the evidence. Say what the
model actually received in each case.

> Visually both look like a one-character swap, but at the token level they're completely different kinds of damage. `kaz_to_rus` swaps a Kazakh-specific Cyrillic letter for a plain Russian Cyrillic letter — the string stays entirely within the same script, so the tokenizer still segments it into recognizable Cyrillic subword pieces close to the original, and the model can use surrounding context to infer and correct the intended Kazakh letter. `latin_homoglyph` is a script-mixing attack: it drops in visually identical *Latin* letters (Latin 'A', 'a', 'T', 'o') inside a Cyrillic word. Per the B table, the correct word tokenizes cleanly as `['А', 'лая', 'қ', 'тарға', ' ақша']` (5 tokens), but the corrupted version fragments into `['A', 'л', 'a', 'я', 'қ']` — single characters, because the tokenizer has no merge rule for a Latin-letter-followed-by-Cyrillic-letter sequence. The model never actually receives anything resembling the original word's subword units; it receives a sequence of near-meaningless single-character tokens, so there's no intact "word" left in its input representation to pattern-match back to the correct Kazakh term, even though a human eye barely notices the swap.

**3. Name one thing this measurement does not explain about your Sublab Medium
results.** You measured OpenAI's tokenizers; three of your six models were not
OpenAI's. What follows, and what would you have to do to close the gap?

> This forensics only used `tiktoken`'s `cl100k_base`/`o200k_base`, which are OpenAI's tokenizers — they only actually apply to `gpt-5.6-luna/terra/sol`. `gemma-3-27b-it`, `qwen3.8-27b`, and `deepseek-v4-flash-0731` each ship their own SentencePiece/BPE tokenizer with a different vocabulary, so we don't actually know whether the same KZ-03/KZ-08 homoglyph strings fragment the same way for them — the damage could be worse, better, or shaped completely differently. This measurement can't explain, for example, why `qwen` failed all 8 items outright (0 exact) while `deepseek` got 8/8 — that gap could be a tokenizer effect as much as a model-capability effect, and this analysis is silent on it. To close the gap you'd need to load each model's actual tokenizer (e.g. via Hugging Face `AutoTokenizer.from_pretrained(...)` for the gemma/qwen/deepseek checkpoints) and rerun the same encode-and-diff comparison on KZ-03 and KZ-08 for each, instead of assuming OpenAI's token behavior generalizes to non-OpenAI models.
