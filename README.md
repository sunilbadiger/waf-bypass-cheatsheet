# WAF / Filter Bypass Cheatsheet

A reference of **filter and WAF evasion techniques** for authorized web application security testing. When a payload is blocked, the question is usually *how* it's being filtered — these notes help reason about that during legitimate testing.

> ⚠️ **Authorized testing & education only.**
> These techniques are for penetration testers and bug bounty hunters working **within program scope and with explicit permission**, and for defenders learning how filters are evaded so they can harden them. Do **not** use these against systems you are not authorized to test. The author assumes no liability for misuse.

## Contents
- [Methodology](#methodology)
- [XSS filter bypass](payloads/xss.md)
- [SQL injection filter bypass](payloads/sqli.md)
- [Encoding & obfuscation reference](payloads/encoding.md)

## Methodology

1. **Confirm the filter, not the vuln.** Establish the injection point reflects/executes first with a benign marker (e.g. `zxcqwe`), then introduce syntax incrementally to see *which character/keyword* trips the block.
2. **Identify the layer.** Is it the WAF (generic signatures, often 403/406) or the app's own input validation (custom error)? They evade differently.
3. **Fingerprint the WAF.** Response codes, headers (`Server`, `cf-ray`), block pages. Tools: `wafw00f`.
4. **Mutate systematically** along these axes:
   - **Case** — `SeLeCt`, `ScRiPt`
   - **Encoding** — URL, double-URL, HTML entity, unicode, hex, base64 (context dependent)
   - **Whitespace** — tabs, newlines, `/**/`, `%0a`, `%0c`, `+`, comments
   - **Keyword splitting** — `SEL/**/ECT`, `java\tscript:`
   - **Alternate syntax** — logical equivalents, different tags/events, JSON vs form
   - **Redundancy** — nested tags `<scr<script>ipt>`, doubled keywords
5. **Respect scope & rate limits.** Throttle; don't hammer the WAF. Non-destructive PoCs only.

## Defensive takeaway
Blocklist filtering loses this game. The durable fixes are **context-aware output encoding**, **parameterized queries**, **positive input validation (allowlists)**, and a WAF as *defense-in-depth*, not the primary control. See each payload file's "Remediation" section.

## License
MIT
