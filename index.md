---
layout: default
title: "Galaxium Lite: the IBM Bob lab"
---

# Galaxium Lite: the IBM Bob lab, step by step

Take one feature from first look to committed code with **IBM Bob**, on a small Python app. You'll give Bob context,
ask it questions, have it plan, build and test the change, check it through a live tool, review it and commit it.
Every step happens in the Bob chat, and Bob asks before it acts.

**You need:** about an hour, Bob IDE, Python 3.11 or later, and a few Bobcoins (our whole run used about 2, roughly
US$1).

**The feature:** a *refund preview*. Before cancelling a booking, a traveller can see what they'd get back:

| Time to departure | Refund |
|---|---|
| more than 7 days | 100% |
| 1 to 7 days | 50% |
| under 24 hours, or already cancelled | 0% |

> **About the videos and pictures.** Each one replays a real run of this lab: Bob Shell 2.0.6 (Bob's command-line
> version, same agent, models and modes), 20 Oct 2026. The IDE around Bob's output is recreated, and waiting time is
> cut. Bob's words, tool calls and results are real. **Your Bob will word things differently**, choose different
> names and maybe take a different route. That's normal. Check your result against the "You should see" line, not
> against the exact wording.
>
> Those runs approved every action automatically, so the replays don't show Bob IDE's approval prompts. Sections
> marked **Bob IDE only** cover what only the IDE has; their recordings are still to come.

**Contents:** [Before you start](#before-you-start) · [Set-up](#set-up-let-bob-set-itself-up) ·
[1. Context](#step-1-give-bob-context) · [2. Ask](#step-2-understand-before-changing) ·
[More `@`](#bob-ide-only-mention-exactly-what-bob-needs) · [3. Plan](#step-3-plan-the-change) ·
[4. Build](#step-4-build-it-and-run-the-tests) ·
[5. MCP](#step-5-check-it-through-an-mcp-tool) · [Review](#bob-ide-only-review-before-you-commit) ·
[6. Commit](#step-6-commit-it) · [7. Reviewer mode](#step-7-optional-a-second-opinion-in-reviewer-mode) ·
[Rollback](#bob-ide-only-undo-a-change-with-rollback) · [The `.bob/` kit](#what-the-bob-kit-did) ·
[Help](#if-something-goes-wrong)

---

## Before you start

1. **Install Bob IDE** and sign in with your IBMid. No licence yet? Start a free trial at
   [bob.ibm.com/trial](https://bob.ibm.com/trial).
2. **Check Python:** `python --version` should print 3.11 or later. On Windows, if it opens the Microsoft Store, use
   `py --version` instead.
3. **Download the lab:** [galaxium-lite.zip](https://github.com/yeokimsiang/bobathon-galaxium-lite/releases/latest/download/galaxium-lite.zip).
   Unzip it to a short path, for example `C:\lab` or `~/lab`. You get this guide and a `galaxium-lite` folder.
4. **Open the `galaxium-lite` folder itself** in Bob IDE (File > Open Folder), not the folder above it. When Bob asks,
   choose **Yes, I trust the authors**. That's what lets Bob load the project's own settings in `.bob/`.
5. **Open the Bob chat panel** and start a new task (the **+** button).

**How every step works:** pick the **mode** in the chat box (Agent, Plan or Ask), paste the **prompt**, send it,
then **approve** each action when Bob asks. Bob shows you the exact command, file edit or tool call before it runs.
Read it, then approve or reject.

> **Bob IDE only: approvals.** *Recording to come:* the trust prompt when you open the folder, and Bob's first
> approval request (a command, with Approve and Reject).
<!-- asset R1: trust prompt; R2: first command approval card -->

---

## Set-up: let Bob set itself up

| Mode | Prompt |
|---|---|
| Agent | `Create a virtual environment, install requirements.txt and run the tests.` |

**Bob does:** loads the project's `verify` skill (you'll see *Loaded skill verify*), creates a `.venv`, installs the
packages and runs the tests. Approve each command.

**You should see:** `11 passed`, in a small table: `pytest | PASS | 11 passed in …`.

![Bob's reply after set-up: Loaded skill verify, then a table with pytest PASS, 11 passed in 0.68s](https://yeokimsiang.github.io/bobathon-galaxium-lite/media/lab0.png)

**Why:** everyone starts from a known-good state, and it's your first practice at chatting and approving.

**If it's slow:** installing packages took several minutes on our machine, and Bob reported a time-out but carried on.
If the tests don't pass, see [Help](#if-something-goes-wrong).

---

## Step 1: Give Bob context

| Mode | Prompt |
|---|---|
| Agent | `/init` |

**Bob does:** reads the project (12 files in our run) and writes its briefing: an `AGENTS.md` in the project root,
plus one per mode in `.bob/rules-agent/`, `.bob/rules-ask/` and `.bob/rules-plan/`. Approve the writes.

**You should see:** a new `AGENTS.md`. Open it. A good one mentions:

- the FastAPI booking API and its MCP tools at `/mcp`
- the `app/`, `tests/` and `.bob/` layout, `uvicorn app.main:app` to run and `pytest -q` to test
- inject `now` rather than calling `datetime.now()` in services
- give every route an `operation_id`, and keep `FastApiMCP` last in `app/main.py`

**Red flags:** it describes an ORM, Docker or a frontend. This app has none.

<video src="https://yeokimsiang.github.io/bobathon-galaxium-lite/media/lab1.mp4" poster="https://yeokimsiang.github.io/bobathon-galaxium-lite/media/lab1.jpg" controls muted playsinline preload="none" width="100%"></video>

**Why:** Bob reads `AGENTS.md` at the start of every chat, so it stops guessing about the project. In our run Bob
copied the warning in `app/main.py` ("Keep this LAST") into the briefing; watch for it in step 4.

**Tip:** `AGENTS.md` is a plain file. Read it, fix anything wrong, and commit it so your team gets the same briefing.

---

## Step 2: Understand before changing

| Mode | Prompt |
|---|---|
| **Ask** | `How does cancelling a booking work today? @/app/services/bookings.py` |

Switch the mode to **Ask** first. Type `@` and pick the file from the list, so Bob gets it attached.

**Bob does:** reads the file and explains it. Ask mode can only read: it can't change anything.

**You should see:** the cancel flow: look up the booking (404 if it's missing), refuse if it's already cancelled or
the flight has left (409), then set the status to `cancelled`. And the important bit: **no refund is calculated
today**. (Ours said: "No refund / seat-release logic".)

![Bob's Ask-mode answer, ending with a Key design points table that includes No refund / seat-release logic](https://yeokimsiang.github.io/bobathon-galaxium-lite/media/lab2.png)

**Why:** you understand the code before you change it, and you know exactly what the feature will add.

---

## Bob IDE only: mention exactly what Bob needs

In step 2 you gave Bob a file with `@`. That's the quickest way to give Bob context: it gets exactly what you point
at, instead of searching the project for it, which is slower and costs more. You can point at more than files. Stay
in **Ask** mode; these prompts only read.

**1. Open the list.** Type `@` in the chat box. Bob IDE suggests open and recent files, folders, recent commits, and
three keywords: `problems`, `terminal` and `git-changes`. Keep typing to filter, then use the arrow keys and Enter.

**2. A few lines, not the whole file.** Add `:first-last` after a file:

| Mode | Prompt |
|---|---|
| **Ask** | `Why must this stay at the end of the file? @/app/main.py:119-122` |

**You should see:** `FastApiMCP` reads the routes that exist when it's created, so a route added below it would not
become an MCP tool. Keep that in mind for step 4: the new route has to go above these lines.

**3. A whole folder.** Bob gets every file directly inside it (not its subfolders):

| Mode | Prompt |
|---|---|
| **Ask** | `What do these tests cover, and what isn't tested yet? @/tests` |

**You should see:** a summary of `test_bookings.py` and `test_flights.py` (and the fixtures in `conftest.py`), then
Bob's list of gaps.

*Recording to come.*
<!-- asset R14: the @ dropdown open, keywords visible; R15: the line-range answer; R16: the folder answer -->

**The rest of the list**, for later steps or your own projects:

| Mention | Bob gets | Try it |
|---|---|---|
| `@git-changes` | Your uncommitted changes: `git status` and the diff | Step 6, before you commit |
| `@terminal` | The last command in the terminal, with its output | When a command fails: `What went wrong? @terminal` |
| `@problems` | The errors and warnings in the Problems panel | `Fix these @problems` |
| `@` + a commit hash | That commit's message and diff | After step 6: type `@`, pick your commit from the list, ask `Explain this commit` |
| `@https://…` | The page, as text | Point Bob at a library's docs |

**Why:** Bob does better with the right context than with more of it. Pointing at three lines gets a sharper answer,
for less, than pointing at the project.

---

## Step 3: Plan the change

| Mode | Prompt |
|---|---|
| **Plan** | *(below)* |

```
Plan a refund preview:
- over 7 days before departure: 100%
- 1 to 7 days: 50%
- under 24 hours, or already cancelled: 0%
You MUST use exactly one explore subagent to read the code before you plan.
Save the plan as plans/refund-preview.md. No code yet.
```

**Bob does:** loads its built-in `create-plan` skill, starts one **explore subagent** that reads the code and reports
back, then writes the plan. Approve the subagent and the file write.

**You should see:** `plans/refund-preview.md`, with a new endpoint, the three rules, a short list of sub-tasks
(model, service, route, tests) and a line about placing the new route **before `FastApiMCP`**. Ours named the
endpoint `GET /bookings/{booking_id}/refund-preview`. Bob may end with a question, like ours: is *exactly* 7 days
100% or 50%? Answer it or move on.

<video src="https://yeokimsiang.github.io/bobathon-galaxium-lite/media/lab3.mp4" poster="https://yeokimsiang.github.io/bobathon-galaxium-lite/media/lab3.jpg" controls muted playsinline preload="none" width="100%"></video>

**Why:** you review the plan before any code exists. Pinning the rules in the prompt keeps the plan small and the
result easy to check. The firm "You MUST use exactly one" wording makes the subagent likely, not certain: Bob
decides. If yours skips it, carry on.

---

## Step 4: Build it and run the tests

**Start a new task first** (the **+** button), so the planning conversation stays out of the build. The `@` mention
brings the plan back in.

| Mode | Prompt |
|---|---|
| Agent | `Build the feature in @/plans/refund-preview.md then run the tests.` |

**Bob does:** reads the plan and the code, edits four files (schema, service, route, tests), loads `verify`, runs the
tests and fixes anything that fails. Approve each edit; look at the first diff before you approve it.

**You should see:** every test passing. Ours: `16 passed` (the 11 you started with, plus 5 new). Your count may
differ; what matters is that they all pass. Booking 2 departs in 3 days, so its preview is 50% of 300.0 = 150.0.

Our first test run failed: the app wouldn't start because of an example the plan had prescribed. Bob read the error,
changed one line in `app/main.py` and ran the tests again. If yours fails first too, let Bob read the error.

<video src="https://yeokimsiang.github.io/bobathon-galaxium-lite/media/lab4.mp4" poster="https://yeokimsiang.github.io/bobathon-galaxium-lite/media/lab4.jpg" controls muted playsinline preload="none" width="100%"></video>

**Why:** done means the tests pass on your screen, not that the code looks right.

> **Bob IDE only: see what changed.** After Bob edits files, a **Changed files** list appears in the chat. Expand it,
> use **Show all**, and click a file to see its diff. Every edit is also a checkpoint you can roll back to
> ([see Rollback](#bob-ide-only-undo-a-change-with-rollback)). *Recording to come.*
<!-- asset R3: an edit approval with its diff; R4: the Changed files list expanded, one diff open -->

---

## Step 5: Check it through an MCP tool

**First, start the app yourself** in a terminal inside the project (Terminal > New Terminal in Bob IDE):

```
# Windows
.venv\Scripts\activate
# macOS / Linux
source .venv/bin/activate

uvicorn app.main:app --port 8000
```

Wait for *Application startup complete*. Leave it running. (If Bob starts the server, it may wait on it forever.)

| Mode | Prompt |
|---|---|
| Agent | `Use the galaxium-lite MCP tool to preview the refund for booking 2.` |

**Bob does:** calls the new tool on the `galaxium-lite` MCP server. Approve the tool call.

**You should see:** booking 2 at 50%, 150.0, the same as step 4. Ours called `galaxium-lite › preview_refund` and got
`{"booking_id": 2, "refund_pct": 50, "refund_amount": 150.0}`. Your tool and field names follow your build.

<video src="https://yeokimsiang.github.io/bobathon-galaxium-lite/media/lab5.mp4" poster="https://yeokimsiang.github.io/bobathon-galaxium-lite/media/lab5.jpg" controls muted playsinline preload="none" width="100%"></video>

**Why:** Bob checks its work through the real running app, not by reading code. The tool exists only because the new
route sits above `FastApiMCP` in `app/main.py`: the rule from step 1, held.

**When you're done:** stop the server (Ctrl + C).

---

## Bob IDE only: review before you commit

Bob IDE has a built-in code review. Run it now, while the feature is still uncommitted.

1. In the chat, type `/review` (or open the **Review** panel).
2. Click **Start Review**. Keep the current branch, with **Include Uncommitted Changes** ticked.
3. Bob lists what it finds in the **Bob Findings** panel. Click a finding to read it.
4. For each one: **Fix with Bob** (Bob starts a task to fix it) or **Dismiss**. If you let Bob fix something, ask it
   to run the tests again before you commit.

**Why:** a second look at the change before it goes into history, from a reviewer that didn't write it.

> *Recording to come:* the Review panel, the findings list, one finding opened, and Fix with Bob or Dismiss.
<!-- asset R5: /review → Start Review options; R6: Bob Findings list; R7: one finding's detail; R8: Fix with Bob (optional) -->

---

## Step 6: Commit it

| Mode | Prompt |
|---|---|
| Agent | `Commit the refund preview feature.` |

**Bob IDE: look first.** Before you send it, try `What's in @git-changes? Anything I shouldn't commit?` in **Ask**
mode. Bob gets your whole uncommitted diff in one go.

**Bob does:** checks `git status` and the diff, runs the tests once more (`verify` again), stages the feature files by
name and commits. Approve each command.

**You should see:** a commit. Ours: `feat: add refund preview endpoint`, 4 files changed. Bob left its own `AGENTS.md`
files and `plans/` uncommitted and said so. Your hash and message will differ. The project has its own git identity
("Ingram Bobathon"), so committing needs no git set-up.

<video src="https://yeokimsiang.github.io/bobathon-galaxium-lite/media/lab6.mp4" poster="https://yeokimsiang.github.io/bobathon-galaxium-lite/media/lab6.jpg" controls muted playsinline preload="none" width="100%"></video>

**Why:** it closes the loop from plan to code to commit, with the tests run one last time first.

**Cost:** our whole lab, set-up included, used about 2.07 Bobcoins (about US$1). In Bob IDE, hover over the usage
gauge in the Bob panel to see yours.
<!-- asset R12: usage gauge tooltip -->


---

## Step 7 (optional): a second opinion in Reviewer mode

This project ships its own **Reviewer** mode (in `.bob/custom_modes.yaml`). It can read code but has no tools to change
it.

| Mode | Prompt |
|---|---|
| **Reviewer** | `Review the refund preview feature: @/app/services/bookings.py @/app/main.py @/tests/test_bookings.py` |

Pick **Reviewer** in the mode list (or type `/reviewer`).

**Bob does:** reads the three files and answers in the shape the mode asks for: a short summary, then findings as
`file:line - issue - suggested fix`, each marked BLOCKING or SUGGESTION. It doesn't edit anything.

**You should see:** *(our recorded run of this step is coming)*.

**Why:** compare it with `/review`. Same goal, different tools: Reviewer is a mode you can write yourself, and its
`groups: [read]` line guarantees it can't touch your code.

---

## Bob IDE only: undo a change with rollback

Bob keeps a checkpoint before each change it makes, so you can undo its work without git.

1. In Agent mode, ask for a small change you don't need:
   `Add a one-line comment at the top of app/db.py saying seed times are relative to when the database was created.`
   Approve the edit.
2. Hover over **that message** in the chat and click its **rollback** button.
3. `app/db.py` is back as it was before that message. Everything Bob changed from that message on is undone.

You can also roll back a single file from the **Changed files** list. Rollback covers only changes Bob made during a
task, not your own edits.

> *Recording to come:* the edit, the rollback button on the message, and the file back to its original state.
<!-- asset R9: hover a user message, rollback button; R10: the file restored; R11: per-file rollback in Changed files (optional) -->

---

## What the `.bob/` kit did

The project's `README.md` explains each file in `.bob/` line by line. Here is where you saw each one at work:

| File | What it is | Where you saw it |
|---|---|---|
| `.bob/skills/verify/SKILL.md` | A skill: how to run and report the tests | Set-up, step 4 and step 6: *Loaded skill verify*, then the `Suite / Result / Detail` table |
| `.bob/mcp.json` | An MCP server: this app's own API as tools | Step 5: `galaxium-lite › preview_refund` |
| `.bob/custom_modes.yaml` | A custom mode: Reviewer, read-only | Step 7 |
| `AGENTS.md`, `.bob/rules-*/` | Not in the kit: Bob wrote them in step 1 | Every later step reads them |

---

## If something goes wrong

| Symptom | Fix |
|---|---|
| Bob doesn't use the kit (no Reviewer mode, no MCP server) | The folder isn't trusted, or you opened a folder above it. Open `galaxium-lite` itself and choose *Yes, I trust the authors*. |
| `python` opens the Microsoft Store | Use `py` instead, or turn the alias off: Settings > Apps > Advanced app settings > App execution aliases. |
| Bob waits forever after starting the server | Start the server yourself in a terminal (step 5), then ask Bob to use the MCP tool. |
| The MCP server shows red, or Bob can't find the tool | The app isn't running on port 8000. Start it, then press **Restart** in Bob's settings > **MCP**. |
| Port 8000 is in use | Stop the other program, or run on another port **and** change the URL in `.bob/mcp.json`. |
| Refund numbers look wrong (booking 3 not at 0%) | Departure times count from when `galaxium.db` was created. Stop the server, delete `galaxium.db`, start again. |
| Bob didn't use a subagent in step 3 | Fine. Bob decides; carry on. |
| You fell behind | Copy `fallback/refund-preview-plan.md` to `plans/refund-preview.md` and go to step 4. |
| Tests fail before you start | Unzip a fresh copy. |

**Start again:** delete the `galaxium-lite` folder and unzip the download again. That restores the code, the git
history and the settings in one go.

**Finished early?** `docs/STRETCH.md` adds origin, destination and price filters to `GET /flights`.

---

Galaxium Lite is based on IBM's [Galaxium Travels](https://github.com/IBM/galaxium-travels) demo (Apache License 2.0),
cut down for the Ingram Micro Bobathon, 29 October 2026. Source:
[github.com/yeokimsiang/bobathon-galaxium-lite](https://github.com/yeokimsiang/bobathon-galaxium-lite).
