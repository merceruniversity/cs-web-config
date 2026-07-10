# IIS web.config for CommonSpot

This is the IIS configuration file used by the CommonSpot servers: both Authoring and ROPS
(Read-Only Production Servers). It should be present in the IIS root of all CommonSpot servers:

- GeorgeLucas (Development Authoring)
- HanSolo (ROD)
- LukeSkywalker (ROD)
- GeneRoddenberry (Production Authoring)
- CaptainKirk (ROP)
- MisterSpock (ROP)

**IIS Manager does not move comments with rules — reorder `web.config` manually if you edit it in
IIS Manager rather than by hand.**

## Files

- `web.config` — the IIS `rewrite`/`outboundRules` config: HTTPS enforcement, kill switches,
  CMS-internal resource handling, robots.txt routing, per-host redirects/rewrites, response
  headers, and caching rules.
- `robots-allow.txt` — served as `robots.txt` for read-only production hosts. Disallows
  CMS-internal paths and known stale dev directories; allows everything else, including uploaded
  documents served via `_cs_resources`/`cs-resources.cfm`.
- `robots-disallow.txt` — served as `robots.txt` for dev, author, and cla.mercer.edu hosts.
  Blanket-disallows the whole host.

## Notes (2026-07-10)

- Retired the ~30 fine-grained per-department `cla.mercer.edu` redirect rules (in place ~6 years,
  well past this site's ~6-month redirect SLA). Replaced with two rules: leave
  `cla.mercer.edu/faculty-staff/` alone (still actually served from that host), redirect
  everything else on `cla.mercer.edu` to the `liberalarts.mercer.edu` homepage.
  - The faculty-staff match accounts for the `www/` prefix that IIS's own file-existence
    fallback rules may already have prepended to `{URL}` by the time this rule runs — routing is
    entirely IIS's job here, not CommonSpot's.
- `cla.mercer.edu` robots.txt handling stays folded into the single shared "Rewrite robots.txt
  Disallow" rule (alongside dev/author hosts) rather than split into its own rule — robots.txt is
  one file per host, not per-path, so there's no way to disallow just `/faculty-staff/` there;
  that would require a `Disallow` line inside the file's own content instead.
- Removed unused `rewriteMap`s (`SSL1 Prefix to Domain`, `Domain to SSL1 Prefix`,
  `HTTPS Server Variable Value to s`) that weren't referenced anywhere in the rules.
- Open question, not yet acted on: `robots-allow.txt` disallows most of the same CMS-internal
  paths that `web.config`'s "Stop Processing for In-Site CMS Resources" rule treats as
  internal-only (`_cs_apps`, `_cs_upload`, `_cs_xmlpub`, `adf`, `cfdocs`, `cfide`, `commonspot`),
  but is missing `/jakarta/` — unclear what that path is/was for, needs confirming before adding.
  `_cs_resources`/`cs-resources.cfm` are intentionally *not* disallowed despite being in that same
  web.config rule — they serve real uploaded documents (PDFs, etc.) that should stay crawlable.
