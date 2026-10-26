---
name: capacity-plan-report
description: Generate a team-wise sprint/iteration capacity plan report from a leave tracker (Excel or Confluence page). Use whenever the user asks for capacity planning, resource availability, leave impact, "how much can we take this sprint", or shares a leave tracker / team roster. Outputs a summary panel on top, then per-team resource tables with charts side by side.
---

# Capacity Plan Report

## Role
Act as an experienced Scrum Master / Project Planning Manager. Turn a leave tracker into a clear capacity plan that shows how much work each team can take on.

## Inputs
- An Excel file (.xlsx/.csv) or a Confluence page containing the leave tracker and team roster.
- Ask the user ONLY for what is missing and cannot be inferred:
  1. Sprint / planning period (start and end date)
  2. Public holidays (or location per resource)
  3. Hours per day (default 8) and focus factor (default 80% for commitment; always show raw capacity too)
  4. Average velocity per team (story points per sprint or per person-day) - for the commitment section
  5. Buffer % for bugs, support and meetings (default 10%)
  6. Previous sprint capacity and delivered points - for the comparison section (optional)

## Step 1 - Read and normalize the data
- Each sheet, tab, or section in the source is usually one team. Keep the team grouping exactly as given.
- Extract for every resource: Name, Team, Role, Technology (Dev / QA / other), Location (if present), Allocation % (default 100%), Leave dates and leave type.
- Treat half-day leave as 0.5 day. Treat weekends and public holidays as non-working days (not as leave).
- If a column is missing or ambiguous, state your assumption in the report footnote. Never invent names, dates, or numbers.

## Step 2 - Calculate (per resource, then per team, then overall)
- Working days = days in period minus weekends minus public holidays
- Total capacity = working days x allocation %
- Not available = approved leave days within the period (include planned and tentative separately if the tracker distinguishes them)
- Available capacity = Total capacity - Not available
- Availability % = Available / Total
- Also report in hours (days x hours per day) and apply the focus factor only if the user gave one. Show both raw and net if focus factor is used.
- Roll up by Technology (Dev vs QA) inside each team.

## Step 3 - Report layout (strict order)

### A. Top panel: Overall Summary
Placed first, above everything.
1. **Summary table** - one row per team plus a Grand Total row.
   Columns: Team | Headcount | Dev | QA | Total Capacity (days) | Not Available (days) | Available (days) | Availability % | Status (RAG)
2. **Summary chart** - one stacked bar chart, one bar per team (Available vs Not Available). Sits beside the summary table (table left, chart right) when the format allows.

### B. Team sections (repeat for EVERY team, same format)
For each team, a heading then two panels **side by side**:
- **Left: Resource table**
  Columns: Name | Role | Technology (Dev/QA) | Location | Allocation % | Total (days) | Leave (days) | Available (days) | Availability % | Leave dates
  End with a team total row.
- **Right: Chart** showing Total vs Available vs Not Available for that team. Use a pie/donut (Available vs Not Available) by default. If the team has more than one technology, use a stacked bar split by Dev / QA instead.
- Below the two panels, add a 1-2 line team insight (biggest dip, risky days, key dependency).

### C. Additional sections (included by default, after all team sections)
1. **Availability heatmap** - rows = resources (grouped by team), columns = each working day. Green = working, red = leave, amber = half day, grey = holiday. Add a bottom row with the daily team availability %.
2. **Recommended commitment** - per team: Available days x average velocity per person-day x focus factor = suggested story points. Show a range (conservative / expected / stretch). Needs velocity input; if missing, ask for it and skip the numbers until provided.
3. **Skill-wise capacity** - table and bar chart by skill/technology (e.g. Java, Spring Boot, Angular, QA manual, QA automation): Total | Not Available | Available | headcount. Use the Technology/skill column from the source; if there is no skill column, fall back to Dev / QA.
4. **Multi-sprint outlook** - capacity for the current sprint plus the next 2 sprints (use leave already in the tracker) as a line or grouped bar chart per team, so dips are visible early.
5. **Buffer allocation** - deduct a buffer from available capacity for bugs, production support, meetings and ceremonies. Default 10% buffer (ask the user to confirm). Show Available capacity -> Buffer -> Net plannable capacity.
6. **Sprint comparison** - compare with the previous sprint (capacity, availability %, commitment vs delivered if provided). If no previous data exists, show a placeholder note and ask for it.
7. **Risks and mitigations** - list from the automatic flags (see below), each with impact, affected team, and a suggested mitigation (cross-skilling, shifting work, rescheduling leave, adjusting scope).
8. **Pending leave approvals** - planned vs approved vs tentative leave, with the capacity impact if the tentative leave is confirmed.
9. **Public holiday impact by location** - holidays per location and the days they remove from each team's capacity.

### D. Footer
- Assumptions and data gaps
- Open questions for the user (missing velocity, previous sprint data, focus factor, buffer %)

## Formatting rules
- Colors: Available = green, Not Available = red or orange, Total/Holiday = grey. Keep identical across all charts.
- RAG status: Green >= 80% available, Amber 60-79%, Red < 60%.
- Label every chart with title, units (days or hours), and values/percentages on the segments.
- Keep every team's section identical in structure and column order so they can be compared.
- Output format: match the user's request. Default to an HTML page (CSS grid, two columns for table + chart) or an Excel workbook (Summary sheet first, then one sheet per team with table on left and chart on the right). In Confluence, use a table/chart macro layout with a two-column section.

## Insights to flag automatically
- Days when 2 or more people of the same technology (Dev or QA) are off together
- Days when a team falls below 60% availability
- Single point of failure: a role or skill with only one person
- Dev to QA ratio imbalance per team
- Leave clustered at the start or end of the sprint

## Handling missing data for the additional sections
- All sections in part C are part of the standard report. Do not skip a section silently.
- If the data for a section is not in the source (velocity, previous sprint, skill column, approval status, location), still include the section heading, fill what you can, and mark the rest "Data needed: <what is missing>".
- At the end, ask the user once for all missing inputs in a single short list.
- If the user asks for a shorter report, they may say "summary only" or "skip <section>"; follow that.

## Quality checklist before finishing
- Totals in the summary match the sum of all team tables
- Available + Not Available = Total for every row
- Every resource in the source appears exactly once
- Heatmap totals match the team tables
- Net plannable capacity = Available - Buffer, and the commitment range is based on it
- Every part C section is present, or marked "Data needed"
- Assumptions are stated; nothing is fabricated
