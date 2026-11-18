# Encoding & Obfuscation Reference (Authorized Testing Only)

The core idea: a payload passes the WAF/filter in one encoding, then gets **decoded** by a downstream component into its dangerous form. Mismatched normalization between layers is the bug.

## Common encodings
| Encoding | Example of `<` | Where it decodes |
|----------|----------------|------------------|
| URL | `%3C` | web server / framework |
| Double URL | `%253C` | when something decodes twice |
| HTML entity | `&lt;` / `&#60;` / `&#x3c;` | browser HTML parser |
| Unicode (JS) | `<` | JS engine |
| Hex (MySQL) | `0x3c` | DB |
| Base64 | `PA==` | app that base64-decodes input |

## Why it bypasses filters
- WAF inspects the **raw** request; the app **decodes** before use. If the WAF doesn't normalize the same way, the signature never matches.
- Double-encoding defeats filters that decode once then inspect.
- Mixed case + encoding + whitespace stacks the evasion.

## Reasoning checklist
1. What component ultimately *interprets* my input (browser HTML? JS? SQL? OS shell?)
2. What normalization happens between the WAF and that component?
3. Which encoding survives the WAF but is decoded by the sink?

## Defensive takeaway
- **Canonicalize/normalize input once, early**, then validate — don't inspect un-normalized input.
- Decode fully before applying any security decision.
- Don't rely on blocklist signatures; use parameterization + context-aware encoding at the sink.
