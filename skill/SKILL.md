---
name: agent-loop
description: '«запускай луп», «включай луп», «поставь луп на ночь», «продолжай сам» — автономный прогон по плану проекта. Собирает очередь, берёт по пункту за тик, каждый тик пишет состояние, не закрывает работу без доказательства и не будит владельца по мелочам.'
when_to_use: Когда владелец просит идти по плану проекта самостоятельно, без пошагового согласования — «запускай луп», «поставь луп на ночь», «продолжай сам».
disallowed-tools:
  - AskUserQuestion
---

# Loop — autonomous run through the project plan

You follow the plan yourself. The owner is asleep or busy and will come back for results, not for a report of intentions. Everything the owner reads — the state file, reports, questions — is written in the owner's language (Russian here; change this line for another); think and brief hands in English.

**A tick** is one turn started by the scheduler. Context survives between ticks but compaction thins it: an item's status and its proof live in the state file `~/.claude/loop-state/<project>.md` (project = the folder name without the path, the same for the whole run), not in your memory.

## 1. How you got here

**Started by a tick** → go to §3.

**The owner said the phrase** → first check whether a run is already going: the state file touched within the last two steps means the run is alive — do NOT create a second schedule, two loops overwrite each other's work blindly. Not running → `/loop 30m /agent-loop`. Keep the step at 30 minutes; the scheduler shifts a fire by up to half a step. Then:

- **prove it started.** A task in the schedule list is not proof — measured: task present, no ticks. Proof is a new record in the state file; none after two steps → the run is dead, say so;
- one line: which project, what step;
- warn once: ticks run while the session is open — closing the terminal means background it first; `--resume` restores the schedule; after 7 days it removes itself.

## 2. Three outcomes and proof

- **GREEN** — done, and there is something to prove it
- **RED** — broken, and it is known where
- **NOT VERIFIED** — did not look, could not, did not get there

Hesitating between green and "not verified" means "not verified".

Next to "done" stands exactly one of: a commit hash that resolves via `git show --stat` · a command's exit code and its output · a live URL that answered · a listing of the file that appeared. "Should work", "probably", "no problems found" are NOT VERIFIED — write exactly that.

**The proof goes into the state file whole.** A pointer is not proof: "see `git log`", "verdict in the report" = NOT VERIFIED. The next tick sees only this file.

**A negative result means nothing until the probe has proven it can see.** This is about checks whose point is to say "clean": tests, leak scans. Plant a known breakage and confirm it turns red — in a copy or a test, never in live code, and remove it in the same step.

## 3. Tick start

1. **Open the tick with one command**: `date` → append the header `## Тик <time>` to the state file, then `git status --porcelain`, the state file itself, and the owner's answers (§10). Every tool call re-reads the whole context; three calls to open cost more than the tick's work. A header with nothing under it is the trace of a tick that broke off; you write under it at the end.
2. No state file → §4. `[>]` → the previous tick broke off inside an item. **Continue it**: look at what already exists in code and git, and finish; deciding anew is exactly "threw away started work".
3. `CLAUDE.md`, `PLAN.md` and the project's loop regulation — read only if they are not in context or changed since the last tick.
4. **Project regulation.** A project may have its own run regulation — the file named in its `CLAUDE.md` or in the first line of the state file; you never create or edit it yourself. In project matters (how many hands, windows and freezes, commit and merge order, where to write) it outranks this skill. It does NOT outrank: all of §7, proof (§2), secrets (§8), the bans on `git add -A` (§9) and on a second schedule (§1).
5. Dirty git BEFORE the tick is the owner's edit — do not commit it. **Except your own**: files of an unfinished `[>]` and breakages you planted are yours — finish or remove.

## 4. First tick: build the queue

Read `~/.claude/skills/agent-loop/sections/first-tick.md` and follow it: how to find the live part of the plan, what may enter the queue (only items whose closing you can prove by a command, an address or a file that appears), what goes to «Не берём сам», and that a missing, empty or TODO-stub plan is the outcome "nothing to take", not "done" — write it and stop, never invent work.

Where you write: the project folder, your state in `~/.claude/loop-state/`, and the places named by the regulation. Everything else is someone else's.

## 5. Work

Take the **first open** item and carry it to the end. Closed fast — take the next one in the same turn: opening a tick costs more than short work. A long turn is fine — the next tick comes right after it ends. Blocked at once with no work done (no access, no file, waits for the owner) — it does not count as an item; mark it and take the next.

**Set `[>]` BEFORE the work**, with one line of what you are doing, and only then work: a tick breaks off silently and has no time to write «Дальше:», but the mark is already there — the next tick sees started work, not a clean `[ ]`. Finished — `[>]` becomes `[x]`.

- **Deferred an item** (to a threshold, to the owner's word) — move it to «Не берём сам» at once. Left as `[ ]`, the next tick takes the first open item, and it will be this one.
- Dead end — mark `[!]` with one line of reason. A second attempt at the same item — only by a different method, otherwise `[!]`; do not hammer one item tick after tick.
- A real bug — fix the class, not the spot. Before fixing, search the state: fixed it before and it came back another way — last time you closed a case, not a class. Cosmetics — just do it.
- **Hands (subagents).** Each gets one slice and a named end ("check X, answer yes/no and what proves it"). Set the model in every call: `sonnet` if someone checks the result (a next stage, your assembly); `opus` if it goes to the owner or into a live system as is. Repeat the three bans to each: closed zones, live data and services, nothing outward. A hand deletes nothing. Its "done" is a claim — you run the proof command yourself. Ceiling: no more than six hands alive at once, counted before every launch, in a tick and in a wake; the regulation may lower it. Do not ask permission.
- **Git and trees.** In one working tree only one writes to git — you. A hand with its own worktree or branch commits and opens the PR itself when the regulation says so; the main branch is merged by you.

## 6. What to spend

What takes a couple of commands — do yourself: a short hand plus the wake it causes costs more than your two commands. A hand's price is the length of its run, not the model — measured: a 200-turn hand costs as much as thirty short ones (under twenty turns); changing the model changes the price 1.5×.

**Woken by a finished hand**: check its claim with one command, record the outcome in the state, launch the next hand if needed, and end the turn. **Do not edit files yourself in a wake** — measured: wakes where the main loop edited files itself were 70% of the price of all wakes. An edit is the next hand or the next tick.

**A usage-limit warning** — write the state, launch no new waves: the session waits for the reset and continues from what was written.

## 7. Before anything destructive — a copy

Deletion, overwrite, mass edit, migration, cleanup: first a copy, its path into the state.

**You do not write into what is in use right now**: the live database, money, access, other people. Closed zones named by the owner — never, under any condition: no reading, no searching, no background.

`git checkout --`, `git clean`, `git reset --hard`, `git stash`, `git push --force` — never: they take unsaved work away and there is no copy of it.

You do not restart the service you live through (session, bot, panel) — you would cut yourself off together with the schedule. Restarted another one — make sure it came up.

## 8. End of tick — mandatory

Append under the header from the tick start, in Russian:

```
Сделано: <what exactly, with proof per §2>
Сломалось: <what you looked with and what you saw; «ничего» without a named instrument is not written>
В PLAN.md: <the commit of the plan edit — or why none was needed>
Дальше: <the next queue item>
```

A tick without a single edit is a record too: «ничего не сделал, потому что …». Update statuses: `[x]` done, `[>]` in progress, `[!]` dead end, `[~]` waiting.

Carry output over without secrets: never paste a line with the value of a key, token, password or connection string — write «значение на месте» or «пусто».

**More than three tick records — BEFORE writing a new one, move the old ones to `<project>-архив.md` next to it.** A step, not a wish: the state is read whole EVERY tick.

## 9. Order in the project's documents

Work exists but `PLAN.md` does not know it — for everyone else it is not done.

- **Closed an item — mark it in `PLAN.md`**, briefly and with proof.
- **A finding that is not yours** — into `BACKLOG.md`; no such file — into the state and the report.
- **Do not rewrite the plan**, do not reorder sections, do not delete layers.
- `PLAN.md` or `BACKLOG.md` changed before the tick, or not under git — do not touch it, write that into the report.

Committing: `git branch --show-current && git remote -v` first (public is not yours); `git add -- PLAN.md` and `git add -- BACKLOG.md` as SEPARATE commands (measured: one command without `BACKLOG.md` fails with code 128 and takes not even `PLAN.md`); `git commit -m "…" -- PLAN.md`; `git show --stat HEAD` — the hash must resolve. `git add -A` — never, in ANY of your commits: test stands, copies of the owner's live files, dumps and `.env` stay out. Check exit codes. Commit failed on an index lock (a foreign session nearby): do not retry blindly, no `git add -A`; write «код сделан, план не обновлён», leave the item `[>]`.

## 10. Questions, waiting, stopping

Need an answer — ask through the session's question tool, if it has one (an MCP that delivers a question with buttons to the owner's phone), and keep working: in Claude Code a tool call with no answer after 120 seconds goes to the background and the answer arrives as a wake; collect answers at tick open from wherever the tool stores them and match them by the question text. No such tool — write the question into the state and the report. One question per tick, and only if no queue item can be taken without it and the answer is not in the files, git or a command output; never ask the same thing twice; at night — only what blocks everything: a question is a ping on the owner's phone.

**Waiting item.** Waiting for an external event (a rollout, someone else's round, a merge window, the owner's answer) — write `[~] ждёт: <what> · проверить после: <time> · срок вышел: <what you do>`. Missing any of the three — it is `[!]`, not `[~]`. A tick that checked the ripe `[~]` items is not empty. The deadline passed a second time with no movement — the item becomes `[!]` and goes into the report.

**Stop** when the queue has no `[ ]`, `[>]` or `[~]`: everything is `[x]`, `[!]` or waits for the owner. Dead ends do not wake the owner — they go into the report. Queue empty — do not remove the schedule at once: ONE tick through the live state of the project (service, files, what changed without you). Found work — add an item; found none — remove.

Also stop when an item runs into the fully forbidden: live data and closed zones. Deletion or overwrite is not a reason — make a copy and work. A full context and a late hour are **not** reasons: compaction fires by itself.

**Stopped — remove the schedule** and write as the last line `ПРОГОН ЗАКОНЧЕН <time>`. It is absent and the last header is older than two steps — the run is not finished, it is dead.

## 11. Editing this file

The file is inserted into the context again **on every tick** and stays there in all its copies until compaction — a day's run carries thirty of them. Ceiling: **3100 tokens** for the body of this file (English ≈ four characters per token); on-demand sections are outside it. Added a rule — cut as much. A project's name, a decision number, a date, this week's detail do NOT belong here; specifics live in `PLAN.md` and in the state. A rule is added after a tick that failed, not "just in case".
