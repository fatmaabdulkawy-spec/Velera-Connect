# Velera Connect — leadership simulation

A browser-based simulation of a property developer's internal platform, built
for a leadership programme. A participant signs in as **Commercial Director** of
Velera Residences and works one day: sign the expo plan, discover a discount
committed without them, redeploy five consultants across an exhibition stand,
then teach a colleague to take reservations on a system he has never used.

Each round ends in a live call with a co-trainer playing the person on the other
side of it. The leadership model is never named on screen — participants
practise the behaviours and read the result in how the conversation ends.

- **Round 1 — Permission.** Connection, attitude, integrity.
- **Round 2 — Production.** Relationships, vision, motivation.
- **Round 3 — People development.** Mentor, monitor.

Everything is fictional. Velera Residences, the Coast Property Expo, their
people and their figures are invented, and the figures are engineered so the
arithmetic resolves cleanly in a workshop.

---

## What's here

```
.
├── index.html     the simulation (participants)
├── scoring.html   the co-trainer scoring app
├── vercel.json    static hosting config (clean URLs: /scoring)
└── README.md
```

Both are single self-contained files — CSS, JavaScript and images inlined. No
build step, no dependencies, no package manager.

```bash
open index.html      # macOS
xdg-open index.html  # Linux
start index.html     # Windows
```

To host, drop them on any static host. The only external requests are the Inter
webfont from Google Fonts and the Teams meeting links, both of which degrade
gracefully offline.

---

## The three views in index.html

The file carries three screens, selected by the URL fragment:

| Fragment | View | Who opens it |
|---|---|---|
| *(none)* | The simulation | Participants |
| `#score` | Co-trainer scoring, legacy copy | superseded by `scoring.html` |
| `#board` | Trainer results board | The trainer |

`scoring.html` is the current scoring app and is a **separate artifact with its
own data store**. A published page can only read its own store, so the board in
`index.html` cannot see scores written by `scoring.html`. Run them as two tabs.

---

## How a round runs

```
Sign in (name + team)
   └─ welcome briefing — the role, the project, the people
        └─ Today            figures · to-do list, one item open
             └─ Approvals   edit the ceiling and allocation, then sign
                  🔔 desktop mail notification, bottom left
                  └─ Mailbox    Finance email · approval record · forwarded chain
                       📅 New meeting → invite N. Sabry      ← Round 1 call
                       └─ log what you agreed
                            └─ Stand Deployment   move 5 consultants over 4 posts
                                 📅 call T. Nabil            ← Round 2 call
                                 └─ sign, then log the call
                                      ⚡ 06:40 · N. Sabry in hospital
                                      └─ Reservations
                                           📞 call 1 — show M. Serry   ← Round 3a
                                           📞 call 2 — watch him do it ← Round 3b
                                                └─ log, then the closing message
```

Tabs stay locked until the round before them closes, so discovery always
follows commitment.

---

## Editing the things you are most likely to change

All of it is in `index.html`. Search for the anchor shown.

### Meeting links

Search for `teams.microsoft.com`. One occurrence, used by every **Join call**
button. Replace with per-round links if each co-trainer needs their own room.

### The cast and the org chart

```js
var OPS=[ ... ]   // the four functions reporting to the participant
```

Names are initial + surname throughout (`N. Sabry`, `T. Nabil`, `M. Serry`).
Keep that convention — the co-trainer scripts let the actor pick a first name
behind the initial, so a woman plays Nour Sabry and a man plays Nader Sabry
without anything in the system changing.

Current structure: the participant is **Commercial Director**, reporting to
**H. Doss, Managing Director**. Reporting to them: **N. Sabry** (Head of Sales,
may discount to 5%), **T. Nabil** (Sales Operations Manager, owns the
deployment), **D. Aziz** (CRM & Contracts, raises requests, approves nothing)
and **L. Hakim** (Marketing).

### Round 1 — the approved plan and the breach

```js
var APPROVED={ceil:5, alloc:12};
var UNITS_OVER=6, PTS_OVER=3, TICKET=11;   // EGP millions
```

The participant edits the ceiling and allocation before signing. The commitment
is always `+3` points and `+6` units above whatever they approved, so no number
lets them escape the round. Cost = `UNITS_OVER × PTS_OVER × TICKET` = EGP 1.98m,
deliberately small — three-tenths of one per cent of the expo — so the call
cannot become an argument about money.

### Round 2 — the deployment

```js
var ST=[ ... ]    // four posts: share of value and duration
var PPL=[ ... ]   // five consultants: appraisal (a) and expos worked (f)
PPL.forEach(function(p){ p.x = p.a + p.f; p.A = p.x >= 7; });
```

`p.x` is the strength weight: appraisal plus expos worked. Tier A is 7 or above,
marked with a gold dot. The submitted plan opens at roughly **70% of team
strength on 13% of the value**.

The model answer — everybody back to their own post — brings it to about 21%.
It is deliberately **not** in the build; it lives in the co-trainer script so
participants cannot find it.

### Round 3 — the units

```js
var UNITS=[ {no:'C-204', t:'...', rate:10, ok:true}, ... ]
```

`rate` is the down-payment percentage and filters the dropdown (10, 15, or 100
for cash). `ok:false` makes a unit refuse the reservation because it is held for
legal review — **C-211** is the planted failure, and demonstrating that refusal
is the most useful thing the participant can show.

### Copy in the modals

Every popup is built by the `modal({...})` helper and the `meetModal(...)`
wrapper. Both take plain HTML strings, so text changes need no other edits.

---

## Shared data

`index.html` declares the `db` and `user` capabilities and writes one document
per participant to `runs/<user id>`: their name, team, the ceiling and
allocation they approved, their final deployment, and their three written logs.
The `#board` view subscribes to that collection.

`scoring.html` declares `db` and writes one document per team and round to
`scores/<team>_r<n>`. It also keeps a copy in `localStorage`, so a co-trainer
whose account cannot write to the shared store still sees their own scores and
can send them on with **Copy results**.

Sharing only works for people inside the owner's organization with at least
Contributor access. Outside viewers read but never write. Worth testing each
co-trainer's device once before a session.

---

## Facilitator notes

- **Screen sharing matters in rounds 2 and 3.** Both call modals carry an amber
  instruction to share, because the participant has to work the deployment or
  the reservation form in front of the co-trainer.
- **Nothing resets by itself.** Reloading starts the simulation over for that
  person; shared data persists until it is cleared.
- **The caps live in the scripts, not the code.** The system will happily let a
  participant sign a bad plan — the consequence is played by the co-trainer.

The three co-trainer scripts (character, what unlocks them, the Egyptian Arabic
lines, the endings) and the printed handouts — the expo visitor guide, the
Velera brochure and the stand plan — are maintained outside this repository.

---

## Deploying on Vercel

The repo is a plain static site: no build step, no framework.

1. In Vercel, **Add New → Project** and import this GitHub repository.
2. Framework preset **Other**. Leave Build Command and Output Directory empty.
3. Deploy. Pages:
   - `https://<project>.vercel.app/` the simulation
   - `https://<project>.vercel.app/scoring` the co-trainer scoring app
   - `https://<project>.vercel.app/#board` the trainer board

**Shared data does not work on Vercel.** The `db` and `user` capabilities are
provided by the claude.ai artifact runtime (`window.claude`). Outside it both
pages detect that it is missing and fall back:

- The simulation runs completely; nothing is written centrally, so `#board`
  stays empty.
- `scoring.html` keeps scores in each co-trainer's browser. Each co-trainer
  sends theirs to the trainer with **Copy results**.

To get live shared scores and the board on Vercel, the `claude.use('db')` calls
need replacing with a real backend (e.g. Firebase, Supabase or Vercel KV
behind a small API route).

## Pushing it

```bash
git init
git add .
git commit -m "Velera leadership simulation"
gh repo create velera-simulation --private --source=. --push
```

Images are base64-inlined in `index.html`, which keeps it portable and
offline-capable at the cost of a large diff whenever a picture changes. If you
expect to iterate on the visuals, extract them to `assets/` and switch the `src`
attributes to relative paths — nothing in the code depends on them being inline.
