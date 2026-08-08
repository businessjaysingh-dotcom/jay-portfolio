# An outreach engine that holds its own conversations

I built a system that finds people, qualifies them, and runs the entire conversation itself — then I documented every decision it makes, including the ones it got wrong.

**▶ [See it running](https://businessjaysingh-dotcom.github.io/jay-portfolio/)** — a short self-playing walkthrough. Press `P`.

**▶ [Three real conversations, decoded](https://businessjaysingh-dotcom.github.io/jay-portfolio/conversations.html)** — replayed message by message, with the rule that fired on each reply shown beside it.

---

## What it actually does

| | |
|---|---|
| **11,115** | prospects harvested from group chats |
| **194** | passed qualification — 1.7%, the filter is meant to be brutal |
| **257** | people cold-messaged |
| **38.5%** | replied to a first message from a stranger |
| **402** | replies written by the system across 131 live conversations |
| **$0.036** | cost per reply |
| **4** | meetings offered — **the part that doesn't work yet** |

Every figure is read out of the system's own logs. Nothing is rounded up.

That last row is the honest weak point. Getting a stranger to reply works. Converting those conversations into booked meetings does not work yet, and I know why: the booking gate requires a qualification score of 60 while the average logged score is 19. The system is correctly refusing to pitch people it hasn't earned. Loosening that gate without fixing lead quality would just produce worse conversations faster.

I'd rather show you that than hide it.

---

## The idea

Good conversations rarely die because someone says no. They die because a thread that was going fine quietly stops — somebody pitched on message two, or asked a question the other person already answered, or kept qualifying a buyer who'd already said they were in.

None of those are talent problems. They're **rules nobody wrote down.**

So I wrote them down as conditions, and built the thing that follows them every time:

```
rapport  ──▶  qualifying  ──▶  educating  ──▶  booking / walk away
   │              │                │                 │
 8 exchanges   3 answers +     give the fix      one offer only
 no questions  score ≥ 50      away for real     never a price
 allowed       to advance      no strings        never a second ask
```

A strong buying signal overrides all of it. If someone says *"can we get started"* and they're qualified, every remaining question is skipped and the link goes out in the next message — because every extra question after a buying signal is a chance to lose the deal.

---

## Architecture

34 Python modules, 10,883 lines, six agents running concurrently on one event loop.

```
entry/          start.py · supervisor.py            boot + crash recovery
agents/         reply · outreach · tracker          six concurrent workers
                reactivation · phone control
intelligence/   brain.py · shared.py                what to say next
                prompts · qualifier · drafter
pipeline/       prospector · scraper · finder       where leads come from
surfaces/       crm.py · jarvis.py · control.py     dashboards + reporting
guards/         quality gate · blacklist            the brakes
archive/        autopilot.py                        the 1,241-line original
```

`archive/autopilot.py` is kept deliberately. Everything above used to live in that one file. It's the before photo.

**Design decisions worth naming:**

- **Reporting came before optimisation.** The reason I can tell you the funnel leaks at the meeting-offer step rather than guess is that `jarvis.py` existed before I tried to improve any number.
- **Brakes before engine.** Refusal detection, a never-contact list, one-touch caps on every re-engagement path, and a hard stop on quoting price. An outreach system without brakes doesn't scale — it just gets banned faster.
- **Atomic writes everywhere.** A crash mid-write once corrupted the conversation log. Every write now goes to a temp file and gets swapped in atomically.
- **One source of truth for runtime settings.** Settings scattered across files is how you end up with two contradictory pause switches.

---

## Six bugs that reached a real person

The failures taught me more than the features. Each one is now a rule I build in from the start.

| What happened | What it taught me |
|---|---|
| Sent the same message three times — he replied *"try be more human"* | My loop guard checked his repeats, never my own |
| Leaked an assistant-style refusal into a live message | Nothing generative reaches a person without an output gate |
| Got caught mid-conversation; my logs contain *"Bro sorry its my AI"* | Rewrote it to answer honestly when asked. Hiding it was never worth it |
| Interrogated a buyer who'd asked to start **twice** | Buying signals must override your own script |
| A crash corrupted its own memory | Durability isn't optional in an unattended system |
| Marked small operators as dead leads | They were the entry-tier customers — the filter was discarding the buyers |

---

## What it looks like when it works

> *"i did my research, but most testimonies and dms i trade with people are scammy. you're the only one basically not trying to sell me some bs"*

Day 34 of a thread. Nine messages, zero pitches — because he'd said he was months away from restarting, and pitching a months-out timeline is how you lose someone. It earned that sentence by not selling.

He was talking to software.

---

## Privacy

Every conversation shown in these pages has had the other party's name, handle and avatar removed **at the source** — they're block characters, not a blur filter. There is no real identity anywhere in this repository, and personal details like location or finances have been stripped from the message text.

The engine's source is not published here. The pages describe the architecture and show selected excerpts; they don't hand over the methodology.

---

## Running the pages

No build step, no dependencies, no internet required.

```bash
git clone https://github.com/businessjaysingh-dotcom/jay-portfolio.git
cd jay-portfolio
open index.html
```

---

## Contact

**Jay** · Canada · building sales systems

- [businessjaysingh@gmail.com](mailto:businessjaysingh@gmail.com)
- Instagram — [@jayycloses](https://instagram.com/jayycloses)
