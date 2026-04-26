# 🤖 Wumpus World — Dynamic Logic Agent

A **Knowledge-Based AI Agent** that navigates a Wumpus World-style grid using **Propositional Logic** and **Resolution Refutation** to deduce safe cells in real time.

> **Live Demo:** [your-vercel-url.vercel.app](https://dynamic-wumpus-logic-agent.vercel.app/)  
> **GitHub:** [github.com/yourusername/wumpus-logic-agent](https://github.com/Faiq-Ali10/Dynamic-Wumpus-Logic-Agent)  

---

## 📌 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [AI / Logic Engine](#ai--logic-engine)
- [CNF Conversion & Resolution Refutation](#cnf-conversion--resolution-refutation)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
- [How to Use](#how-to-use)
- [Tech Stack](#tech-stack)
- [Assignment Info](#assignment-info)

---

## Overview

This project implements a **Knowledge-Based Agent** that autonomously navigates a configurable Wumpus World grid. The agent:

- Receives **dynamic percepts** (Breeze, Stench, Glimmer) as it moves
- Maintains a **Propositional Logic Knowledge Base (KB)**
- Uses **Resolution Refutation** to prove cells safe before entering them
- Shoots its arrow when it can logically confirm the Wumpus location
- Seeks and retrieves gold, then returns to the start cell

The world is randomly generated at the start of every episode — the agent has zero prior knowledge of hazard locations.

---

## Features

| Feature | Description |
|---|---|
| Dynamic Grid | User-configurable rows × columns (3×3 up to 8×8) |
| Random Hazards | Pits and Wumpus placed randomly each episode |
| Percept Engine | Breeze near pits, Stench near Wumpus, Glimmer on gold |
| Propositional KB | TELL/ASK interface with biconditional rules |
| Resolution Refutation | Full CNF clause resolution to prove cell safety |
| Arrow Logic | Agent shoots when Wumpus location is uniquely resolved |
| Real-Time Metrics | Move count, inference count, safe cell count |
| Step / Auto Run | Manual step-through or automated traversal |
| Reveal Mode | Toggle to expose hidden hazards for debugging |
| Color-Coded Grid | Visual encoding of cell knowledge state |

---

## AI / Logic Engine

### Knowledge Base (KB)

The KB is a set of propositional sentences. The agent uses two operations:

**TELL** — add a new sentence to the KB after sensing a cell:

```
TELL: BREEZE_1_0           // breeze perceived at (1,0)
TELL: RULE: B_1_0 ⟺ P_2_0 ∨ P_1_1 ∨ P_0_0
TELL: ¬STENCH_1_0          // no stench → Wumpus not adjacent
TELL: ¬W_2_0               // Wumpus at (2,0) ruled out
TELL: ¬W_1_1               // Wumpus at (1,1) ruled out
```

**ASK** — query before every move to check if a cell is safe:

```
ASK: ¬P_2_0 ∧ ¬W_2_0  →  TRUE (safe to enter)
ASK: ¬P_1_1 ∧ ¬W_1_1  →  UNKNOWN (do not enter)
```

### Biconditional Rules

When the agent perceives a **Breeze** at cell (r, c), it TELLs the KB:

```
B(r,c) ⟺ P(r-1,c) ∨ P(r+1,c) ∨ P(r,c-1) ∨ P(r,c+1)
```

When there is **no Breeze** at (r, c), the agent immediately infers:

```
¬P(r-1,c) ∧ ¬P(r+1,c) ∧ ¬P(r,c-1) ∧ ¬P(r,c+1)
```

The same symmetric rules apply for **Stench** and Wumpus location.

---

## CNF Conversion & Resolution Refutation

### Step 1 — Convert KB to Conjunctive Normal Form (CNF)

Each fact in the KB is treated as a unit clause (a clause with one literal). Biconditional rules are split into implications, then into disjunctive clauses:

```
B_1_0 ⟺ P_2_0 ∨ P_1_1
  →  (¬B_1_0 ∨ P_2_0 ∨ P_1_1)  ∧  (¬P_2_0 ∨ B_1_0)  ∧  (¬P_1_1 ∨ B_1_0)
```

All clauses are stored as sets of literals.

### Step 2 — Negate the Query

To prove `¬P_2_0` (cell (2,0) has no pit), we assume the negation is true:

```
Assume: P_2_0
Add clause {P_2_0} to the clause set
```

### Step 3 — Resolution Loop

The resolver iterates over all pairs of clauses, looking for complementary literals:

```
{P_2_0} and {¬P_2_0, B_1_0}
  → Resolve on P_2_0 / ¬P_2_0
  → Resolvent: {B_1_0}

{B_1_0} and {¬B_1_0}   (KB contains ¬BREEZE_1_0)
  → Resolve on B_1_0 / ¬B_1_0
  → Resolvent: {}  ← EMPTY CLAUSE (contradiction!)
```

An **empty clause (□)** is the contradiction. It means our assumption (`P_2_0`) is false, so `¬P_2_0` is **entailed by the KB**. The cell is proven safe.

If no contradiction is found within the iteration limit, the query is **UNKNOWN** and the agent avoids that cell.

### Agent Decision Policy

```
for each adjacent cell (nr, nc):
    if KB ⊨ ¬P(nr,nc) ∧ ¬W(nr,nc):
        mark as SAFE → prefer unvisited ones
    else if stench and arrows > 0 and only one candidate:
        SHOOT → mark wumpus dead → cell becomes safe
    else:
        add to FRONTIER (risky, avoided if possible)

move to best safe cell (unvisited preferred over visited)
if has gold: pathfind back to (0,0) via safe cells
```

---

## Project Structure

```
wumpus-logic-agent/
├── index.html          # Main app (single-file, self-contained)
├── README.md           # This file
└── assets/
    └── screenshot.png  # App screenshot
```

> This project is a single self-contained `index.html` for easy Vercel deployment and code review. All logic (KB, resolution engine, rendering) is documented inline.

---

## Getting Started

### Prerequisites

- Any modern web browser (Chrome, Firefox, Safari, Edge)
- No build tools or dependencies required

### Run Locally

```bash
git clone https://github.com/yourusername/wumpus-logic-agent.git
cd wumpus-logic-agent
open index.html
```

Or serve with any static file server:

```bash
npx serve .
# or
python3 -m http.server 8080
```

Then open `http://localhost:8080` in your browser.

### Deploy to Vercel

```bash
npm install -g vercel
vercel --prod
```

Vercel detects the static `index.html` and deploys instantly. Paste the generated URL on the first page of your PDF report.

---

## How to Use

1. **Configure** — Set grid dimensions (Rows × Cols) and number of Pits in the top panel
2. **New Episode** — Click to randomly generate the world; agent starts at (0,0)
3. **Step** — Advance the agent one move at a time and watch the KB log update
4. **Auto Run** — Let the agent run continuously; adjust speed with the slider
5. **Reveal All** — Toggle to expose all hidden pits, Wumpus, and gold
6. **KB Log** — Watch real-time TELL / ASK / Resolution entries scroll in the panel

### Cell Color Legend

| Color | Meaning |
|---|---|
| 🔵 Blue | Agent's current position |
| 🟢 Green | Proven safe, not yet visited |
| 🌿 Light green | Visited and safe |
| ⬜ Gray | Unknown / unvisited |
| 🔴 Red | Confirmed pit |
| 🟥 Dark red | Confirmed Wumpus |
| 🟡 Yellow | Gold location (reveal mode only) |

---

## Tech Stack

| Layer | Technology |
|---|---|
| UI | Vanilla HTML / CSS / JavaScript |
| Logic Engine | Custom Propositional Resolution (zero external libraries) |
| Deployment | Vercel (static hosting) |
| Version Control | Git / GitHub |

No external JS libraries are used. The resolution engine, KB, and renderer are all implemented from scratch in under 300 lines of JavaScript.

---

## Assignment Info

| Field | Detail |
|---|---|
| Course | Artificial Intelligence |
| Assignment | A6 — Dynamic Wumpus Logic Agent |
| Submission format | `AI_A6_[RollNumber].zip` |
| Required links | Live URL · GitHub · LinkedIn |
| Components | Source code + PDF Report |

---

## License

Submitted as academic coursework. All logic implementation is original.
