Updates via https://webhostpro.com/ and https://webhostproblog.com

Goal
Maintain and improve a renderer that replaces Webalizer’s per-domain `index.html` with a modern static UI inside cPanel, while continuing to use Webalizer-generated monthly `usage_YYYYMM.html` files as the data source.

Environment and paths

* Server: cPanel + WHM, CloudLinux 9, LiteSpeed. About 150 sites, low traffic.
* Webalizer data exists in multiple layouts, including SSL:

  * `/home/*/tmp/webalizer/index.html`
  * `/home/*/tmp/webalizer/*/index.html`
  * `/home/*/tmp/webalizer/ssl/*/index.html` (primary source we want)
* The renderer runs as root and rewrites `index.html` in place, preserving ownership and mode.
* The renderer should always rebuild the modern UI when a `usage_YYYYMM.html` exists, even if the original Webalizer index backup is missing.

Current UI requirements

* Flat dark UI (no gradients).
* Cards at top show monthly totals. Only show the month label once (no repeated “· Month” in card labels).
* Sections:

  1. Top Referrers (Top 25) with Hits
  2. Top Pages (Top 25) with Hits and Visits columns aligned and right-justified
  3. Monthly Reports list (3 columns on desktop)

Data extraction requirements

* Parse the latest monthly file (`usage_YYYYMM.html`) and extract:

  * Top Pages: only real pages (php, html, htm, plus directory-style routes) and exclude assets (js, css, images, fonts, etc.).
  * Top Pages must be sorted descending by Hits.
  * Top Pages must include both Hits and Visits if available in the table.
  * Top Referrers must select the correct table/column and must never treat percent-only or numeric-only labels as referrers. It should show referrer domains/URLs with Hits.
* Webalizer HTML tables vary. Prefer robust content-based detection and column scoring. Do not rely on a single fixed anchor or heading.

Operational requirements

* Do not modify Webalizer itself. We only replace what users see by rewriting `index.html`.
* Keep changes safe:

  * Atomic writes (write temp, chown, chmod, replace)
  * Preserve existing file ownership in each domain folder
  * Do not crash the stats pipeline, fail gracefully per-folder
* Keep the original Webalizer index available if present:

  * Store once as `index.webalizer.bak.html` when a real Webalizer index is encountered.

What to deliver

* Update the uploaded `whp-webalizer-ui-render` file only.
* Make the scanning include SSL folders and ensure the renderer updates at least the SSL path example: `/home/prohost/tmp/webalizer/ssl/webhostpro.com`.
* Ensure Top Referrers is correct (not percent tables) and Top Pages is correct (filtered, sorted, shows Hits + Visits).
* Provide a quick internal sanity check path or function that makes it easy to validate extraction against a single folder.

Constraints

* I edit files via GUI. Any instructions should be “paste this into the file” style, not shell wrapper commands.

