# Security Property Extraction

## Goal

Given a **CVE id**, research it and write a **security property**: a
description that makes the real security risk clear, and lets a reviewer judge whether *any*
implementation is vulnerable — by **whether a vulnerability condition holds**, not by whether the
code matches a specific "golden" secure/vulnerable implementation.

## Write only what you found — do not over-reason

Write **only facts you clearly found**, plus what is **obviously inferable** from them. Do not
guess, over-reason, or pattern-match from "similar" CVEs/CWEs. **If you didn't find something,
leave it `null`** — an honest gap beats a confident invention. Note real gaps in `unresolved`.

Useful sources: NVD (including its JSON API), the **fix commit/PR diff** (the plain-text
`.diff`/`.patch` shows what actually changed), the CWE page, and vendor / oss-security advisories.
Prefer plain-text sources over JavaScript-rendered pages.

## Output (JSON)

```jsonc
{
  "cve": "...",

  // factual context looked up from the sources (grounding rule applies — null if not found)
  "metadata": {
    "cwe": ["CWE-NNN", "..."],   // only the id(s) NVD/MITRE assign — no names or commentary; put any doubts about the label in `unresolved`. null if none/noinfo

    "product": "affected product / project",
    "cvss": "base score + vector",
    "references": ["..."]
  },

  "vuln_class": "one line: the concrete weakness in this CVE",

  // plain language, no code: who the attacker is, what they can do, what they gain
  "risk_narrative": "...",

  // the condition that must always hold for safety, stated behaviorally and independently of
  // any implementation
  "invariant": "...",

  "judging_criteria": {
    // PRIMARY basis for judging any implementation; closely tied to `risk_narrative`. List the
    // conditions (e.g. an attack vector) such that, once any one is satisfied, the exact, clear
    // risk described above actually happens. Each is independently sufficient — if ANY one holds,
    // the implementation is vulnerable. Include cosmetic / "false fixes" (bypassable or misplaced
    // mitigations).
    "vulnerable_if": ["...", "..."],

    // the negation: none of the vulnerable_if conditions hold and the invariant holds. Open-ended
    // — the technique is not prescribed, and the behaviors that satisfy it are many, so never
    // present a list as complete: either append "etc." if you give examples, or just state it as
    // the negation of vulnerable_if.
    "secure_if": "..."
  },

  // differences that vary between implementations but DON'T affect the property
  // (so a reviewer doesn't penalize "not matching golden")
  "security_irrelevant_differences": ["..."],

  // what you couldn't determine, or that doesn't apply, each with a one-line reason. [] if none.
  "unresolved": ["..."]
}
```

Fill what the evidence supports; leave the rest `null`. Keep it concise.

## Before returning, check

- **Effect, not mechanism.** Capture the effect that makes the difference between secure and vulnerable — not how some implementation achieves it. E.g. "a filename can never be treated as a command-line option", not "the code prepends `--`".
- **Grounded.** Every claim traces to something you found; gaps are `null` and noted in `unresolved`, never invented.
