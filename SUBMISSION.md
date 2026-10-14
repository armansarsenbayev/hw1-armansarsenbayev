# HW1 submission

**Name:** Arman Sarsenbayev
**Student ID:** s23069008
**Group:** CSS4007-ENG-8
**Repository:** https://github.com/armansarsenbayev/hw1-armansarsenbayev

## AI tool disclosure

State which AI tools you used and for what. Expected and fine; undisclosed use
is not.

> I used Claude Code (Anthropic) to help implement the TODO functions in the
> three sublabs, to run the scripts, and to draft the written answers from the
> numbers in my tables. I reviewed the code, re-ran the scripts myself, and
> edited the answers. All numbers below are copied from the program output
> saved in `outputs/` (which is gitignored).

**Gemma: free vs paid route.** `google/gemma-4-26b-a4b-it:free` returned HTTP
429 on every call for ~20 minutes (12 retries with backoff, then again later):

```
Error code: 429 - 'google/gemma-4-26b-a4b-it:free is temporarily rate-limited
upstream. Please retry shortly, or add your own key ...'
'provider_name': 'Google AI Studio', 'limit_source': 'upstream_provider_shared_pool'
```

The paid models on the same key worked at the same moment, so it was the free
pool, not my key. I therefore ran the **paid route to the same weights,
`google/gemma-4-26b-a4b-it`** ($0.0765 in / $0.255 out per 1M, from
OpenRouter's `/models` on 2026-10-14; added to `RATES_PER_MTOK`). Every Gemma
number below is from that paid route.

---

## Sublab Easy — the registration bot and its bill

**How I laid the catalogue out inside the system prompt, and why:**

> One labelled block per course (code + title, then `credits`, `prerequisites`,
> `meets`, `seats`, `instructor` on separate lines), then the student, then the
> rules, then "how to answer" instructions. I computed the derived facts in
> Python instead of leaving them to the model: seats left, a `FULL - cannot
> register` marker, and `STUDENT ALREADY PASSED THIS COURSE` on the four
> completed courses. Arithmetic like 60 − 60 is exactly what a model gets
> wrong, so I don't ask it to do it. The refusal rule ("the catalogue is the
> only source of truth … never invent a course") is placed *before* the
> catalogue and repeated in the rules.

**My turn 5 (Kazakh or Russian):**

> Мен үшінші курс студентімін. Қандай курстарға әлі тіркеле аламын?

### Run 1 — OpenAI, `gpt-5.6-luna`

| Turn | Input tokens | Output tokens | Cost $ |
|---|---|---|---|
| 1 | 872 | 593 | 0.000886 |
| 2 | 1173 | 151 | 0.000416 |
| 3 | 1294 | 66 | 0.000338 |
| 4 | 1379 | 29 | 0.000311 |
| 5 | 1436 | 392 | 0.000758 |
| **total** | 6154 | 1231 | 0.002708 |

### Run 2 — OpenRouter, `google/gemma-4-26b-a4b-it` (paid route, see above)

| Turn | Input tokens | Output tokens | Cost $ |
|---|---|---|---|
| 1 | 1005 | 287 | 0.000150 |
| 2 | 1318 | 150 | 0.000139 |
| 3 | 1501 | 76 | 0.000134 |
| 4 | 1591 | 28 | 0.000129 |
| 5 | 1650 | 391 | 0.000226 |
| **total** | 7065 | 932 | 0.000778 |

### Turn 4, verbatim

The turn where you asked for CSS-4090, which does not exist. Paste both replies
exactly as they came back — do not tidy them.

**OpenAI:**

```
**CSS-4090 Quantum Machine Learning does not exist in the 2026-FALL catalogue**, so I cannot register it.
```

**OpenRouter:**

```
I cannot add **CSS-4090 Quantum Machine Learning** to your schedule. This course does not exist in the catalogue.
```

### Written answers

**1. The two providers used almost identical code. What actually changed, and
what did not?**

> What changed: the `base_url` (`https://openrouter.ai/api/v1`), the API key
> (`OPENROUTER_API_KEY`), and the model name, which on OpenRouter carries a
> vendor prefix (`google/…`). What did not change: the `openai` library, the
> `OpenAI` class, the `chat.completions.create` call, the message format
> (`system`/`user`/`assistant`), and where the reply and token counts live in
> the response (`choices[0].message.content`, `usage.prompt_tokens`,
> `usage.completion_tokens`). OpenRouter speaks the OpenAI wire format, so
> switching provider is configuration, not code.

**2. Why did the input token count climb on every turn when your questions
stayed roughly the same length? Use the numbers from your own table. What
happens to the bill at fifty turns?**

> The model is stateless, so `run_turn` resends the whole history every call:
> system prompt + every earlier question + every earlier answer. With luna the
> input went 872 → 1173 → 1294 → 1379 → 1436. Turn 1 is ~855 tokens of system
> prompt plus a 17-token question. Each later jump is roughly the previous
> answer plus the new question: +121 after a 151-token answer, +85 after 66,
> +57 after 29. The first jump (+301) is smaller than turn 1's 593 output
> tokens, because only the visible reply goes back into the history — the rest
> of the 593 was (my estimate) hidden reasoning, billed once as output and
> never re-sent.
>
> At fifty turns each call carries all the previous ones, so input per turn
> grows linearly and the *total* input grows roughly quadratically. With an
> ~850-token system prompt and ~140 tokens added per turn, 50 turns would send
> about 50·850 + 140·(1+2+…+49) ≈ 42k + 172k ≈ 214k input tokens — about 35×
> what these five turns cost, not 10×, for the same kind of questions.
>
> Gemma shows the same climb: 1005 → 1318 → 1501 → 1591 → 1650. Here each jump
> is almost exactly the previous reply plus the new question: +313 after 287
> output tokens, +183 after 150, +90 after 76, +59 after 28. Gemma has no
> hidden reasoning, so its whole output is re-sent — which supports the
> explanation of luna's smaller first jump above. Gemma also starts higher
> (1005 vs 872) on the same system prompt: a different tokenizer.

**3. Turn 4: did the bot refuse, or did it invent CSS-4090?** If it refused, what
in your system prompt held the line? If it invented, what did it make up —
credits, a room, an instructor?

> luna refused: "CSS-4090 Quantum Machine Learning does not exist in the
> 2026-FALL catalogue, so I cannot register it." It invented nothing. What held
> the line, I think, is that the prompt says the catalogue is *the only source
> of truth*, gives the explicit count "(8 courses)", and spells out what not to
> invent (course, credits, room, schedule, instructor). Gemma also refused:
> "I cannot add **CSS-4090 Quantum Machine Learning** to your schedule. This
> course does not exist in the catalogue." No invented credits, room or
> instructor. Both bots held the line, so for this prompt the refusal rule
> was strong enough even for the smaller model.

**4. Where else was either bot wrong?** Turn 2 asks for two courses that meet at
the same hour; two courses in the catalogue are full. Did the bots notice?

> luna noticed both traps. Turn 1: it excluded CSS-4400 (full) and the four
> passed courses (including CSS-3011, which is also full), and flagged that
> CSS-4007 and CSS-4102 both meet Tuesday 09:00–10:50. Turn 2: it refused the
> registration because of that collision. One thing to question: in turn 2 it
> also cited "11 credits, below the minimum of 15" as a reason to refuse. The
> minimum is a term-load rule, not a per-request rule, so that is a stricter
> reading than the catalogue states. Turn 3's arithmetic (6 + 5 = 11, within
> 30, below 15) was correct.
>
> **Gemma missed the Tuesday collision completely.** Turn 1 listed CSS-4007,
> CSS-4102 and FIN-3300 as eligible and then *recommended* "CSS-4007 +
> CSS-4102 + FIN-3300 = 16 credits" — a timetable that cannot exist, since
> CSS-4007 and CSS-4102 both meet Tuesday 09:00–10:50. In turn 2 it checked
> each course and wrote "no collision" for both, and rejected the request
> only because 11 < 15 credits. So it got the right answer (reject) for the
> wrong reason. The collision was in the prompt as two `meets:` lines; I
> never computed it for the model the way I computed FULL and PASSED, and
> the one fact I left for it to derive is the one it got wrong. Gemma did
> handle the full courses (CSS-4400 "FULL", CSS-3011 excluded as passed) and
> the arithmetic (6 + 5 = 11). Turn 5 in Kazakh repeated the turn-1 answer
> but the language leaked: "Бұл курстарды сіз әу already passed (өттіңіз)"
> and "План толып тұр (FULL)" — English and Russian mixed into the Kazakh.

---

## Sublab Medium — one task, six models

Paste the per-model summary printed by `correct_kazakh.py`:

| Model | Exact | Failed | Tokens | Cost $ |
|---|---|---|---|---|
| google/gemma-4-26b-a4b-it (paid route) | 4 | 0 | 34817 | 0.00861 |
| qwen/qwen3.8-27b | 7 | 0 | 141802 | 0.44868 |
| deepseek/deepseek-v4-flash-0731 | 8 | 0 | 33058 | 0.00898 |
| gpt-5.6-luna | 6 | 0 | 5664 | 0.00542 |
| gpt-5.6-terra | 7 | 0 | 2757 | 0.01936 |
| gpt-5.6-sol | 7 | 0 | 3130 | 0.05960 |

Input / output tokens for the same eight prompts:

| Model | Input | Output |
|---|---|---|
| gemma | 1,492 | 33,325 |
| qwen | 1,850 | 139,952 |
| deepseek | 1,939 | 31,119 |
| luna / terra / sol | 1,372 each | 4,292 / 1,385 / 1,758 |

The OpenAI models all got the same 1,372 input tokens (same tokenizer). The
three others counted the *same prompts* as 1,492–1,939 — each has its own
tokenizer (and chat template). Output is where the money went:

- **qwen** averaged ~17,500 output tokens per sentence (up to 44,536 on
  KZ-08) for a one-line answer — almost all of it reasoning. That is why it
  cost $0.45, 50–80× the others, and alone blew the "under $0.10" estimate
  for the assignment.
- **gemma** spent 32,768 of its 33,325 output tokens on **one** sentence,
  KZ-05: it ran to the output cap and returned the sentence unchanged. That
  single runaway call was $0.0084 of its $0.0086. The other seven cost
  ~$0.0002 together.
- **luna** produced ~3× terra's output for the same visible JSON — reasoning
  tokens, billed as output.

### Per sentence (6 models × 8 sentences)

`E` = exact match. Otherwise the `char_diff` and my own judgement:
✓ = correct Kazakh, ~ = errors fixed but text changed beyond them,
✗ = not repaired / broken.

| Sentence | Errors | gemma | qwen | deepseek | luna | terra | sol |
|---|---|---|---|---|---|---|---|
| KZ-01 | kaz_to_rus | 43 ~ | E | E | 43 ~ | E | E |
| KZ-02 | drop_hyphen | E | E | E | E | E | E |
| KZ-03 | latin_homoglyph | 35 ✗ | E | E | E | E | E |
| KZ-04 | kaz_to_rus (partial) | 1 ✓ | E | E | E | 1 ✓ | 1 ✓ |
| KZ-05 | join_words | 32 ✗ | E | E | 1 ✓ | E | E |
| KZ-06 | double_letter | E | E | E | E | E | E |
| KZ-07 | kaz_to_rus + join_words | E | E | E | E | E | E |
| KZ-08 | latin_homoglyph + double_letter | E | 1 ~ | E | E | E | E |
| **exact** | | 4 | 7 | 8 | 6 | 7 | 7 |
| **judged correct** | | 4 ✓ + 1 ~ | 7 ✓ + 1 ~ | 8 | 7 ✓ + 1 ~ | 8 | 8 |

### Which error types did each model repair?

Rows are error labels, columns are models. Write "yes", "no" or "partial".

| Error type | gemma | qwen | deepseek | luna | terra | sol |
|---|---|---|---|---|---|---|
| kaz_to_rus | partial | yes | yes | partial | yes | yes |
| latin_homoglyph | partial | yes | yes | yes | yes | yes |
| drop_hyphen | yes | yes | yes | yes | yes | yes |
| join_words | partial | yes | yes | yes | yes | yes |
| double_letter | yes | yes | yes | yes | yes | yes |

- **gemma, kaz_to_rus = partial:** KZ-04 and KZ-07 fixed; KZ-01 restored all
  the letters but, exactly like luna, pluralised *мемлекеттің елшісінен* →
  *мемлекеттердің елшілерінен*. Its change list also claims edits it never
  made ("replaced а with ә in биркатар").
- **gemma, join_words = partial:** KZ-07 fixed, KZ-05 not — the runaway
  generation above, and the returned `changes` says "split
  'ғимараттардыңқаншасы' into 'ғимараттардыңқаншасы'", i.e. nothing.
- **qwen, KZ-08:** the homoglyphs and the double л were fixed, but it also
  "corrected" *Ким* → *Кім*. Ким here is Kim Jong Un's surname, not a word
  with a Russian и — an over-correction of a proper name. Its change list
  also reports "replaced г with к in кездескісі", which it did not do.
- **deepseek:** 8/8 exact, no over-edits.

**The `latin_homoglyph` row: what happened?** Describe what you observed. The
explanation is Sublab Harder's job, not this one's.

> Five of six models repaired both homoglyph rows (KZ-03 and KZ-08). luna's
> change list even named them: "латынша A кириллше А-ға ауыстырылды",
> "латынша t кириллше т-ға ауыстырылды".
>
> **Gemma is the one that broke on it.** On KZ-03 it returned
> *Алақтарға* for *Алаяқтарға* — it replaced the Latin letters but **lost the
> я**, turning "scammers" into a non-word. Its change list says "Corrected
> 'Aлaяқtарға' to 'Алақтарға' by replacing Latin/Russian letters with Kazakh
> letters", so it did see that the word was mixed-script, but could not
> rebuild it. It also added a comma and a question mark nobody asked for.
> Interestingly, on KZ-08 (*Дoнaлльд Tрамп*) the same model was exact —
> a famous name is easy to recover from context even when the letters are
> scrambled; a less common word like *алаяқ* is not. It is the only sentence
> in the whole table where a model damaged the word it was fixing.

**Where a model returned good Kazakh that was not identical to the original,
say so here.** Exact match is not correctness.

> - luna KZ-05, terra/sol/gemma KZ-04: identical except an added `?` at the
>   end. These are questions, so the `?` is fine Kazakh — counted as repaired.
> - luna and gemma, KZ-01: all the Russian letters were restored correctly,
>   but both also changed *мемлекеттің елшісінен* ("from the ambassador of a
>   state") to *мемлекеттердің елшілерінен* ("from the ambassadors of states").
>   That is grammatical Kazakh, and arguably better matches "бірқатар"
>   (several), but it changed the text beyond the letter errors it was asked
>   to fix — the prompt said "do not rephrase". Marked "~"/partial.
> - qwen, KZ-08: *Кім* instead of *Ким* — one character, but it is wrong
>   (a proper name), not a stylistic variant. Marked "~".

**Cheapest model that was good enough, and why:**

> **deepseek-v4-flash, $0.0090 for all eight.** It is the only model that was
> exact on all eight sentences with no over-edits. Strictly cheaper ones:
> luna ($0.0054) fixed every error type but rewrote KZ-01 against the
> instruction, and gemma ($0.0086) broke KZ-03 and failed KZ-05 — not good
> enough. If a single over-edit is acceptable, luna is the cheapest; for a
> correction tool that must "change only what is broken", deepseek is worth
> the extra $0.0036. The price ranking is also not the quality ranking: qwen
> cost 50× deepseek and was worse; sol cost 6.6× deepseek for the same result.
> Note that deepseek still produced ~3,900 output tokens per sentence — it is
> cheap because its *rate* is low ($0.28/M out), not because it is brief.

---

## Sublab Harder — open the tokenizer

Sentence averages: kk 43.8 chars, ru 46.2, en 48.5 (6 sentences each).
$ per 1,000 sentences is at `gpt-5.6-sol`'s input rate, $5.00 / 1M tokens.

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
(`o200k_base`.)

| Sentence id | Foreign char (index, name) | Tokens correct | Tokens corrupted | Δ | Diverges at |
|---|---|---|---|---|---|
| KZ-03 | 0 `A` LATIN CAPITAL LETTER A; 2 `a` LATIN SMALL LETTER A; 5 `t` LATIN SMALL LETTER T | 16 | 20 | +4 | 0 |
| KZ-08 | 1 `o` LATIN SMALL LETTER O; 3 `a` LATIN SMALL LETTER A; 9 `T` LATIN CAPITAL LETTER T | 21 | 24 | +3 | 1 |

**Token pieces around the divergence:**

```
KZ-03, o200k_base
correct  : ['А', 'лая', 'қ', 'тарға', ' ақша', 'ңызды', ' ауд', 'арып']
corrupted: ['A', 'л', 'a', 'я', 'қ', 't', 'ар', 'ға', ' ақша', 'ңызды']

KZ-08, o200k_base  (also has a doubled л)
correct  : ['Д', 'он', 'аль', 'д', ' Т', 'рамп', ' К', 'им']
corrupted: ['Д', 'o', 'н', 'a', 'л', 'ль', 'д', ' T', 'рамп', ' К', 'им']

For contrast, KZ-01 (kaz_to_rus), o200k_base — 28 -> 26 tokens
correct  : ['Қ', 'ас', 'ым', '-', 'Ж', 'ом', 'арт', ' То', 'қа', 'ев', ' бір', 'қ']
corrupted: ['К', 'ас', 'ым', '-', 'Ж', 'ом', 'арт', ' Т', 'ока', 'ев', ' бир', 'к', 'атар']

cl100k_base, KZ-03 — 44 -> 45 tokens ('�' = half of a Cyrillic letter's UTF-8 bytes)
correct  : ['А', 'ла', 'я', '�', '�', 'т', 'ар', '�', '�', 'а']
corrupted: ['A', 'л', 'a', 'я', '�', '�', 't', 'ар', '�', '�', 'а']
```

### C. Did it get better?

| Language | cl100k_base | o200k_base | Change |
|---|---|---|---|
| kk | 0.760 | 0.319 | −58% |
| ru | 0.466 | 0.267 | −43% |
| en | 0.203 | 0.203 | 0% |

### Written answers

**1. What is the Kazakh tax?** The ratio against English in both encodings, the
dollar figure from A, and how much it changed between the two tokenizers.

> Per character, Kazakh cost 3.75× English under `cl100k_base` and 1.58× under
> `o200k_base`. In money at sol's input rate: $0.1667 vs $0.0492 per 1,000
> sentences with cl100k, $0.0700 vs $0.0492 with o200k. The newer tokenizer cut
> Kazakh's tokens per char by 58% (0.760 → 0.319) while English did not move at
> all (0.203 both) — so the gap shrank from 3.75× to 1.58×, but it is not
> gone. It also showed up in my own bot: turn 5 (Kazakh, 65 chars) is 21 tokens
> in o200k and 48 in cl100k; turn 1 (English, 77 chars) is 17 in both.
> Russian sits in between (2.30× → 1.32×), so the tax is partly "Cyrillic" and
> partly "Kazakh-specific letters".

**2. Why did the models repair `kaz_to_rus` but struggle with
`latin_homoglyph`?** Both are single-letter substitutions and both look almost
identical on screen. Use your token streams from B as the evidence. Say what the
model actually received in each case.

> In `kaz_to_rus` the model receives *ordinary Cyrillic tokens* of a
> Russian-looking word: `' Т','ока','ев'`, `' бир','к','атар'`. The token count
> even went *down* (28 → 26) because the Russianised text is closer to what the
> tokenizer saw in training. These are well-trained tokens, and "Kazakh word
> spelled on a Russian keyboard" is a pattern the model has seen a lot, so
> mapping `бир` → `бір` is a familiar step.
>
> In `latin_homoglyph` the model receives something different: the word is
> broken into single-letter fragments from two scripts. `Алаяқтарға` was
> `'А','лая','қ','тарға'` (4 tokens); with three Latin letters it became
> `'A','л','a','я','қ','t','ар','ға'` (8 tokens). `Дональд` went from
> `'Д','он','аль','д'` to `'Д','o','н','a','л','ль','д'`. The model never sees
> the word as a unit — it gets a sequence of Latin `a`/`t`/`o` tokens (which
> it knows from English) interleaved with lone Cyrillic letters. Nothing on
> screen tells you this, but the ids are completely different from the first
> token on (divergence at index 0 and 1).
>
> What I actually observed: all three OpenAI models still fixed both homoglyph
> sentences. So for strong models the fragmented stream was recoverable —
> probably because they reassemble the letters from context and the
> substitution touches only 3 letters. The tokenizer explains why it is
> *harder* (more tokens, no whole-word tokens, mixed scripts); it does not mean
> every model fails.
>
> The one model that did fail is the smallest one: Gemma (26B total, ~4B
> active parameters) turned *Aлaяқtарға* into *Алақтарға* — it swapped the
> Latin letters back but dropped the я. That is what you would expect if the
> model has to rebuild the word from fragments (`'A','л','a','я','қ','t',…`
> in o200k) instead of recognising one familiar token like `'лая'`: put the
> fragments back slightly wrong and you get a plausible-looking non-word.
> Gemma got *Дoнaлльд Tрамп* exactly right, though — the context
> ("Трамп", "Ким Чен Ын") pins down the famous name even when its letters
> are fragmented. Caveat: these are o200k streams, not Gemma's own tokenizer
> (see question 3), so this is a plausible mechanism, not a measured one.

**3. Name one thing this measurement does not explain about your Sublab Medium
results.** You measured OpenAI's tokenizers; three of your six models were not
OpenAI's. What follows, and what would you have to do to close the gap?

> Gemma, Qwen and DeepSeek each have their own tokenizer (different vocabulary
> and merges), so my o200k token streams are not what those three models were
> given. Their Kazakh tax and the way a homoglyph splits a word could be better
> or worse than OpenAI's, so I cannot use this sublab to explain *their* rows
> in the Medium table. To close the gap I would load each model's own tokenizer
> (e.g. from Hugging Face with `transformers.AutoTokenizer` for the open-weight
> Gemma/Qwen/DeepSeek checkpoints) and repeat measurements A and B, or compare
> against the `usage.prompt_tokens` OpenRouter reports for the same prompt.
>
> My own Medium numbers already show the gap is real: the *same eight
> prompts* were 1,372 input tokens for every OpenAI model, but 1,492 for
> Gemma, 1,850 for Qwen and 1,939 for DeepSeek (as reported by OpenRouter,
> which also includes each model's chat-template overhead). Different
> tokenizers, different counts — so the o200k pieces I pasted in B are not
> the pieces Gemma saw when it broke KZ-03.
>
> The measurement also says nothing about **output** cost, which dominated
> Medium: qwen's 139,952 output tokens and gemma's single 32,768-token runaway
> on KZ-05 are about how a model reasons and when it stops, not about how the
> input was split. It also does not explain differences *between* luna, terra and sol: they
> share the same tokenizer (identical 1,372 input tokens) yet differ in
> quality — that comes from model size and reasoning, not tokenization.
