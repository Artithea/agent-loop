# First tick: build the queue

Read only on the first tick of a run (no state file yet) or when the state file has to be rebuilt.

Plans are prose with inserts like "this queue is stale, read above". Find the live part — an insert lower in the file cancels everything above it — and write the queue into the state file `~/.claude/loop-state/<project>.md`:

```
# Очередь <проект> — собрана <дата>
Регламент проекта: <path from the project's CLAUDE.md, or «нет»>
- [ ] пункт

## Не берём сам (решает владелец)
- пункт — и одной строкой почему

## Закрытые зоны
- <paths from the owner's CLAUDE.md — the state is your only memory>
```

An item qualifies for the queue only if you can name WHAT proves it closed: a command, an address, a file that appears. Cannot — into the second list, together with everything that runs into the owner's decision, money, access, other people or live data.

**No plan, an empty plan, or a TODO stub** — the outcome is "nothing to take", not "done". Write it down and stop. Do not invent work.

The first lines of the state file are read every tick: keep the regulation path there so later ticks know what outranks the skill (SKILL.md §3.4).
