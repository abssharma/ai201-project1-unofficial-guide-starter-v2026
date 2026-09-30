# The Unofficial Guide

**Abhinav Sharma → corpus: `campus_life`**

> **This file is your submission.** Fill it in as you go — most sections get
> written during the milestone that produces them, not at the end.
>
> How the starter works, and every command you'll need, is in `RUNNING.md`.
> Leave that file alone.
>
> **Paste everything as text.** No screenshots, no video. A typed table gets
> full credit; a picture of the same table gets none.
>
> Delete these instruction blocks as you replace them. The `<!-- -->` comments
> are notes to you and don't show up when the page renders — you can leave them
> or remove them.

---

# Unit 1

## What This Does

A retrieval-augmented generation (RAG) system built around the `campus_life` corpus that retrieves relevant campus documents for a user's question and uses those documents to generate an answer with source attribution. The system is designed to answer questions about campus housing, dining, courses, administration, transportation, and other campus-life topics while refusing questions that fall outside the information covered by the corpus.

## Chunking Strategy

**Chunk size:** Not fixed by character count. Each source document is kept as one chunk.
**Overlap:** None

The starter chunker used fixed 800-character windows, but the `campus_life` corpus contains 88 short, mostly self-contained documents. The documents range from 178 to 549 characters, with an average of 317 characters, so none of them need to be split into multiple chunks. After inspecting the documents and sample chunks, I chose to preserve each source document as one complete retrieval unit instead of splitting related information across arbitrary character boundaries, implementing this strategy in `chunker.py::split_documents`.

## Sample Chunks

**Chunk 1** (source: `admin_add_drop_deadline.txt#0`) | produced by: `chunker.py::split_documents`

```
On the add/drop deadline

You can add a course through the end of the second week. Dropping is a longer window — through the end of week six — but a drop after week two shows as a W on your transcript. Nothing anywhere on the registrar's site says this plainly, and students find out from each other.
```

**Chunk 2** (source: `course_biol_160.txt#0`) | produced by: `chunker.py::split_documents`

```
BIOL 160 Cell Biology 

I lived here my sophomore year. Format is lecture three times a week with a weekly lab. Assessment: four unit tests and a cumulative final. Not curved. 

Expect 9 to 11 hours a week, the heaviest first-year course by reputation. 

The one piece of advice: the unit tests come fast, roughly every three weeks; falling behind once is very hard to recover from.
```

**Chunk 3** (source: `course_hist_118_workload.txt#0`) | produced by: `chunker.py::split_documents`

```
Workload for HIST 118 Modern World History

People keep asking so: a lot of reading, about 120 pages a week, but no problem sets. That's real time, not optimistic time.

It's front-loaded — the first month is heavier than the rest, partly because you're learning the format.
```

**Chunk 4** (source: `dining_pellew_dining_hall_followup.txt#0`) | produced by: `chunker.py::split_documents`

```
Re: Pellew Dining Hall

Adding to what people have said about Pellew Dining Hall. The wait figure of 12 to 18 minutes at peak matches what I've seen. If you're trying to eat between classes, go before 11:45 and it's a different building entirely.

Also worth saying: the furthest hall from anywhere, next to the athletics centre. Nobody tells you this at orientation.
```

**Chunk 5** (source: `housing_innisfree_hall.txt#0`) | produced by: `chunker.py::split_documents`

```
Innisfree Hall — what it's actually like

Transferred in last year, so take this with a grain of salt. Built 1991, renovated 2022. Rooms are doubles arranged as pairs sharing one bathroom between two rooms.

The good: the shared-bathroom-between-two-rooms arrangement is the best compromise on campus.

The bad: no air conditioning, which matters for the first three weeks of September.

Laundry costs $1.75 wash, $1.75 dry, app-based. On noise: moderate; the building is L-shaped and the short wing is much quieter.
```

## Sample Answer

**Question:** How long should I expect the walk from Fenwick Court to central campus to take in winter?

**Answer:**
```
The walk from Fenwick Court to central campus normally takes 18 minutes, but you should add four minutes in winter because the path past the pond ices over.
```

**Sources:** `housing_fenwick_court.txt`, `transit_walking.txt`

**My relevance cutoff:** `0.6`

The retrieval distance for this question was the best among all in-scope questions, at 0.259, which is below the 0.6 relevance cutoff. The retrieved chunks included both the normal Fenwick Court walking time and the additional winter walking time. This question also demonstrates semantic difficulty, multi-document retrieval, reasoning and calculation, clear grounding, and strong relevance.

| Question | In corpus? | Best distance |
|---|---|---|
| Where on campus can I do the cheapest complete wash-and-dry laundry cycle, and how much does it cost? | In | 0.338 |
| Which residence offers the most independent living arrangement and a full kitchen? | In | 0.502 |
| If I want the shortest wait for lunch, which dining hall should I choose? | In | 0.342 |
| How long should I expect the walk from Fenwick Court to central campus to take in winter? | In | 0.259 |
| Which course has no final exam, requires a lab, and drops the lowest of three midterms? | In | 0.438 |
| What is the capital of Mongolia? | Out | 0.825 |
| How do I change the oil in a diesel engine? | Out | 0.939 |
| Who won the 1994 World Cup? | Out | 0.886 |
| What is the recommended dosage of ibuprofen for a headache? | Out | 0.844 |
| How do I write a for loop in Rust? | Out | 0.896 |

## How I Used AI

**1.** I asked Claude to help me evaluate whether the starter chunker’s fixed 800-character windows made sense for my `campus_life` corpus. After inspecting the documents, I found that they were short and mostly self-contained, ranging from 178 to 549 characters. Based on that analysis, I changed `split_documents()` to keep each source document as one complete chunk.

**2.** I asked Claude to help me evaluate my Milestone 2 questions. During Milestone 4, I found that my question "Which course has no final exam but drops the lowest midterm?" could match both STAT 150 and PHYS 130. I revised the question to include the compulsory lab and three-midterm details so that PHYS 130 was the specific expected answer.

<!-- ── Stretch features ─────────────────────────────────────────────────────
     Doing one? Say so here BEFORE you start. A feature this README never
     claims earns nothing.
     ───────────────────────────────────────────────────────────────────────── -->

---

# Unit 2

## Run Log (Before)

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 2. Every answer names a source | 5 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 3. Gate stops out-of-corpus questions | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 4. Sample chunks are 300-700 chars and don't cut a sentence | 4 of 5 | 4/5 | 4/5 | 4/5 | MET |
| 5. Answer matches `expects` | 4 of 5| 4/5 | 4/5 | 4/5 | MET |

### Real output (from `results/` log labelled `before`, 2026-09-30)

**Criterion 1: retrieved chunk contains the answer.** Produced by `store.py::search`. Run 1, Fenwick question (best distance 0.2594):

Sources retrieved: health_center.txt, housing_fenwick_court.txt, transit_shuttle.txt, transit_walking.txt, winter_gear.txt

Both files holding the answer (`housing_fenwick_court.txt`, `transit_walking.txt`) were retrieved. The other four questions also had their answer file in the top 5 in every run (e.g. `housing_morrow_house_laundry.txt`, `housing_tamsin_court.txt`, `dining_north_kitchen_followup.txt`, `course_phys_130.txt`).

Confirmed by reading the chunks. Lunch question: dining_north_kitchen_followup.txt#0 contains "The wait figure of none matches what I've seen." Laundry question: housing_morrow_house_laundry.txt#0 contains "$1.50 wash, $1.25 dry", and the other four laundry files were also in the top 5, so the "cheapest" comparison was possible from the retrieved chunks alone.

**Criterion 2: every answer names a source.** Produced by `generate.py` (answer text) using chunks from `store.py::search`. Run 1, kitchen question:

```
Tamsin Court offers the most independent housing on campus and is the only option with a full kitchen (housing_tamsin_court.txt).
```

**Criterion 3: gate stops out-of-corpus questions.** Produced by `run_eval.py::check_out_of_scope`, cutoff 0.6. Refused 5 of 5:

| Out-of-scope question | Best distance | Gate |
|---|---|---|
| What is the capital of Mongolia? | 0.825 | refused |
| How do I change the oil in a diesel engine? | 0.934 | refused |
| Who won the 1994 World Cup? | 0.886 | refused |
| What is the recommended dosage of ibuprofen for a headache? | 0.844 | refused |
| How do I write a for loop in Rust? | 0.896 | refused |

**Criterion 4: chunk length.** Produced by `chunker.py::split_documents`, measured on the five README sample chunks:

```
admin_add_drop_deadline.txt            300  (in range)
course_biol_160.txt                    379  (in range)
course_hist_118_workload.txt           274  (out of range)
dining_pellew_dining_hall_followup.txt 367  (in range)
housing_innisfree_hall.txt             516  (in range)
```
4 of 5 in range. Same number in all three columns because it is deterministic.

**Criterion 5: answer matches `expects`.** Produced by `scorer.py::judge`. A passing answer (run 1, PHYS 130 question, expects "PHYS 130"):

```
PHYS 130 Mechanics has no final exam, a compulsory lab, three midterms with the lowest dropped, and a lab practical (from course_phys_130.txt and course_phys_130_exams.txt).
```

A failing answer (run 1, Fenwick question, expects "22 minutes"; it failed in all three runs):

```
The walk from Fenwick Court to central campus takes about 18 minutes, but you should add four minutes in winter because the path past the pond ices over. 

Sources: `transit_walking.txt` and `housing_fenwick_court.txt`
```

## Verdicts

| # | Criterion | Verdict | How I decided |
|---|---|---|---|
| 1 | Retrieved chunk contains the answer (4 of 5) | MET | 5/5 in all three runs. In every question the file(s) holding the answer were in the top 5, and I opened the lunch and laundry chunks to confirm they contain the wait figure and prices. Not close. Caveat: the laundry ("cheapest") question needed several chunks together, since no single chunk states which residence is cheapest. |
| 2 | Every answer names a source (5 of 5) | MET | 15 of 15 answers (5 questions x 3 runs) named at least one source file. The target was all five, and it held every run. |
| 3 | Gate stops out-of-corpus questions (4 of 5) | MET | 5/5 refused. The worst in-scope distance was 0.502 and the best out-of-scope was 0.825, so the 0.6 cutoff sits in a wide gap. The target was safe, not tested hard. |
| 4 | Sample chunks 300-700 chars, no cut sentence (4 of 5) | MET | 4/5 in range (lengths 300, 379, 274, 367, 516). It passes by the narrowest margin: add/drop is exactly 300 and HIST 118 missed at 274. The "no cut sentence" half is trivially true because each chunk is a whole document. |
| 5 | Answer matches `expects` (4 of 5) | MET | 4/5 in all three runs. Q4 (Fenwick) failed every run because answers said "18 minutes, plus four minutes" and never "22 minutes". The target of 4 of 5 holds, but with zero slack. |

**Note:** My Unit 1 table lists the diesel-engine out-of-scope question at a best distance of 0.939, while the Unit 2 run log (`run_eval.py::check_out_of_scope`) reports 0.934. Retrieval should be deterministic, so I have left the Unit 1 section as written and am flagging the difference here. It does not affect any verdict: both values are far above the 0.6 cutoff, and the question was refused either way.

## Diagnoses

**Result:** No criterion was MISSED at its target, so there is no criterion-level miss to diagnose. One question did fail every run, and I diagnose it below. I also found that several of my targets were too easy.

### Fenwick question (Q4): failed 3 of 3 runs on the `expects` check

- **Question:** "How long should I expect the walk from Fenwick Court to central campus to take in winter?" (`expects`: "22 minutes")
- **Stage:** generation.
- **Mechanism:** Retrieval and chunking worked. `transit_walking.txt#0` was retrieved (best distance 0.259, the closest of any question) and contains both figures: "Fenwick Court to central campus: 18 minutes" and "Add four minutes in winter." No chunk states the total of 22, so the model has to add the numbers.
`GROUNDING_INSTRUCTION` in `generate.py` says nothing about combining figures and says "Be brief," which is the likely reason that in all three runs the model reported "18 minutes, plus four minutes" and never stated the sum. The answer has the facts but not the figure the question asked for. The Milestone 4 run tests this explanation.
- **Scorer caveat:** `scorer.py::judge` is an exact substring match, so part of this failure is strictness. A human would call the answer correct. But criterion 5 explicitly covers arithmetic, and this is 1 of my 2 arithmetic questions, needing the model to add 18 and 4. The other one (laundry) passed, so the weakness is specific to this case, not to arithmetic in general. It is still a real weakness in what the system produces.

### Pattern

Retrieval and chunking were not the cause of the Fenwick failure, and it is narrower than "the model can't do arithmetic". The laundry question also needs addition (wash + dry, then comparing five residences), and the model stated "$2.75" correctly in all three runs. The difference I can see is that the laundry chunks list wash and dry as two prices for one cycle, while `transit_walking.txt` gives 18 minutes as a base and "add four minutes in winter" as an adjustment, which the model reported as an adjustment without summing. That is my hypothesis from the outputs, not something the runs prove. Also, `expects` for the laundry question is "Morrow House", so the scorer never checked the total; I checked the $2.75 by reading the answers.

### Second Observation (checked, not a pipeline problem)

The output of `python app.py ask "How long should I expect the walk from Fenwick Court to central campus to take in winter?" --show-prompt` appeared to show words joined together in the chunk text, for example "wayround", "kitchenettemeans", "aboutthree", "pointof", "aweek" and "fora". I checked and could not reproduce it in the pipeline: `repr()` of the text returned by `ingest.py::load_documents` for `transit_walking.txt` shows normal spaces ("...people take the long way round." and "Ridgeway Café"), `grep -rla "wayround"` finds the string nowhere in the repo, and a `grep` of `store.py`, `app.py` and `gate.py` found nothing that rewrites chunk text. The joins were most likely an artifact of copying the terminal output. This is not a loading, chunking or retrieval defect, so I made no change and it does not affect any verdict.

```
(best distance 0.259, cutoff 0.6)

======================================================================
System instruction sent with the prompt
======================================================================
You answer questions using only the documents provided to you.

Rules:
- Use only the information in the documents below. Do not use anything you know from elsewhere.
- If the documents don't cover the question, say you don't have enough information. Do not guess.
- Name the document your answer came from, using the filename given in each excerpt.
- Be brief. Two or three sentences is usually enough.

======================================================================
The assembled prompt, exactly as sent
======================================================================
Documents:

[from transit_walking.txt]
Walking times across campus

Rough numbers, measured rather than guessed. Aldridge Hall to the science quad: 4 minutes. Fenwick Court to central campus: 18 minutes. Morrow House to Kestrel Commons: 7 minutes. Library to RidgewayCafé: 3 minutes.

Add four minutes in winter. The path past the pond genuinely ices over and people take the long wayround.

[from transit_shuttle.txt]
The campus shuttle

Runs a loop every 20 minutes from 7am to 11pm on weekdays and every 40 minutes on weekends. The published timetable is optimistic by about five minutes in the morning and accurate the rest of the day.

It's free with a student ID. The stop outside Fenwick Court is the one that gets skipped when the driver is behind, which is worth knowing if you live there.

[from housing_fenwick_court.txt]
Fenwick Court — what it's actually like

Just finished a year in this building. Built 2015. Rooms are suites of four with a shared kitchenette.

The good: in-suite bathrooms, and the kitchenettemeans you can skip a meal plan tier.

The bad: the furthest housing from central campus, about 18 minutes on foot.

Laundry costs $2.00 wash, $1.75 dry, app-based. On noise: thin walls between suites; the kitchenettes carry sound.

[from winter_gear.txt]
What winter is actually like here

Cold from mid-November to early March, with aboutthree weeks in January where it stays below freezing all day. The buildings are heated to the pointof being too warm, so layers matter more than a heavy coat.

The paths get cleared by 7am on weekdays and considerably later on weekends.

[from health_center.txt]
The health centre

Walk-in hours are 8am to 11am; everything after that is by appointment and appointments run about aweek out. If something is urgent, go at 8am and wait rather than booking.

Counselling is separate, in the same building, and has its own intake process with a shorter wait than people expect — usually three or four days fora first session.

---

Question: How long should I expect the walk from Fenwick Court to central campus to take in winter?

Answer using only the documents above, and name the file you used.
======================================================================

The walk from Fenwick Court to central campus normally takes 18 minutes, but you should add four minutes in winter because the path past the pond ices over (transit_walking.txt, housing_fenwick_court.txt).

Sources retrieved: health_center.txt, housing_fenwick_court.txt, transit_shuttle.txt, transit_walking.txt, winter_gear.txt

0 model calls this session, 1 served from cache
```

**Tests for Second Observation**

`grep -rl "wayround" corpora/campus_life/`
```
empty
```

`python -c "from ingest import load_documents; d=[x for x in load_documents() if x.source=='transit_walking.txt'][0]; i=d.text.find('long'); print(repr(d.text[i:i+30]))"`
```
'long way round.'
```

`grep -n "replace\|re.sub\|join\|split\|wrap" store.py app.py gate.py`
```
store.py:56:    Chroma's built-in embedder, wrapped to look like the other two.
app.py:35:    from chunker import split_documents, describe as describe_chunks
app.py:46:    chunks = split_documents(documents)
app.py:73:            + "\n  ".join(sources)
app.py:79:            + "\n  ".join(found)
app.py:88:        positions = [int(piece) for piece in spec.split(",") if piece.strip()]
app.py:104:    from chunker import split_documents
app.py:106:    chunks = split_documents(load_documents(args.corpus or config.CORPUS))
app.py:167:        preview = r.text[:52].replace("\n", " ")
app.py:192:    reason this is one function ratherthan two: a web wrapper that re-decided
app.py:281:    print(f"Sources retrieved: {', '.join(outcome['sources'])}\n")
```

`grep -rla "wayround" . 2>/dev/null | head`
```
empty
```

`python -c "from ingest import load_documents; d=[x for x in load_documents() if x.source=='transit_walking.txt'][0]; print(repr(d.text))"`
```
'Walking times across campus\n\nRough numbers, measured rather than guessed. Aldridge Hall to the science quad: 4 minutes. Fenwick Court to central campus: 18 minutes. Morrow House to Kestrel Commons: 7 minutes. Library to Ridgeway Café: 3 minutes.\n\nAdd four minutes in winter. The path past the pond genuinely ices over and people take the long way round.'
```

### Were my targets too easy? Yes.

Criteria 1-3 were provided in the template; criteria 4 and 5 are the two I wrote in Unit 1. I assess all five, but only 4 and 5 reflect my own target-setting.

**My own criteria:**

- **Criterion 4:** I also ran the length check over all 88 chunks (`chunker.py::split_documents`). Only 47 of 88 (53%) are between 300 and 700 characters. The other 41 are under 300, since the longest chunk is 549. My verdict stays MET because the criterion was judged on the 5 README sample chunks (4 of 5), but the full-corpus result shows it was easy to pass through sample choice. The "no cut sentence" half is always true because each chunk is a whole document. The 300-character floor was my own choice, and I set it above nearly half my documents without checking their lengths against it.
- **Criterion 5:** 4 of 5 passed with zero slack, because the one allowed failure was Fenwick. I would tighten it to 5 of 5, and change the laundry `expects` so it checks the total ($2.75) as well as the residence name.

**Provided criteria:**

- **Criterion 1:** 5/5 in every run against a target of 4 of 5, so it had slack. I judged it by reading which files were retrieved, which is a generous reading for the laundry question, where several chunks were needed together.
- **Criterion 2:** 15 of 15 answers named a source, but `GROUNDING_INSTRUCTION` already tells the model to name the file, so this was nearly guaranteed. It could be tightened to "names the correct source file".
- **Criterion 3:** the worst in-scope distance (0.502) and the best out-of-scope distance (0.825) are far apart, so 4 of 5 was very safe. It could be tightened to 5 of 5 refused, using out-of-scope questions that share vocabulary with campus life, since the provided five were from entirely different worlds and were never going to come close to the cutoff.

## The Improvement

**What I changed:** Added one rule to `GROUNDING_INSTRUCTION` in `generate.py`: "If the question needs figures from the documents added or compared, state the final result explicitly and show the calculation." Nothing else changed: same chunks, same index, same cutoff (0.6), same top-k (5), same questions and scorer.

**Why I picked it:** The Fenwick question failed all three runs at the generation stage. Retrieval returned `transit_walking.txt` with both figures (18 minutes, plus four in winter), but the instruction never told the model to combine them and asked it to be brief, so it never stated 22 minutes.

### Run Log (After)

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 2. Every answer names a source | 5 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 3. Gate stops out-of-corpus questions | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 4. Sample chunks are 300-700 chars and don't cut a sentence | 4 of 5 | 4/5 | 4/5 | 4/5 | MET |
| 5. Answer matches `expects` | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |

### Real output (from `results/` log labelled `after`, 2026-09-30)

**Fenwick question, run 3 (failed 3 of 3 before, passed 3 of 3 after):**

```
The walk from Fenwick Court to central campus normally takes 18 minutes, but you should add four minutes in winter because the path past the pond ices over (transit_walking.txt, housing_fenwick_court.txt). 

Calculation: 18 minutes + 4 minutes = 22 minutes. 
Final result: 22 minutes.
```

**Laundry question, run 1 (the rule also applies here):**

```
Based on the documents provided, the cheapest complete wash-and-dry laundry cycle is in Morrow House, where it costs $2.75 ($1.50 wash + $1.25 dry). 

Calculation: $1.50 (wash) + $1.25 (dry) = $2.75.

This information comes from `housing_morrow_house_laundry.txt`.
```

**Did it help?** Yes, on the one failure I targeted. Fenwick went from 0 of 3 to 3 of 3, so criterion 5 went from 4/5 to 5/5 in every run. Retrieval distances and retrieved sources were identical to Before, so the change came from the prompt alone. That fits my diagnosis that the model had the two figures and was not told to combine them.

This is limited evidence: one question, three runs, one prompt change. Some things I noticed:
- The rule is followed inconsistently. Fenwick showed the sum in all three runs, in three different formats (inline "18 + 4 = 22", a "Calculation:" block, and "Final result:"). Laundry showed an explicit calculation only in run 1; runs 2 and 3 just stated $2.75.
- Nothing else got worse. The other four questions still passed, all 15 answers still name a source, and laundry still gives $2.75 and Morrow House. Answers for kitchen, lunch and PHYS 130 are worded slightly differently, which is normal variation between runs.
- The scorer is still a substring match, so the passes show "22 minutes" appeared, not that the arithmetic was shown in a consistent way. A phrasing like "22 min" would still fail.
- Criterion 5 now passes 5/5 but still only for the reason I diagnosed. It says nothing about arithmetic questions I have not written.

## What's Still Broken

No criterion is MISSED after the fix (all five MET, criterion 5 now 5/5). What remains is weakness in how I measured, and in how much the fix can be trusted.

- **The fix is only tested on one question.** Fenwick went from 0/3 to 3/3, but that is one question and three runs. I don't know whether the rule generalises to other "base plus adjustment" questions. I would write three or four more questions including arithmetic calculations and run them.
- **The rule is applied inconsistently.** In the After log, Fenwick showed the sum in all three runs but in three different formats. Laundry showed an explicit calculation only in run 1. Nothing forces the model to show its working. I would play around by tightening the instruction wording or test a fixed output format, and measure again.
- **The scorer is a substring match.** `scorer.py::judge` would fail a correct answer that said "22 min", and for laundry it checks only "Morrow House", so a wrong total would still pass. I checked $2.75 by reading the answers, not by scoring. What I'd do: make `expects` check the total as well as the name.
- **Criterion 4 measures very little.** Only 47 of 88 chunks are between 300 and 700 characters, yet it was MET on a 5-chunk sample, and the "no cut sentence" half is always true because each chunk is a whole document.
- **The answers got a bit longer for some runs**, against the "two or three sentences" instruction, because of the added calculation lines.

## What I'd Do Differently

**Criterion 4 (mine):** I'd measure all chunks instead of a 5-chunk sample, and drop the 300-character floor, which sits near my corpus mean (317) so nearly half the documents fall below it. I'd replace it with something that can fail, such as "every chunk contains at least one complete fact a question could be answered from."

**Criterion 5 (mine):** I'd tighten the target to 5 of 5 and make `expects` stricter for the arithmetic questions: "22 minutes" plus the shown calculation for Fenwick, and "$2.75" as well as "Morrow House" for laundry. I'd also add more arithmetic questions, since I only had two.

**Provided criteria (briefly):** Criterion 2 would say "names the correct source file", because the grounding instruction already tells the model to name a file. Criterion 3 would use out-of-scope questions that share vocabulary with campus life.

## How I used AI (Unit 2)

In Unit 2, I used Claude to help diagnose the Fenwick failure. It first told me the two figures (18 and +4) were in different documents. When I printed the prompt with `--show-prompt`, both were in `transit_walking.txt`, so I corrected my diagnosis. Moreover, it suggested several explanations for the joined words in the prompt (`ingest.py`, a stale index, hidden characters). I tested each with `grep` and `repr()`, and none held up, so I concluded it was a copy artifact and made no change.
