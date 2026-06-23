---
name: knowledge-answering
description: "Risponde a domande fattuali interrogando le Knowledge bases attached, cita le fonti, dichiara apertamente quando un'informazione non è nelle KB."
---
# Knowledge answering

The user has attached some Knowledge bases (notebooks) to you. They're listed under `## Knowledge bases` in your AGENTS.md, each with a `notebook_id` you'll use to query.

Follow this flow for ANY factual question — about contracts, policies, processes, products, history, anything that might be in a document the user trusts.

## Stage 1 — recognise + decide

Questions that ALWAYS trigger this skill:
- "cosa dice il contratto X / la clausola Y / la policy Z"
- "quali sono i nostri ..." / "qual è la nostra ..."
- "come si fa ..." (when the user implies there's a documented process)
- numeric / date / name questions where being wrong is worse than admitting you don't know

Questions where you can skip KB lookup:
- generic world-knowledge that doesn't depend on the company ("qual è la capitale della Francia?")
- creative / opinion / brainstorm requests
- direct math / unit conversion

## Stage 2 — pick the right notebook(s)

If only one notebook is attached → query it.

If multiple notebooks attached:
- pick the one whose **name** best matches the question's topic (e.g. "Contratti vendite 2026" for a contract question)
- if the topic is ambiguous OR spans multiple notebooks, query each one in parallel-thinking, then synthesise
- if no notebook name clearly relates, do query all of them anyway — empty/no-match results are cheap

If NO notebook is attached: skip to Stage 5 (fallback).

## Stage 3 — query

```
call_recipe("knowledge.query", {
  notebook_id: "<id from AGENTS.md>",
  question: "<rephrase the user's question for the retrieval engine — clear, specific, single intent>"
})
```

Don't ask the user to repeat themselves. If their question has 2 sub-parts, issue 2 separate queries; don't try to cram both into one.

## Stage 4 — synthesise + cite

When the recipe returns useful content:
- write a direct answer in the user's language
- always cite the source: `"Secondo [Nome della KB / titolo del documento]: ..."`
- quote sparingly — paraphrase, but be faithful to what the source says
- if the source is unclear or contradictory, surface that ("Il documento dice X ma in un altro punto dice Y, ti conviene verificare con [persona di riferimento]")

If the recipe returns nothing useful (empty, off-topic, just a snippet):
- proceed to Stage 5 (fallback) — don't pretend you got an answer

## Stage 5 — fallback (explicit)

If no KB had the answer, you MAY answer from your own training-data knowledge, but you MUST flag it:

> "Questo non l'ho nelle KB che mi hai dato. Per quel che so io più in generale: [risposta]. Verifica con [persona di riferimento] se ti serve la versione ufficiale."

Never silently improvise something that sounds authoritative. The cost of a wrong-but-confident answer on a contract clause is much higher than the cost of saying "non lo so per certo".

## Stage 6 — ask before assume

If the question is ambiguous about which document / which subject:
- ask ONE focused clarification question before querying
- example: user asks "qual è la durata standard?" → "Della garanzia, del contratto di vendita, o del rapporto di consulenza?"

Don't snowball clarifications: max 1 round, then commit and answer.

## Don't

- Don't list out the KB infrastructure to the user ("sto consultando la knowledge base X via knowledge.query..."). Just answer.
- Don't quote `notebook_id` UUIDs in the chat — those are internal handles, the user doesn't care.
- Don't tell the user "non posso rispondere" without first trying the KBs. Try, fail, fallback explicitly.
