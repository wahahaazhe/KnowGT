---
name: knowgt
description: Generate concise, human-readable GitHub Trending briefings from WebFetch/WebSearch results, compare against prior markdown reports when available, and write daily or weekly trend summaries. Use when the user asks to analyze GitHub Trending, compare today's GitHub trends with history, explain trending repositories in plain language, or create a repeatable GitHub Trending report for Codex, Claude Code, or another coding agent.
---

# GitHub Trending Briefing

Create a plain-language briefing from GitHub Trending. Prefer WebFetch/WebSearch over local scraping scripts. The value of this skill is trend interpretation, not crawler maintenance.

## Workflow

1. Determine scope:
   - Default period: `daily`.
   - Also support `weekly` and `monthly` when requested.
   - Default output: a dated markdown briefing.

2. Fetch current data:
   - Fetch `https://github.com/trending?since=daily`, `weekly`, or `monthly`.
   - Capture repository name, URL, language, total stars, stars gained in the period, and short description.
   - If GitHub Trending is unavailable, use WebSearch to find a reliable mirror or GitHub search fallback, and state the fallback used.

3. Find history:
   - If the current project has `trending_reports/`, read the newest 2-5 markdown reports.
   - If the user gave another history directory, use that.
   - If no history exists, create a first-run briefing and say there is no comparison baseline.

4. Compare:
   - Identify new entries that did not appear in recent reports.
   - Find recent best performers: consistently hot projects + high-growth hidden gems.
   - Group the trend by theme, such as AI coding agents, developer tools, infra, UI, data, security, or education.

5. Write the briefing:
   - **Structure rule**: Output exactly 4 sections (New On The List, Recent Best, Trend Read, Full List). NEVER split by growth rate into multiple tiers (e.g., "Rising Stars / Steady Growth / Stable").
   - Use direct language. Avoid jargon such as architecture, paradigm, empowerment, enablement, leverage, and strategic moat unless the repository itself uses those words and they are necessary.
   - **Column name must be "Plain-English Use"**, not "Description", "Summary", or "Features".
   - **Explain from user perspective**: Every "Plain-English Use" cell must include user-centric phrases like "helps you...", "lets you...", "saves you from...". Explain what the project **helps the user do**, not its internal implementation or marketing tagline.
   - **Key principle: a non-technical person should understand it.** If someone outside tech can't grasp what the project does after reading your explanation, it's not good enough.
   - Don't copy-paste similar vague descriptions across projects (e.g., "AI-native", "modern architecture"). Each project's explanation must capture what makes it unique.
   - Include a simple analogy only when explaining an unfamiliar project.
   - For "who needs it", name concrete people and situations, not broad labels like developer or engineer.

6. Save markdown output when working in a repo:
   - **Must generate TWO reports**:
     - **Briefing** → `trending_reports/trending_briefing_YYYY-MM-DD.md` (quick overview, exactly 4 sections)
     - **Detailed** → `trending_reports/trending_detailed_YYYY-MM-DD.md` (deep-dive using "Repository Explanations (Detailed)" template)
   - In the detailed report, every first-time project gets a full breakdown. Projects already covered in prior reports get a short citation.
   - Do not overwrite an existing report unless the user asks.

7. [Optional] Generate HTML report:
   - **HTML is optional**, not required by default. Only generate when user explicitly asks for "HTML" or "visualization".
   - If needed, read the template at `skills/productivity/github-trending-briefing/reference/briefing-template.html`:
   - Read the template at `skills/productivity/github-trending-briefing/reference/briefing-template.html` to understand its CSS and DOM structure. The template is a **single-page, two-tab design** that merges both briefing and detailed views.
   - Generate `trending_reports/trending_briefing_YYYY-MM-DD.html` using the **exact same CSS and DOM structure** as the template. Fill every section with real data:
     - **Hero**: update title, date, since-period, source, baseline chips.
     - **Stats bar**: replace sample numbers with actual counts (total repos, new entries, recent best).
     - **Briefing tab** ("New On The List" table, "Recent Best" table, "Trend Read" paragraphs, "Full List" table). Remove empty-row placeholders once real rows exist.
     - **Detailed tab** (`.detail-list`): for each **first-time** project, render a full `.project-card` with header (rank, name+link, lang-tag, stars) and body (what-is-it, analogy in `.proj-analogy`, helps-list in `.proj-helps`, who-needs-it with `.persona` tags). For projects already covered in prior reports, render a shorter `.project-card` with lower opacity and a note referencing the earlier report instead of repeating the full breakdown.
     - **Tab-switching `<script>`**: keep the exact `switchTab()` function from the template so tabs work.
   - The HTML file must be self-contained (no external resources). All CSS stays inline in `<style>`, all JS inline in `<script>`.
   - After writing the HTML file, open it in the default browser:
     - **macOS**: run `open trending_reports/trending_briefing_YYYY-MM-DD.html`
     - **Linux**: run `xdg-open trending_reports/trending_briefing_YYYY-MM-DD.html`
     - **Windows**: run `start trending_reports/trending_briefing_YYYY-MM-DD.html` (cmd) or `Start-Process trending_reports/trending_briefing_YYYY-MM-DD.html` (PowerShell)
   - Detect the platform automatically — do not ask the user which OS they are on.

## Briefing Shape

Use this structure unless the user asks for something else:

```markdown
# GitHub Trending Briefing - YYYY-MM-DD

> Generated: YYYY-MM-DD
> Source: github.com/trending
> Baseline: <recent report dates or "none">

## New On The List

| Rank | Project | Language | Stars Gained | Plain-English Use |
|------|---------|----------|--------------|-------------------|

## Recent Best

| Project | Evidence | Why It Matters |
|---------|----------|----------------|

## Trend Read

Short paragraphs explaining the strongest pattern in normal language.

## Full List

| Rank | Project | Language | Total Stars | Stars Gained |
|------|---------|----------|-------------|--------------|
```

## Repository Explanations (Detailed)

Use only in detailed reports. Keep "Plain-English Use" column in briefings concise.

```markdown
Analyze this GitHub Trending project. Requirements:

- No jargon like "architecture", "paradigm", "empowerment", "leverage"
- Every explanation needs a everyday analogy (explain like I'm your grandma)
- Focus on "what it helps you do", not technical internals
- Name concrete people and situations, not "developers" or "engineers"

Output format:
What is it? (Under 30 words, plain language)
Analogy (One everyday object or scenario)
What can it help you do? (3 items, each starts with a verb)

Help you...
Let you...
Save you from...

Who actually needs it? (Be specific, not "developers")

Example: A marketing specialist who makes daily PPTs for the boss
Example: A new hire inheriting messy legacy code
```

## Output Self-Check

After writing, check each item. Rewrite if anything fails:
- [ ] Are there exactly 4 sections (New On The List, Recent Best, Trend Read, Full List)?
- [ ] Is the 5th column of "New On The List" table named "Plain-English Use"?
- [ ] Does every "Plain-English Use" cell contain user-centric phrases like "helps you / lets you / saves you from"?
- [ ] Was the list split into multiple tiers by growth rate (e.g., "Rising Stars / Steady Growth / Stable")? If yes, merge back into a single table.
- [ ] Are there any jargon words like "architecture / empowerment / paradigm / leverage"?

