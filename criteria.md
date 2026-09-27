# Acceptance criteria — The Unofficial Guide

Five criteria that say what "working" means for this system, written in unit 1
**before** any results existed.

An acceptance criterion names a target: a number, a count, a rate, or something
a person could plainly observe. *"Retrieval works"* is an opinion. *"For at
least 4 of my 5 test questions, the top results include a chunk containing the
answer"* is a criterion.

Under each one, write a sentence or two on **why that target** and not a
stricter or looser one. A reason that says something about your corpus or your
pipeline earns credit; *"80% seemed reasonable"* does not.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

---

## 1. Retrieved chunks contain the answer

For at least 4 of my 5 test questions, the retrieved chunks include one that
contains the answer.

**Why this target:**
<!-- e.g. "One of my questions is about a topic only two documents mention, so
     I expect that one to be hard." -->
My price matching question does not contain the word textbook, but the only document that answers it is about textbooks and spends half of the document talking about library reserve copies. The other four answers are stated in documents whose titles match the question directly, so I expect those to pass and this one to be hard. 

---

## 2. Every answer names a source

Every answer the system produces names at least one source document.

**Why this target:**
<!-- Why all five and not four? What about your setup makes that achievable —
     or what would have to go wrong for it not to be? -->
In `generate.py` the grounding instructions tells the model to name the documents the answer came from. If it misses any it would be because the model ignored the line rather than the filename not being available. 

---

## 3. The relevance gate stops out-of-corpus questions

When I ask a question my documents clearly don't cover, the relevance gate
stops it and the system returns "I don't have enough information about that" —
in at least 4 of 5 tries.

<!-- The five questions are the ones in `OUT_OF_SCOPE` at the bottom of
     `questions.py`, and `run_eval.py` puts them through the gate and writes
     what happened into your run log. Swap them for your own if you'd rather —
     just keep five of them, or the "4 of 5" above has nothing to be 4 of. -->

**Why this target:**
<!-- What did your distances look like when you set the cutoff in Milestone 4?
     Was there a clean gap, or did the two groups overlap? -->
Four of the five questions have no relevance to the campus corpus so I expect the gate to be able to catch them. The fifth would be the one that asks about ibuprofen dosage which has connection to the health_center.txt document but it does not contain any information on medications. That is the one I expect to land closest to the cutoff. 

---

## 4. Something about your chunks

<!-- YOU WRITE THIS ONE.

     How would you know if your chunks were the right size? Name something
     countable or observable.

     Examples of the right shape — don't copy these, they should come from
     what you actually saw in Milestone 3:
       - "At least 4 of 5 sampled chunks read as a complete thought, with no
          sentence cut in half at either end."
       - "No chunk is shorter than 200 characters, since anything below that
          in my corpus turned out to be a heading with no content under it." -->

Every chunk begins with its document's title line.

**Why this target:**

Every document in the corpus has the same structure of a title line, a blank line, and then two or three short paragraphs. I say every chunk because it either holds for all of them or none. It matters because the answers to every one of my five test questions is in the body, not in the title line, and the body paragraph holding it often does not repeat the topic word. Like in the `transit_shuttle.txt` where the answer says "the published timetables is optimistic by about five minutes" and does not mention "shuttle". Also the `admin_study_abroad.txt` says "the financial aid package travels with you" and not "abroad". Both words exist only in the title line. 

**Revised in unit 2**: For at least 4 of 5 test questions, the first ranked retrieved chunk is the one containing the answer.

**Why revised**: The original could not fail. Two separate facts combine to make it true before any run existed. Every document in this corpus was written with a title line, a blank line, then the body, and `split_documents` returns each document as one chunk with no text modifications. Neither one alone would guarantee it, but together they leave no input on whch the criterion could come out false. I wrote it in Part 1 of the project while I was still planning to chunk on paragraphs, where it would have been a real test. Under paragraph chunking, every chunk after the first would have lost the title line. When my sample chunking showed that paragraph pieces were to short to carry the context needed to answer a question, I switched to one document per chunk and did not go back to check what that did to this criterion. So it passed three runs for the five questions without measuring anything. 


The concern behind it was real where the title line carries the topic word the body omits. But one chunk per document now guarantees that, so what is left worth testing is the risk my design carries. A whole document chunk has one embedding standing in for every paragraph in the document, including the ones with nothing to do with the question. That cuts both ways where the right document's vector gets pulled away from the question by its off topic paragraph and a wrong document's vector can get pulled toward the question by some incidental paragraph that happens to share vocabulary. Either one can swap rank 1 and rank 2. Criterion 1 can not verify this because it only asks for the answer to be in the top 3 but ranking can make it countable. 

I set it to be 4 of 5 rather than 5 because of which question is exposed. The sentence that answers the price-matching question is a close match to the `money_textbooks.txt` document where it says "The campus store price-matches". The question asks how to get the campus store to match a lower price but that sentence is one of three paragraphs in the chunk, where the other two are about two-hour library reserve copies. Whole document chunking averages a strong match on a third of the chunk together with two thirds about something else. The question does not mention about this being about textbooks which is a word found in the title line. Meanwhile 17 of 88 documents metion price or costs like dining and housing. Any one of those could sit above a diluted vector. That is the question I expect to loose the top slot, and I am not predicting which document beats it. 

---

## 5. Your choice

<!-- YOU WRITE THIS ONE TOO.

     Pick something you actually care about getting right. It could be about
     speed, about refusals, about a particular kind of question your corpus
     handles badly, about source attribution being correct rather than merely
     present — anything, as long as it names a number or an observable
     outcome. -->

For at least 4 of 5 questions, the system's final answer contains the question's expects string, not just the retrieved chunks.

**Why this target:**

The first criterion checks that the right chunk came back. In this criterion it checks that the fact survives generation. The model can retrieve the textbook document and still write that "the store offers price matching" without saying ask at the counter. I set it at 4 of 5 rather than 5 because short expects like "210", "no penalty", and "five minutes" are most likely to be reproduced exactly, whereas "travels with you" and "ask at the counter" can be paraphrased while still answering the question correctly. So I expect one of those to miss on the wording rather than substance. 

---

<!-- ─────────────────────────────────────────────────────────────────────────
     UNIT 2 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath it, like this:

         ## 1. Retrieved chunks contain the answer

         For at least 4 of my 5 test questions, the retrieved chunks include
         one that contains the answer.

         **Why this target:** ...

         > **Revised in unit 2:** For at least 4 of 5 questions, the top three
         > results contain the answer.
         >
         > **Why revised:** I couldn't judge "the chunks include one that
         > contains the answer" the same way twice — I scored two questions
         > differently on Monday than on Wednesday. The new version is
         > something I can actually check.

     That's a revision because the criterion couldn't be MEASURED.

     Lowering a target because you missed it is not a revision, and it costs
     you the point:

         ✗ "I said 4 of 5 but got 2 of 5, so 2 of 5 is more realistic."

     A number you missed stays where it is, gets diagnosed, and gets a fix
     attempted. That's where the points are.

     The whole reason the originals stay visible is so someone can see what you
     said before you knew the answer.
     ───────────────────────────────────────────────────────────────────────── -->
