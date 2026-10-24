# Web App (React Frontend)

## What Is This? (Plain English)

This is the website that students actually use. It's like a course catalog on steroids — you can browse all 235 De Anza pathways, search by name or course, filter by degree type, and most importantly, **check if your chosen courses clash in time** before you register.

There's also an AI chat assistant (like ChatGPT but specifically trained on De Anza data) that you can ask questions like "Does MATH 1A conflict with CHEM 1A in Fall 2025?"

The "Conflict Dashboard" button in the filter row opens the administrative analysis dashboard in a new tab.

## Tech Stack

| Technology | Why We Use It |
|-----------|---------------|
| **React 19** | Makes the UI interactive (clicking, filtering, updating without page reloads) |
| **Vite 8** | Dev server + build tool (makes development fast) |
| **Plain CSS** | Styling — no extra libraries needed |

## File Structure

```
web-app/
├── index.html              # The single HTML page that loads everything
├── vite.config.js          # Build tool config
├── package.json            # Lists all JavaScript dependencies
├── public/
│   ├── dashboard.html      # The full analysis dashboard (served as static file)
│   ├── favicon.svg         # Browser tab icon
│   └── icons.svg           # UI icons
└── src/
    ├── main.jsx            # Entry point — mounts React app into the page
    ├── App.jsx             # Main component — holds all the state and layout
    ├── config.js           # API URLs and term definitions
    ├── styles.css          # All the CSS styles
    ├── chat/
    │   ├── chatEngine.js   # Simple chatbot (pattern matching, no AI)
    │   └── chatAgent.js    # Smart chatbot (talks to AWS AI service)
    ├── components/
    │   ├── SearchFilter.jsx    # The search bar + dropdowns + dashboard link
    │   ├── ProgramList.jsx     # Grid of pathway cards
    │   ├── ProgramCard.jsx     # One pathway card (click to see details)
    │   ├── ProgramDetails.jsx  # Full pathway view + interactive planner
    │   └── ChatWidget.jsx      # The floating chat bubble
    └── data/
        ├── pathways.js              # Transforms raw data into app-friendly format
        ├── deanza_pathways.json     # All 238 pathways (parsed from PDFs)
        ├── pathway_conflicts.json   # Precomputed conflict results
        └── department_conflicts.json # Department-level conflict summary
```

## How the App Works

### When you open the page:
1. `App.jsx` loads all pathway data from `data/pathways.js`
2. You see a search bar, filter dropdowns, the "Conflict Dashboard" link, and a grid of pathway cards
3. Everything is already loaded — no waiting for a server

### When you search or filter:
1. You type in the search bar or pick a dropdown value
2. React instantly filters the list (no server call needed — all data is local)
3. The grid updates showing only matching pathways

### When you click a pathway card:
1. `ProgramDetails.jsx` opens as a full-screen overlay
2. Shows the 2-year course map (what to take each quarter)
3. Has an **Interactive Planner** where you:
   - Pick a term (like "Fall 2025")
   - Select which courses you plan to take
   - Click "Check Conflicts"
   - The app calls the AWS `/plan` API and shows you which courses overlap

### The Chat System (two modes):

**Simple mode (chatEngine.js)** — A basic pattern-matcher that handles common questions without needing the internet:
- "Show me biology pathways" → filters the local data
- "Check conflicts for MATH 1A and CHEM 1A" → calls the /plan API

**Smart mode (chatAgent.js)** — Connects to an AI service on AWS:
- Can understand complex questions
- Remembers context from earlier in the conversation
- Queries the actual database for answers

### The Dashboard Link:
In the filter row (same row as "Degree Type" and "Department" dropdowns), there's a gold "Conflict Dashboard" button pushed to the right. Clicking it opens the full analysis dashboard in a new browser tab.

## Running Locally

```bash
cd web-app
npm install    # Download dependencies (one time)
npm run dev    # Start dev server → opens at http://localhost:5173
```

Browsing pathways works immediately. The interactive planner and AI chat need the AWS backend deployed.
