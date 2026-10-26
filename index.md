---
layout: default
title: "Galaxium Lite: the IBM Bob lab"
---

# Galaxium Lite: the IBM Bob lab, step by step

Take one feature from first look to committed code with **IBM Bob**, on a small Python app. You'll give Bob context,
ask it questions, have it plan, build and test the change, check it through a live tool, review it and commit it.
Every step happens in the Bob chat, and Bob asks before it acts.

**You need:** about an hour, Bob IDE, Python 3.11 or later, and a few Bobcoins (our Bob Shell run of the lab used
about 2, roughly US$1; in Bob IDE, one long task costs more, so start a new task where the steps say so).

**The feature:** a *refund preview*. Before cancelling a booking, a traveller can see what they'd get back:

| Time to departure | Refund |
|---|---|
| more than 7 days | 100% |
| 1 to 7 days | 50% |
| under 24 hours, or already cancelled | 0% |

**Try the app first:** [the app you're starting with](https://yeokimsiang.github.io/bobathon-galaxium-before/) runs in
your browser, nothing to install. Cancel a booking: it can't tell you what you'd get back. That's what you'll add.

> **About the videos.** They're cut from a real run of this lab in Bob IDE, 21 Oct 2026 (the rollback from an earlier
> run the same day). Waiting is sped up (the
> bar under the video says by how much) or cut, the camera zooms in on what matters, and a few things are blurred:
> other extensions, the editor, and anything personal. Nothing Bob said or did is changed. **Your Bob will word
> things differently**, choose different names and maybe take a different route. That's normal. Check your result
> against the "You should see" line, not against the exact wording.

**Contents:** [Before you start](#before-you-start) · [Set-up](#set-up-let-bob-set-itself-up) ·
[1. Context](#step-1-give-bob-context) · [2. Ask](#step-2-understand-before-changing) ·
[More `@`](#bob-ide-only-mention-exactly-what-bob-needs) · [3. Plan](#step-3-plan-the-change) ·
[4. Build](#step-4-build-it-and-run-the-tests) ·
[5. MCP](#step-5-check-it-through-an-mcp-tool) · [Review](#bob-ide-only-review-before-you-commit) ·
[Rollback](#bob-ide-only-take-back-a-change-with-rollback) · [6. Commit](#step-6-commit-it) ·
[7. Reviewer mode](#step-7-optional-a-second-opinion-in-reviewer-mode) · [The `.bob/` kit](#what-the-bob-kit-did) ·
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

> **Approvals.** Each action arrives as a card under *Tools awaiting approval*, showing the command, edit or tool
> call. **Approve once** runs it, **Reject** stops it. **Approve for task** (or *Approve edit tools for task* and the
> like) runs it and lets that kind of action through for the rest of the task. Commands are the exception: in our run
> Bob still asked before each new one. Approve once while you learn, so you see every step; in the videos we chose
> *for task* to keep them short.

---

## Set-up: let Bob set itself up

| Mode | Prompt |
|---|---|
| Agent | `Create a virtual environment, install requirements.txt and run the tests.` |

**Bob does:** looks around the project, creates a virtual environment (usually `.venv`), installs the packages and
runs the tests: three commands, each one shown to you first. Approve each command. Bob may also load the project's
`verify` skill (*Used skill verify*).

**You should see:** `11 passed`. Ours ended *Everything is set up and working*, with *Tests — 11 passed, 0 failed*.

<video src="https://yeokimsiang.github.io/bobathon-galaxium-lite/media/ide-setup.mp4" poster="https://yeokimsiang.github.io/bobathon-galaxium-lite/media/ide-setup.jpg" controls muted playsinline preload="none" width="100%"></video>

**Why:** everyone starts from a known-good state, and it's your first practice at chatting and approving.

**If it's slow:** installing packages took about two minutes in our run, and longer on a slow connection. In
another run Bob reported a time-out but carried on.
If the tests don't pass, see [Help](#if-something-goes-wrong).

---

## Step 1: Give Bob context

| Mode | Prompt |
|---|---|
| Agent | `/init` |

**Bob does:** reads the project (*Explored 3 folders, 10 files* in our run) and writes its briefing: an `AGENTS.md`
in the project root, plus one per mode in `.bob/rules-agent/`, `.bob/rules-ask/` and `.bob/rules-plan/`. Approve the
writes.

**You should see:** a new `AGENTS.md`. Open it. A good one mentions:

- the FastAPI booking API and its MCP tools at `/mcp`
- the `app/`, `tests/` and `.bob/` layout, `uvicorn app.main:app` to run and `pytest -q` to test
- inject `now` rather than calling `datetime.now()` in services
- give every route an `operation_id`, and keep `FastApiMCP` last in `app/main.py`

**Red flags:** it describes an ORM, Docker or a frontend. This app has none.

<video src="https://yeokimsiang.github.io/bobathon-galaxium-lite/media/ide-init.mp4" poster="https://yeokimsiang.github.io/bobathon-galaxium-lite/media/ide-init.jpg" controls muted playsinline preload="none" width="100%"></video>

**Why:** Bob reads `AGENTS.md` at the start of every chat, so it stops guessing about the project. In our run Bob
turned the warning comment in `app/main.py` into a rule: *`FastApiMCP(app)` + `mcp.mount_http()` must stay at the
bottom of `main.py`. Any route defined after that line is invisible to MCP.* Watch for it in step 4.

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
today**. (Ours said: "There's no refund calculation — cancellation only flips the status field".)

<video src="https://yeokimsiang.github.io/bobathon-galaxium-lite/media/ide-ask.mp4" poster="https://yeokimsiang.github.io/bobathon-galaxium-lite/media/ide-ask.jpg" controls muted playsinline preload="none" width="100%"></video>

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
become an MCP tool. Keep that in mind for step 4: the new route has to go above these lines. Ours confirmed it had
read only those lines (*Read file app/main.py (119-122)*) and drew the order as a diagram.

**3. A whole folder.** Bob gets every file directly inside it (not its subfolders):

| Mode | Prompt |
|---|---|
| **Ask** | `What do these tests cover, and what isn't tested yet? @/tests` |

**You should see:** a summary of `test_bookings.py` and `test_flights.py` (and the fixtures in `conftest.py`), then
Bob's list of gaps. Ours also read the code the tests call, to check what isn't covered, and listed gaps such as
cancelling an unknown booking and booking a flight that has already left.

<video src="https://yeokimsiang.github.io/bobathon-galaxium-lite/media/ide-mention.mp4" poster="https://yeokimsiang.github.io/bobathon-galaxium-lite/media/ide-mention.jpg" controls muted playsinline preload="none" width="100%"></video>

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
endpoint `GET /bookings/{booking_id}/refund-preview` and listed four sub-tasks, the last one 5 new tests. Bob may end
with questions, like ours: is *exactly* 7 days 100% or 50%? Should an already-cancelled booking get a 0% preview or an
error? Answer them or move on.

<video src="https://yeokimsiang.github.io/bobathon-galaxium-lite/media/ide-plan.mp4" poster="https://yeokimsiang.github.io/bobathon-galaxium-lite/media/ide-plan.jpg" controls muted playsinline preload="none" width="100%"></video>

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

**Bob does:** reads the plan and the code, edits four files (schema, service, route, tests), runs the tests and fixes
anything that fails. Approve each edit; look at the first diff before you approve it. Ours loaded the built-in
`executing-plans` skill and worked through the plan as a todo list.

**You should see:** every test passing. Ours: `16/16 passed` (the 11 you started with, plus 5 new). Your count may
differ; what matters is that they all pass. Booking 2 departs in 3 days, so its preview is 50% of 300.0 = 150.0.

Ours passed on the first run. In another run the first test run failed and Bob read the error, changed one line and
ran the tests again. If yours fails first, let Bob read the error.

<video src="https://yeokimsiang.github.io/bobathon-galaxium-lite/media/ide-build.mp4" poster="https://yeokimsiang.github.io/bobathon-galaxium-lite/media/ide-build.jpg" controls muted playsinline preload="none" width="100%"></video>

**Why:** done means the tests pass on your screen, not that the code looks right.

> **Bob IDE only: see what changed.** After Bob edits files, a bar above the chat box counts them (*1 file changed*).
> Click **Show all** and then a file to see its diff. Every change is also a checkpoint you can roll back to
> ([see Rollback](#bob-ide-only-take-back-a-change-with-rollback)).

![The bar above the Bob IDE chat box: 1 file changed, with Undo all and Show all](https://yeokimsiang.github.io/bobathon-galaxium-lite/media/ide-changed-files.png)
<!-- asset R3: an edit approval with its diff; R4: Show all expanded, one diff open -->

---

## Step 5: Check it through an MCP tool

The project's `.bob/mcp.json` connects Bob to this app's own API as MCP tools. For that, the app must be running,
and running **the code you just built**.

**1. Start the app yourself.** Open a terminal in Bob IDE (Terminal > New Terminal) and run:

```
.venv\Scripts\python -m uvicorn app.main:app --port 8000
```

On macOS or Linux: `.venv/bin/python -m uvicorn app.main:app --port 8000`. If set-up named the folder something
other than `.venv`, use that name. Leave the terminal open: the app runs until you stop it.

Why not ask Bob? An app server never finishes, so Bob would sit waiting on it. In our run Bob tried a background
start, which exited without a word, then a normal start, which kept the task waiting until we stopped it.

**2. Check Bob can see it:** Bob's settings menu > **MCP**. `galaxium-lite` should say **Connected**. If it still says
Disconnected, press refresh (the round arrow above the list, right of **+**). Ours went from Disconnected to
Connected in two seconds.

**3. Use the tool.** The same task is fine.

| Mode | Prompt |
|---|---|
| Agent | `Use the galaxium-lite MCP tool to preview the refund for booking 2.` |

**Bob does:** calls the new tool on the `galaxium-lite` MCP server. Approve the tool call.

**You should see:** booking 2 at 50%, 150.0, the same as step 4. Ours showed *Ran Preview Refund (galaxium-lite)*,
then: *Booking 2 qualifies for a 50% refund — $150.00.* Your tool and field names follow your build.

<video src="https://yeokimsiang.github.io/bobathon-galaxium-lite/media/ide-mcp.mp4" poster="https://yeokimsiang.github.io/bobathon-galaxium-lite/media/ide-mcp.jpg" controls muted playsinline preload="none" width="100%"></video>

**Why:** Bob checks its work through the real running app, not by reading code. The tool exists only because the new
route sits above `FastApiMCP` in `app/main.py`: the rule from step 1, held.

**When you're done with the lab:** press Ctrl+C in the app's terminal.

---

## Bob IDE only: review before you commit

Bob IDE has a built-in code review. Run it now, while the feature is still uncommitted.

1. In the chat, type `/review`, or open **Review** in the Source Control side bar.
2. Leave **Review against** as it is, keep **Include Uncommitted Changes** ticked, and click **Start Review**.
3. Bob lists what it finds in the **Bob Findings** tab, in the bottom panel next to Terminal. Click a finding to read
   it.
4. For each one: **Fix with Bob** or **Dismiss**. **Fix with Bob** starts a new task named after the finding (ours:
   *Fix review issue in bookings.py*), makes the change and offers to run the tests. Let it.

**You should see:** a short list of findings, each with a file and lines. Ours had two: *Departed bookings silently
return 0% with no indication flight has passed* (a warning) and *Missing test for the exact 7-day boundary*. Fix with
Bob on the first added a guard and a test, then ran the tests: 14 passed. But its guard makes a flight that has
already left an error (409), where our rules say 0%. A fix can change more than you asked for: read it, and roll it
back if you don't want it (next section).

<video src="https://yeokimsiang.github.io/bobathon-galaxium-lite/media/ide-review.mp4" poster="https://yeokimsiang.github.io/bobathon-galaxium-lite/media/ide-review.jpg" controls muted playsinline preload="none" width="100%"></video>

The rollback at the end of the video comes from our first run that day; the rest is the run the other videos come
from. In that run we kept the fix, so it's in the step 6 commit.

**Why:** a second look at the change before it goes into history, from a reviewer that didn't write it.

---

## Bob IDE only: take back a change with rollback

Bob keeps a checkpoint before every change it makes. Rollback returns your files, and the chat, to how they were
before one of your messages. Try it on the review fix, especially if, like ours, it does something your rules didn't
ask for. The video in the review section ends with one.

1. Scroll to the message that started the change: for a review fix, the first message of the *Fix review issue…*
   task.
2. Click the **rollback** button on that message (the round arrow; its tooltip says *Rollback and delete this chat*).
3. Bob asks **Roll back to this point?** *This will undo all changes and messages made after this message.* Click
   **Confirm**.

If the message has no round arrow (in one of our runs, the fix task's first message didn't), ask Bob to undo the fix
instead, and try rollback with the small change at the end of this section.

![A message in the Bob IDE chat with its rollback button highlighted; the tooltip reads Rollback and delete this chat](https://yeokimsiang.github.io/bobathon-galaxium-lite/media/ide-rollback-button.png)

![The confirmation: Roll back to this point? This will undo all changes and messages made after this message. Confirm or Cancel](https://yeokimsiang.github.io/bobathon-galaxium-lite/media/ide-rollback-confirm.png)

**You should see:** the file back as it was before that message, and the messages after it gone.

**Roll back before you commit.** Rollback puts files back; it doesn't touch git. In our first run the fix had already
been committed: after the rollback the commit was still there, and the undo showed up as an uncommitted change.
Rollback covers only changes Bob made, not your own edits.

**No review fix to undo?** Ask for a small change you don't need, then roll it back:
`Add a one-line comment at the top of app/db.py saying seed times are relative to when the database was created.`
<!-- asset R10: the file restored (open it, or Source Control showing no change) -->

---

## Step 6: Commit it

| Mode | Prompt |
|---|---|
| Agent | `Commit the refund preview feature.` |

**Bob IDE: look first.** Before you send it, try `What's in @git-changes? Anything I shouldn't commit?` in **Ask**
mode. Bob gets your whole uncommitted diff in one go.

**Bob does:** runs the tests once more (with `verify`), checks `git status` and the diff, stages the feature files by
name and commits. Approve each command. Ours loaded the built-in `finishing-a-development-branch` skill, which asks how
to finish: commit locally, push, keep as is or discard. We answered *Commit to main locally*.

**You should see:** a commit. Ours: `feat: add refund preview endpoint`, 5 files: the four feature files and the plan.
The tests ran first: 17 passed, one more than step 4 because the review fix added a test. Bob left its own `AGENTS.md`
files uncommitted. Your hash and message will differ. The project has its own git identity ("Ingram Bobathon"), so
committing needs no git set-up.

**Read the message before you approve.** Ours got the rules wrong: it said 100% over 24 hours and 50% at 2 to 24
hours, while the code it was committing used 7 days and 1 day, as planned. We approved it as written, so the wrong
message is in our history. If yours is wrong, reject the card and tell Bob what to fix.

<video src="https://yeokimsiang.github.io/bobathon-galaxium-lite/media/ide-commit.mp4" poster="https://yeokimsiang.github.io/bobathon-galaxium-lite/media/ide-commit.jpg" controls muted playsinline preload="none" width="100%"></video>

**Why:** it closes the loop from plan to code to commit, with the tests run one last time first.

**Cost:** our Bob Shell run of the whole lab, set-up included, used about 2.07 Bobcoins (about US$1). In Bob IDE,
each task shows what it has cost so far in its header, next to how full its context is.


---

## Step 7 (optional): a second opinion in Reviewer mode

This project ships its own **Reviewer** mode (in `.bob/custom_modes.yaml`). It can read code but has no tools to change
it.

| Mode | Prompt |
|---|---|
| **Reviewer** | `Review the refund preview feature: @/app/services/bookings.py @/app/main.py @/tests/test_bookings.py` |

Pick **Reviewer** in the mode list (or type `/reviewer`).

**Bob does:** reads the three files, plus any others it needs, and answers in the shape the mode asks for: a short
summary, then findings as `file:line - issue - suggested fix`, each marked BLOCKING or SUGGESTION. It doesn't edit
anything. There are no approvals to give, because there's nothing to change.

**You should see:** a one-paragraph summary, then a list of findings, each with a file and line. Ours read six files,
changed none, asked for no approvals and listed 8 findings, 4 marked BLOCKING. (In our Bob Shell run of the lab, the
same review cost 0.08 Bobcoins.) Two worth a look:

- **Exactly 7 days gets 50%, not 100%.** Bob calls that an off-by-one bug, and the missing test for it a blocker.
  Our rules say 1 to 7 days is 50%, so the code is right. It's the question Bob asked in step 3, though, and the
  test is worth adding.
- **The summary and the list disagree.** Ours said "two blocking bugs" and then marked four BLOCKING. Read the list.

A reviewer gives you findings, not orders. You decide which ones matter.

<video src="https://yeokimsiang.github.io/bobathon-galaxium-lite/media/ide-reviewer.mp4" poster="https://yeokimsiang.github.io/bobathon-galaxium-lite/media/ide-reviewer.jpg" controls muted playsinline preload="none" width="100%"></video>

**Why:** compare it with `/review`. Same goal, different tools: Reviewer is a mode you can write yourself, and its
`groups: [read]` line guarantees it can't touch your code.

---

## What the `.bob/` kit did

The project's `README.md` explains each file in `.bob/` line by line. Here is where you saw each one at work:

| File | What it is | Where you saw it |
|---|---|---|
| `.bob/skills/verify/SKILL.md` | A skill: how to run and report the tests | Whenever Bob runs the tests; ours used it in step 6: *Used skill verify*, then the `Suite / Result / Detail` table |
| `.bob/mcp.json` | An MCP server: this app's own API as tools | Step 5: *Ran Preview Refund (galaxium-lite)* |
| `.bob/custom_modes.yaml` | A custom mode: Reviewer, read-only | Step 7 |
| `AGENTS.md`, `.bob/rules-*/` | Not in the kit: Bob wrote them in step 1 | Every later step reads them |

---

## If something goes wrong

| Symptom | Fix |
|---|---|
| Bob doesn't use the kit (no Reviewer mode, no MCP server) | The folder isn't trusted, or you opened a folder above it. Open `galaxium-lite` itself and choose *Yes, I trust the authors*. |
| `python` opens the Microsoft Store | Use `py` instead, or turn the alias off: Settings > Apps > Advanced app settings > App execution aliases. |
| Set-up: Bob can't create `.venv` (access denied, or files in use) | Something still runs from it, usually the app from an earlier try. Stop it (Ctrl+C in its terminal, or close old terminals). A different folder name is fine too: later steps work with whatever set-up made. |
| Bob sits waiting on a command that never ends (for example, it started the app itself) | Press the square stop button in the chat box. Start the app yourself, as in step 5. |
| Bob says it can't use MCP tools, or calls the API over HTTP instead | The app wasn't running when Bob looked, or the MCP status is out of date. Start the app (step 5), press refresh in Bob's settings > **MCP** until `galaxium-lite` says Connected, then ask again. |
| The MCP tool is missing, or answers like before your change | The app is running old code: it was started before step 4. Stop it (Ctrl+C), start it again, then refresh in Bob's settings > **MCP**. |
| Port 8000 is in use by another program | Stop that program, or run on another port **and** change the URL in `.bob/mcp.json`. |
| Refund numbers look wrong (booking 3 not at 0%) | Departure times count from when `galaxium.db` was created. Stop the app, delete `galaxium.db`, start the app again. |
| Bob didn't use a subagent in step 3 | Fine. Bob decides; carry on. |
| You fell behind | Copy `fallback/refund-preview-plan.md` to `plans/refund-preview.md` and go to step 4. |
| Tests fail before you start | Unzip a fresh copy. |

**Start again:** delete the `galaxium-lite` folder and unzip the download again. That restores the code, the git
history and the settings in one go.

**Here's what Bob built in our run:** [the finished app](https://yeokimsiang.github.io/bobathon-galaxium-after/),
running in your browser (press Cancel on a booking), and
[the code Bob wrote](https://github.com/yeokimsiang/bobathon-galaxium-after/commit/19486bfa370fdb28675f77677906a09f442cdc17).
Compare it with yours: your Bob will have picked different names and wording.

**Finished early?** `docs/STRETCH.md` adds origin, destination and price filters to `GET /flights`.

---

Galaxium Lite is based on IBM's [Galaxium Travels](https://github.com/IBM/galaxium-travels) demo (Apache License 2.0),
cut down for the Ingram Micro Bobathon, 29 October 2026. Source:
[github.com/yeokimsiang/bobathon-galaxium-lite](https://github.com/yeokimsiang/bobathon-galaxium-lite).
