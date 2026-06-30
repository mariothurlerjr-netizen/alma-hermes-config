# Job search matching: CV + LinkedIn

Use this workflow when Mario asks for job matching, role recommendations, or shortlist generation from his profile.

## Inputs
- CV/resume file, usually `.docx` or `.pdf`
- LinkedIn profile URL or pasted profile text
- Optional: target geography, remote preference, industries to avoid, comp floor, language

## Output
Return, in order:
1. Candidate positioning, one line
2. Best-fit role family, ranked
3. Search keywords for LinkedIn
4. Shortlist criteria, what to prefer / avoid
5. Gaps or risks that may reduce match rate
6. Next action, what to do now

## Matching rules
- Prefer role families over generic titles, for example Sales Director, VP Sales, CRO.
- Treat LinkedIn as a filtering surface, not as a source of infinite browsing.
- When the user provides both CV and LinkedIn, reconcile them against each other. If they disagree, prefer the more recent signal but flag the mismatch.
- Use the user’s stated history and operating style to distinguish true fit from title inflation.
- Separate must-have criteria from nice-to-have criteria.

## Ranking heuristics
- Rank roles by closeness to the user’s actual operating history.
- Prefer roles with high leverage, clear ownership, and measurable outcomes.
- Penalize roles that look like culture-fit traps, unsupported general management, or vague strategy without execution ownership.
- If the profile is strong for commercial leadership, show adjacent roles as second-tier options rather than collapsing everything into one bucket.

## Pitfalls
- Do not overfit to headline keywords alone.
- Do not treat a LinkedIn URL as sufficient if the resume is available.
- Do not guess at missing chronology if the CV has it.
- Do not produce a long essay when the user asked for shortlist and ranking.

## Suggested downstream artifacts
- LinkedIn keyword list
- Job shortlist by title family
- Fit matrix by company/industry
- Rewrite suggestions for headline/about section