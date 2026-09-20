# The Unofficial Guide

Lloani Mendez 
Corpus: campus_life

<!--> **This file is your submission.** Fill it in as you go — most sections get
> written during the milestone that produces them, not at the end.
>
> How the starter works, and every command you'll need, is in `RUNNING.md`.
> Leave that file alone.
>
> **Paste everything as text.** No screenshots, no video. A typed table gets
> full credit; a picture of the same table gets none.
>
> Delete these instruction blocks as you replace them. The `` comments
> are notes to you and don't show up when the page renders — you can leave them
> or remove them.-->

---

# Unit 1

## What This Does

<!-- Three or four sentences. Which corpus you picked, and the kinds of
     questions your system answers. Write it for someone who has never seen
     this repo.

     Milestone 5. -->
For this project I choose to focus on the campus_life corupus, which answers questions a college student may ask about the new campus they have moved to. It can range from housing and dining questions to course information where you would like to ask a student. For example, "Is there a penalty for declaring my major late?". The system searches through 88 documents, answering using only the ones it retrieves, and names the files each answer came from. If the documents do not cover the question, it refuses instead of guessing. 

## Chunking Strategy

**Chunk size:** one document - no fixed character count (317 characters on average, shortest 178, longest 549)

Every document in the corpus is a title line, a blank line, then two or three short paragraphs with one or two sentences each. Each chunk keeps its document's title line, because the answers to my test questions live in the body and the body often never repeats the topic word. Like in `transit_shuttle.txt` where it says "the published timetable is optimistic by about five minutes" without ever saying "shuttle". Without the title, that chunk doesn't come back for a shuttle question.

**Overlap:** None

There is nothing for overlap to rescue. Chunks end where the document ends, so no sentence and no thought gets cut in half. Overlap would only pull in facts from a different post and shift what the chunk is about.

I changed my mind twice here, and the second time is the one that mattered.

I originally wrote 264 as a chunk size because the documents are short and each answer sits in a single sentence, so I thought a tight character window would keep the chunk focused. Anything extra would be a different fact that could pull the answer off course. But as I went along with writing the code I figured it would be more efficient to split by paragraphs. This would actually reduce that chance of the sentence being cut in half, so the size and overlap knobs stopped applying.

Then I printed five chunks and saw that splitting on the paragraphs caused a problem of not having enough context. For example, `housing_morrow_house.txt` resulted in "The good: cheapest housing tier by about $900 a year, and the singles are real singles" as a chunk. It did not have context of "the bad" about the place so it would only ever get one side of it. A dining followup chunk opened with "Also worth saying" which is a continuation of a sentence the retriever would never see just cause this was considered another paragraph. My Milestone 1 read said each paragraph was a self contained fact, and reading the chunks themselves is what showed me that wasn't true. 

So I measured the corpus instead of guessing: 88 documents, median 309 characters, longest 549. Nothing in it is long enough to need splitting at all, so I made the rule one post, one chunk.

Two downsides. No splitting means the few larger documents carry more information than a single question needs like `housing_innisfree_hall.txt` now covers the bathroom arrangement, air conditioning, laundry prices and noise in one chunk. The other is that the seven `_followup` files share nearly all their wording, differing only in the hall name and two facts, so a general dining question could retrieve an arbitrary one of them.

<!-- What about YOUR documents made you pick these numbers? Short posts and
     long sectioned guides don't want the same chunking, and "800 seemed
     reasonable" earns nothing. Point at something you noticed when you read
     the documents in Milestone 1.

     If you changed your mind partway through, say so and say why. That's worth
     more than pretending you got it right first time.

     Milestone 3. -->

## Sample Chunks

<!-- Five chunks, pasted as text. Label each one and name the file it came from
     AND the function that produced it — the grader checks your code against
     what you claim here.

     `python app.py chunks -n 5` prints all three for you. Copy them straight
     across.

     Milestone 3. -->

**Chunk 1** — source: `admin_add_drop_deadline.txt#0` — produced by: `chunker.py::split_documents`

```
On the add/drop deadline

You can add a course through the end of the second week. Dropping is a longer window — through the end of week six — but a drop after week two shows as a W on your transcript. Nothing anywhere on the registrar's site says this plainly, and students find out from each other.

```

**Chunk 2** — source: `course_biol_160.txt#0` — produced by: `chunker.py::split_documents`

```
BIOL 160 Cell Biology

I lived here my sophomore year. Format is lecture three times a week with a weekly lab. Assessment: four unit tests and a cumulative final. Not curved.

Expect 9 to 11 hours a week, the heaviest first-year course by reputation.

The one piece of advice: the unit tests come fast, roughly every three weeks; falling behind once is very hard to recover from.
```

**Chunk 3** — source: `course_hist_118_workload.txt#0` — produced by: `chunker.py::split_documents`

```
Workload for HIST 118 Modern World History

People keep asking so: a lot of reading, about 120 pages a week, but no problem sets. That's real time, not optimistic time.

It's front-loaded — the first month is heavier than the rest, partly because you're learning the format.
```

**Chunk 4** — source: `dining_pellew_dining_hall_followup.txt#0` — produced by: `chunker.py::split_documents`

```
Re: Pellew Dining Hall

Adding to what people have said about Pellew Dining Hall. The wait figure of 12 to 18 minutes at peak matches what I've seen. If you're trying to eat between classes, go before 11:45 and it's a different building entirely.

Also worth saying: the furthest hall from anywhere, next to the athletics centre. Nobody tells you this at orientation.
```

**Chunk 5** — source: `housing_innisfree_hall.txt#0` — produced by: `chunker.py::split_documents`

```
Innisfree Hall — what it's actually like

Transferred in last year, so take this with a grain of salt. Built 1991, renovated 2022. Rooms are doubles arranged as pairs sharing one bathroom between two rooms.

The good: the shared-bathroom-between-two-rooms arrangement is the best compromise on campus.

The bad: no air conditioning, which matters for the first three weeks of September.

Laundry costs $1.75 wash, $1.75 dry, app-based. On noise: moderate; the building is L-shaped and the short wing is much quieter.

```

## Sample Answer

<!-- One complete question and answer, pasted as text, with the source line
     visible. Milestone 4. -->

**Question:**

How accurate is the campus shuttle timetable?

**Answer:**

```
python app.py ask "How accurate is the campus shuttle timetable?"
  (best distance 0.412, cutoff 0.65)

According to `transit_shuttle.txt`, the published timetable is optimistic by about five minutes in the morning and accurate the rest of the day.

Sources retrieved: course_stat_150.txt, transit_shuttle.txt, transit_walking.txt

1 model calls this session, 455 tokens (425 in, 30 out)
```

**My relevance cutoff:**

<!-- The number you set in config.py, and how you got there.

     You ran five questions your corpus covers and the five in OUT_OF_SCOPE
     that it clearly doesn't, and wrote down the best distance for each. What
     did those two groups look like? Where was the gap? Put the actual numbers
     here — the table below wants all ten rows.

     Milestone 4. -->

I set on 0.65 for relevance cutoff.

My five questions landed between 0.2170 and 0.4844. The five out-of-scope ones landed between 0.8246 and 0.9340. Nothing landed in between, so the gap is 0.34 wide and I put the cutoff in the middle of it: (0.4844 + 0.8246) / 2 = 0.6545, rounded to 0.65. That leaves about 0.17 of room on each side, so a real question would have to come back noticeably worse than my worst one before the gate wrongly refused it, and an out-of-scope question would have to come back noticeably better before it slipped through.

The gap is wide enough that anything from roughly 0.52 to 0.78 would score the same 5 of 5 both ways on these ten questions. I went with the midpoint because I have no reason to lean toward refusing over answering or the other way round, and the ten numbers I have are a sample, not every question the system will ever get.

I set top_k = 3 because after the third chunk there was a clear difference in distance. For example, the first question in chart has third chunk distance as 0.5426 and the fourth chunk as 0.6510. It was a pattern I saw on the other 4 questions so I found it best to keep the top_k at a lower number than 4 or 5. 

| Question | In corpus? | Best distance |
|---|---|---|
| Is there a penalty for declaring my major late? | Yes | 0.2170 |
| Which group study rooms have whiteboards that actually erase? | Yes | 0.3436 |
| How accurate is the campus shuttle timetable? | Yes | 0.4116 |
| Can my financial aid help cover the costs of studying abroad? | Yes | 0.4698 |
| What do I neeed to do to get the campus store to match a lower price? | Yes | 0.4844 |
| What is the capital of Mongolia? | No | 0.8246 |
| What is the recommended dosage of ibuprofen for a headache? | No | 0.8442 |
| Who won the 1994 World Cup? | No | 0.8859 |
| How do I write a for loop in Rust? | No | 0.8960 |
| How do I change the oil in a diesel engine? | No | 0.9340 |

## How I Used AI

<!-- Two specific moments. For each: what you asked for, what came back, and
     what you changed about it.

     "I asked Claude to write the chunking function from my notes. It ignored
     the overlap, so I added that myself" is the level of detail we're after.
     "I used AI to help me code" is not.

     Milestone 5. -->

**1.**

I found AI helpful in working out what my criteria should be. For my last criterion I had thought that just having the question's expects string in the final was good. But I had to go back and forth about three times before I finalized it, and I asked questions back on its reasoning each time rather than just accepting its suggestions. Two things came out of that. First was that it pointed out that my criterion overlapped with the first one unless I was explicit that this one checks the fact surviving generation, not just retrieval. The other was that it got me thinking about which expects strings could actually be matched literally. Shorter ones are more likely than the longer phrases. That is why it help me add that missing factor of set it at 4 of 5 instead of 5 that I expect one of the questions to miss on wording. 

**2.**

Another time AI helped was in Milestone 3, where I started from my Milestone 1 notes on the corpus. I had written down that the documents were short and that splitting on character count would be best. But as I was changing the structure of `split_documents` function, I figured paragraph splitting would be more reliable at not cutting sentences midway. When I tested it, the paragraph chunks came out too short, where a single post got broken into pieces that no longer had the full context needed to answer the question accurately. As I went along I had AI reason through what I was thinking and suggest what could go wrong, which pushed me to check the actual numbers of 88 documents with median of 309 characters and longest 549. Nothing was close to needing a split. So I changed my strategy to one document, one chunk, and `split_documents` now returns each document as a single chunk.  


<!-- ── Stretch features ─────────────────────────────────────────────────────
     Doing one? Say so here BEFORE you start. A feature this README never
     claims earns nothing.
     ───────────────────────────────────────────────────────────────────────── -->

---

# Unit 2

<!-- These sections get ADDED to what's already above. Don't delete or rewrite
     unit 1 — the point is that someone can see what you said before you knew
     how it went. -->

## Run Log — Before

<!-- Your five criteria, three runs each. `python run_eval.py --label before`
     runs the questions, puts the OUT_OF_SCOPE ones through the gate, and
     writes it all into results/ for you. Targets come from criteria.md; the
     verdict column is your call.

     Criterion 3 is measured in one deterministic pass rather than three, so
     the same number goes in all three run columns. That's correct, not lazy.

     Milestone 1. -->

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 |  |  |  |  |
| 2. Every answer names a source | 5 of 5 |  |  |  |  |
| 3. Gate stops out-of-corpus questions | 4 of 5 |  |  |  |  |
| 4. | | | | | |
| 5. | | | | | |

<!-- Underneath, paste the REAL output for each criterion from one of your
     runs — the actual text your system produced, not a description of it.
     Name the file and function that produced it. -->

## Verdicts

<!-- MET or MISSED for each of the five, against the target you wrote last
     unit — not a new one. Plus a sentence on how you decided. That sentence
     matters most where it was close.

     If your target said 4 of 5 and your runs came out 4, 3, 4, that's a MISS.
     The target has to hold, not show up occasionally.

     Milestone 2. -->

| # | Criterion | Verdict | How I decided |
|---|---|---|---|
| 1 |  |  |  |
| 2 |  |  |  |
| 3 |  |  |  |
| 4 |  |  |  |
| 5 |  |  |  |

## Diagnoses

<!-- For each miss: which stage caused it, and how. The stage alone isn't
     enough — you need the mechanism.

     Not a diagnosis: "Question 3 didn't work."
     A diagnosis:     "Question 3 asks about laundry costs. The answer is in
                       one sentence that got split across two chunks, so
                       neither chunk on its own contains it."

     The five stages: loading → chunking → embedding → retrieval → generation.

     Look for a pattern. If three misses all ask about numbers, that's one
     problem, not three.

     Missed nothing? Say so, then say honestly whether your targets were set
     low, and which one you'd tighten and to what.

     Milestone 3. -->

## The Improvement

**What I changed:**

**Why I picked it:**

<!-- Connect it to a specific diagnosis above in one sentence. If you can't,
     you picked a fix because it sounded impressive. -->

### Run Log — After

<!-- Same format, same five criteria, three runs each.
     `python run_eval.py --label after` -->

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 |  |  |  |  |
| 2. Every answer names a source | 5 of 5 |  |  |  |  |
| 3. Gate stops out-of-corpus questions | 4 of 5 |  |  |  |  |
| 4. | | | | | |
| 5. | | | | | |

**Did it help?**

<!-- Say plainly whether it did, and how you know. If it made things worse,
     say that — a change that backfired, honestly reported, earns full credit
     and is more interesting than one that worked. What matters is that you can
     tell.

     Milestone 4. -->

## What's Still Broken

<!-- For each criterion still missed after your fix: what you'd do about it,
     and why you stopped where you did.

     "I ran out of time" is fine if it's true. Pretending nothing is left is
     not.

     Milestone 5. -->

## What I'd Do Differently

<!-- Knowing what you know now — which of your five criteria would you write
     differently, and why?

     Milestone 5. -->
