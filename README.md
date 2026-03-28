# Java Mastery — Interactive Learning Platform

A free, interactive Java learning platform built as a static site.  
No server, no cost, no install. Runs entirely in the browser.

## Features

- 📖 **Rendered lessons** with syntax-highlighted Java code and one-click copy
- ✅ **Progress tracking** — check off sections as you complete them (saved in browser)
- 🧩 **Interactive exercises** — reveal answers on demand, mark as solved
- 🃏 **Flashcard mode** — flip-to-reveal interview Q&A with scoring
- 🔍 **Search** — instant search across all weeks simultaneously

---

## Setup (GitHub Pages — Free Public URL)

### Step 1 — Create a GitHub account (if you don't have one)
Go to https://github.com and sign up for free.

### Step 2 — Create a new repository
1. Click the **+** icon → **New repository**
2. Name it `java-mastery` (or anything you like)
3. Set it to **Public**
4. Click **Create repository**

### Step 3 — Upload the files
Upload the following using GitHub's web UI (drag and drop or "Add file → Upload files"):

```
java-mastery/
  index.html          ← the entire app
  content.js          ← week list (edit this to add new weeks)
  content/            ← folder containing all your .md files
    week1_java_mastery.md
    week2_java_mastery.md
    Week_3_OOP_Pillars_III_IV_Polymorphism_Abstraction_Generics.md
    Week_4_Essential_APIs_Collections_Framework.md
    Week_5_Streams_Functional_Programming.md
    week6_concurrent_collections_safety.md
    week7_architecture_io_modules.md
    week8_concurrency_revolution.md
    java_revision_guide.md
    capstone_projects.md
```

> **Important:** The `content/` folder must be created first. On GitHub, you can create it by uploading a file with the path `content/week1_java_mastery.md`.

### Step 4 — Enable GitHub Pages
1. Go to your repository → **Settings**
2. Scroll to **Pages** in the left sidebar
3. Under **Source**, select **Deploy from a branch**
4. Branch: **main**, folder: **/ (root)**
5. Click **Save**

### Step 5 — Your site is live!
Wait ~60 seconds, then visit:
```
https://YOUR-USERNAME.github.io/java-mastery/
```

---

## Adding a New Week

1. Drop the new `.md` file into the `content/` folder on GitHub
2. Open `content.js` and add one entry to the `WEEKS` array:

```js
{
  id: 'week9',
  num: '09',
  title: 'Your Week Title',
  file: 'your_new_file.md',
  special: false,
},
```

That's it — the site will automatically pick it up.

---

## How Content is Auto-Detected

The app scans your markdown for specific section headings:

| Heading in your .md | Effect in the app |
|---|---|
| `## Practice Exercises` | Powers the **Exercises** tab |
| `## Interview Questions` | Powers the **Flashcards** tab |
| Any other `## Heading` | Becomes a checkable lesson section |

Exercise format (within the exercises section):
```markdown
### Exercise 1: Your Title
Question text here...

**Answer:**
Answer and code here...
```

Flashcard format (within the interview questions section):
```markdown
**Q1: Your question here?**

**Answer:**
Your answer here...
```

---

## Local Preview (without GitHub)

Just open `index.html` in a browser — but note that fetching local files requires a simple server due to browser security. Use:

```bash
# Python (built-in, no install needed)
python3 -m http.server 8080
# Then open: http://localhost:8080
```

---

## Tech Stack

| Library | Version | Purpose |
|---|---|---|
| marked.js | latest | Markdown → HTML rendering |
| Prism.js | 1.29 | Java syntax highlighting |
| Vanilla JS | — | Everything else |
| Google Fonts | — | Sora + JetBrains Mono |

Zero build step. Zero npm. Zero config.
