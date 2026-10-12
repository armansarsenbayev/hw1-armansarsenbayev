# HW1 submission

**Name:** Arman Sarsenbayev
**Student ID:** TODO
**Group:** TODO
**Repository:** TODO

## AI tool disclosure

State which AI tools you used and for what. Expected and fine; undisclosed use
is not.

> I used Claude Code (Anthropic) to help implement the TODO functions in the
> three sublabs, to run the scripts, and to draft the written answers from the
> numbers in my tables. I reviewed the code, re-ran the scripts myself, and
> edited the answers. All numbers below are copied from the program output
> saved in `outputs/` (which is gitignored).

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

### Run 2 — OpenRouter, `google/gemma-4-26b-a4b-it:free`

TODO — needs `OPENROUTER_API_KEY`.

| Turn | Input tokens | Output tokens | Cost $ |
|---|---|---|---|
| 1 | | | |
| 2 | | | |
| 3 | | | |
| 4 | | | |
| 5 | | | |
| **total** | | | |

### Turn 4, verbatim

The turn where you asked for CSS-4090, which does not exist. Paste both replies
exactly as they came back — do not tidy them.

**OpenAI:**

```
**CSS-4090 Quantum Machine Learning does not exist in the 2026-FALL catalogue**, so I cannot register it.
```

**OpenRouter:**

```
TODO
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

**3. Turn 4: did the bot refuse, or did it invent CSS-4090?** If it refused, what
in your system prompt held the line? If it invented, what did it make up —
credits, a room, an instructor?

> luna refused: "CSS-4090 Quantum Machine Learning does not exist in the
> 2026-FALL catalogue, so I cannot register it." It invented nothing. What held
> the line, I think, is that the prompt says the catalogue is *the only source
> of truth*, gives the explicit count "(8 courses)", and spells out what not to
> invent (course, credits, room, schedule, instructor). Gemma: TODO.

**4. Where else was either bot wrong?** Turn 2 asks for two courses that meet at
the same hour; two courses in the catalogue are full. Did the bots notice?

> luna noticed both traps. Turn 1: it excluded CSS-4400 (full) and the four
> passed courses (including CSS-3011, which is also full), and flagged that
> CSS-4007 and CSS-4102 both meet Tuesday 09:00–10:50. Turn 2: it refused the
> registration because of that collision. One thing to question: in turn 2 it
> also cited "11 credits, below the minimum of 15" as a reason to refuse. The
> minimum is a term-load rule, not a per-request rule, so that is a stricter
> reading than the catalogue states. Turn 3's arithmetic (6 + 5 = 11, within
> 30, below 15) was correct. Gemma: TODO.

---

## Sublab Medium — one task, six models

Paste the per-model summary printed by `correct_kazakh.py`:

| Model | Exact | Failed | Tokens | Cost $ |
|---|---|---|---|---|
| google/gemma-4-26b-a4b-it:free | TODO | | | |
| qwen/qwen3.8-27b | TODO | | | |
| deepseek/deepseek-v4-flash-0731 | TODO | | | |
| gpt-5.6-luna | 6 | 0 | 5664 | 0.00542 |
| gpt-5.6-terra | 7 | 0 | 2757 | 0.01936 |
| gpt-5.6-sol | 7 | 0 | 3130 | 0.05960 |

All three OpenAI models got the same 1,372 input tokens; output was 4,292
(luna), 1,385 (terra), 1,758 (sol). luna's output is ~3× the others even though
its visible JSON is the same size — the extra are reasoning tokens, billed as
output.

### Which error types did each model repair?

Rows are error labels, columns are models. Write "yes", "no" or "partial".

| Error type | gemma | qwen | deepseek | luna | terra | sol |
|---|---|---|---|---|---|---|
| kaz_to_rus | TODO | TODO | TODO | partial | yes | yes |
| latin_homoglyph | TODO | TODO | TODO | yes | yes | yes |
| drop_hyphen | TODO | TODO | TODO | yes | yes | yes |
| join_words | TODO | TODO | TODO | yes | yes | yes |
| double_letter | TODO | TODO | TODO | yes | yes | yes |

**The `latin_homoglyph` row: what happened?** Describe what you observed. The
explanation is Sublab Harder's job, not this one's.

> All three OpenAI models repaired both homoglyph rows exactly (KZ-03 and
> KZ-08, which also had a doubled letter). luna's change list even named them:
> "латынша A кириллше А-ға ауыстырылды", "латынша t кириллше т-ға
> ауыстырылды". So on OpenAI models the homoglyph row was *not* the failure I
> expected. TODO: what the three OpenRouter models did.

**Where a model returned good Kazakh that was not identical to the original,
say so here.** Exact match is not correctness.

> - luna, KZ-05 and terra/sol, KZ-04: identical except an added `?` at the end.
>   These are questions, so the `?` is fine Kazakh — counted as repaired.
> - luna, KZ-01: all the Russian letters were restored correctly, but it also
>   changed *мемлекеттің елшісінен* ("from the ambassador of a state") to
>   *мемлекеттердің елшілерінен* ("from the ambassadors of states"). That is
>   grammatical Kazakh, and arguably better matches "бірқатар" (several), but it
>   changed the text beyond the letter errors it was asked to fix. Marked
>   "partial".

**Cheapest model that was good enough, and why:**

> Among the OpenAI models: luna at $0.0054 for all eight. It repaired every
> error type; its only miss was an over-edit on KZ-01. terra was cleaner (no
> rephrasing) but cost 3.6× more, and sol 11× more for the same 7/8 exact.
> TODO: revisit once gemma (free) and deepseek (cheapest paid) are in.

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
> every model fails. TODO: compare with gemma/qwen/deepseek.

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
> It also does not explain differences *between* luna, terra and sol: they
> share the same tokenizer (identical 1,372 input tokens) yet differ in
> quality — that comes from model size and reasoning, not tokenization.
