# AMPS Learning Kit

A self-contained, interactive kit for learning **AMPS (Advanced Message Processing System)** — the 60East Technologies pub/sub messaging engine — from a **JavaScript frontend developer's** perspective. No prior AMPS or trading knowledge required.

## Contents

| File | What it is |
|------|------------|
| **AMPS_Playground.html** | The main interactive playground. Open in any browser (no setup, works offline). Click-through lessons for every concept, an interactive diagram, clickable payloads, a regex-subscription explorer, a message-command inspector, a quiz, and a printable cheat sheet. |
| **AMPS_CheatSheet.html** | Standalone, print-first one-page reference (commands, filter operators, gotchas). Open and use "Print / Save as PDF". |
| **AMPS_Learning_Guide.md** | A written guide: 4-week learning plan, concepts, runnable JS snippets, production concerns. |
| **AMPS_Interactive_Tutorial.html** | An earlier interactive intro with a live order-blotter demo. |

## How to use

Just open **AMPS_Playground.html** in a browser. Everything is self-contained — no build step, no dependencies, no internet needed.

Sidebar covers:
- **Core concepts:** Publish/Subscribe, Topic, Message, Publish, Subscribe, Content Filtering, State of the World, Command
- **Power features:** Out-of-Focus (OOF), Delta Messaging, Transaction Log & Replay, Queues
- **Going to production:** Connecting & Logon, Message Types, Error Handling, High Availability
- **Review & test:** Cheat Sheet, Quiz

## Status

Work in progress — core concepts 1–4 (Publish/Subscribe, Topic, Message, Publish) have been deeply expanded with interactive demos and plain-English explanations. Concepts 5+ are functional and being refined.

## Reference

Official docs: https://crankuptheamps.com/clients/amps-client-javascript
