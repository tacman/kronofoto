# Fortepan / OPAN Platform Notes

*Personal reference notes, not part of the kronofoto codebase — kept in Tac's fork on this branch, not merged upstream. Recommendations tracked separately.*

*As of 2026-07-03.*

## Links

| Site | Live site | Repo |
|---|---|---|
| fortepan.us | https://fortepan.us | https://github.com/fortepan-us/kronofoto (canonical) · https://github.com/tacman/kronofoto (Tac's fork) |
| fortepan.hu | https://fortepan.hu | https://github.com/fortepan/fortepan.hu (frontend only; backend closed) |
| fortepan.eu | https://fortepan.eu | not published anywhere |
| kora (MSU Matrix) | not confirmed | https://github.com/matrix-msu/kora |

## fortepan.us — kronofoto (code we have, full stack)

- **Repo**: `fortepan-us/kronofoto` (working copy: `tacman/kronofoto` fork)
- **Backend**: Python / **Django** (>=4.2), SQLite in dev
- **Frontend**: Django templates + webpack/rollup/gulp-built JS, no SPA framework
- **Hosting today**: University of Iowa. Migration target under discussion: MSU Matrix Center — but Matrix has said it only wants to host **PHP** projects, which directly conflicts with kronofoto's Python/Django stack.
- **Notable subsystems**:
  - **On-demand signed image resizer** (`imageutil.py`): `ImageSigner`/`ImageCacher` classes generate a cryptographically **signed resize URL** (`django.core.signing.Signer`) encoding path + target width/height. Requests hit a `kronofoto:resize-image` view, which uses **Pillow** (`FixedResizer`/`FixedWidthResizer`/`FixedHeightResizer`) to resize + re-encode as JPEG (q=60), then caches the result to storage under `images/{block1}/{block2}/{sig}.jpg`. This is the "invisible" image pipeline — not obvious just browsing the site.
  - Feature modules (views/models): MyList, Embed (shipped), Exhibit, Krono360/geolocation (in development — this is the deliverable tied to the $32k reinstated NEH "Foto360º" grant), ActivityPub federation, tag search, vector tiles/map, photosphere.
  - **Architectural note (Tac's stated concern)**: frontend currently has direct DB access via Django views/ORM — no API boundary. Tac's proposal (not yet agreed) is to decouple this into a JSON API + JS widget layer.

## fortepan.hu — frontend only (code we have); backend closed (no code)

- **Repo**: `fortepan/fortepan.hu` — explicitly a "public dev repository," frontend only
- **Frontend stack**: **Eleventy** (static site generator) + **Liquid** templates, **Sass/PostCSS**, **Webpack**, **Stimulus** (Basecamp's HTML-first JS framework) + vanilla JS. Content managed via **Forestry/Tina CMS**. Hosted on **Netlify**.
- **Backend**: **Drupal + ElasticSearch** — closed/proprietary, no admin or source access. Team reportedly dislikes their own backend but won't share it (territorial-access blocker, same team as fortepan.eu).
- Considered the **gold standard for UI/UX** among the three sites, despite the backend being a black box to us.
- No visible custom image-resize pipeline in the frontend repo — thumbnail JS handles responsive re-layout only (`resizeThumbnails()`), not actual server-side image transformation; real image serving is inside the closed Drupal backend.

## fortepan.eu — no code available at all

- **Frontend**: Stimulus
- **Backend**: **CakePHP** (both admin and frontend API)
- **Database**: MariaDB (photo metadata) — a MySQL fork, functionally equivalent for planning purposes
- **Storage**: AWS S3
- **Image processing**: AWS **Serverless Image Handler** — a managed Lambda-based on-the-fly resizing service (this is fortepan.eu's counterpart to kronofoto's in-app Pillow resizer above; fortepan.eu offloads it to AWS rather than doing it in application code)
- **CDN**: Amazon CloudFront
- **Status**: Source not published anywhere (no GitHub/GitLab/Bitbucket found). Bettina plans to ask the team to publish it. This is the **top blocker** for OPAN Options 1 and 3 (both depend on this team handing over code/access) — a bigger risk than any framework debate.
- Same team as fortepan.hu; Tac describes both as "territorial" about code/backend access.
- Platform ambition beyond the 3 sites: OPAN has been experimenting with a **Swiss version** of this platform, and is working to integrate fortepan.hu's map view (currently .hu-exclusive) into fortepan.eu — the seed of a shared "super platform" for all OPAN members (see Option 1 below).

## kora — MSU Matrix's own platform (code we have; reference option, not a Fortepan site)

- **Repo**: `matrix-msu/kora`
- **Stack**: **PHP 8.1 / Laravel 10** (`laravel/laravel`), Laravel Socialite, Laravel UI, Flysystem, Doctrine DBAL, geocoding (Nominatim provider)
- **Purpose**: MSU Matrix's flagship archive/metadata-schema-first platform — not narrative/timeline-first like Fortepan. Not currently on Bettina's shortlist, but relevant because adopting it would remove the "foreign codebase" risk for Matrix (they already build/maintain Laravel apps), unlike CakePHP.

## Quick-reference table

| Site | Frontend | Backend | Database | Storage/Image/CDN | Code access | Hosting |
|---|---|---|---|---|---|---|
| fortepan.us | Django templates + JS build chain | Python/Django (kronofoto) | SQLite (dev) | in-app Pillow resizer, signed URLs | full (fork) | Univ. of Iowa → maybe Matrix (PHP-only conflict) |
| fortepan.hu | Eleventy/Liquid/Stimulus | Drupal + ElasticSearch | unknown (closed) | unknown (closed) | frontend only | Netlify (frontend) |
| fortepan.eu | Stimulus | CakePHP | MariaDB | AWS S3 + Serverless Image Handler + CloudFront | none | unknown (AWS-based) |
| kora (MSU Matrix, not a Fortepan site) | Laravel/Blade (assumed) | PHP 8.1/Laravel 10 | unknown | unknown | full | Matrix-hosted |

**Open blocker to track**: fortepan.eu/hu team granting real code + backend access — this gates Options 1 and 3 of the OPAN platform decision regardless of which framework is preferred.

## The three OPAN platform options (Bettina's proposal, 2026-07-03)

**Option 1 — Fortepan.eu / Matrix.** Move Fortepan US assets and metadata to Matrix, but power the project through Fortepan.eu's PHP (CakePHP) code. Shelve kronofoto entirely, giving up its 2 advanced storytelling apps (Exhibit, Krono360) — "at least for now," with no stated path to bring them back. Rationale: fortepan.eu has "the right interface to jump off of." Open risk (flagged by Bettina herself): unknown whether Matrix's IT team can support CakePHP. Upside: with more funding, this could grow into a shared super-platform for all OPAN members — building on the existing Swiss-version experiment and the in-progress hu→eu map-view integration.

**Option 2 — Kronofoto / OPAN-controlled server / assets @ Matrix.** Move kronofoto's code to a server OPAN controls (not Matrix); keep only photographic assets + metadata at Matrix. Gradually layer a Fortepan-style front end on top of kronofoto, evolving capabilities and interface over time (possibly the backend too, eventually). Makes the code available to all existing and new OPAN members. Felipe Bengoa would help facilitate. This is the only option with zero territorial-access dependency today.

**Option 3 — Fortepan.eu / OPAN-controlled server / assets @ Matrix.** Build a version of fortepan.eu on an OPAN-controlled server (not Matrix), with help from the current fortepan.eu developer and Felipe Bengoa. Relocate only assets + metadata to Matrix. Unlike Option 1, explicitly plans to add kronofoto's advanced storytelling apps back in "eventually."

**Cross-cutting risk**: Options 1 and 3 both assume the fortepan.eu/hu team will actually hand over source code and/or grant access — unconfirmed as of this writing. Option 1 additionally assumes Matrix's IT team can operate a CakePHP stack (MariaDB, S3, Lambda-based image handler, CloudFront) — also unconfirmed.

## A note on framework choice: why Symfony is worth a look

Setting kronofoto (Python) aside, OPAN's properties already run **three different PHP frameworks**: Drupal (fortepan.hu's backend), Laravel (kora, MSU Matrix's own platform), and CakePHP (fortepan.eu). Of these three, **CakePHP is the odd one out** — it shares no code with either of the others. Drupal and Laravel, by contrast, both have real, direct ties to **Symfony**:

- **Drupal** has been built on top of Symfony components since **Drupal 8 (2015)** — routing, dependency injection, and the HTTP kernel are literally Symfony code inside Drupal core, and this continues through Drupal 9, 10, and 11 (Drupal 11 currently runs on Symfony 7).
- **Laravel** (kora's framework — i.e. what Matrix's own team already builds and maintains) composes more than a dozen Symfony components under the hood: Console, Routing, HttpFoundation, HttpKernel, Mailer, Translation, EventDispatcher, and others. It's its own framework with its own conventions, but Symfony is woven through it.
- **CakePHP** shares no components with either — it's architecturally its own island.

So a Symfony-based rebuild would have genuine technical kinship with fortepan.hu's backend and partial kinship with kora/Matrix's stack, in a way that adopting or extending CakePHP simply doesn't.

Symfony is also a larger, more findable talent pool than CakePHP — worth weighing given all of OPAN's teams are small. Per JetBrains' 2025 State of PHP survey: Laravel 64%, Symfony 23%, CakePHP under 10% (grouped with CodeIgniter/Yii as "smaller but stable"). Symfony also ships **LTS releases every 2 years**, each with 3 years of bug-fix support and 4 years of security support — a support cadence suited to a multi-decade institutional archive.

**The team-size reality check**: every team here — OPAN, fortepan.us, fortepan.eu/hu — is small. That rules out a big-bang rewrite onto an unfamiliar framework as a Phase 1 move, regardless of its long-term merits. That's the case for treating this as a two-phase effort rather than a single decision:

- **Phase 1**: Rebuild fortepan.us itself as a Symfony project. Scoped to one site, one small team, no dependency on the fortepan.eu/hu access blocker being resolved first.
- **Phase 2**: Once Phase 1 proves the pattern, extend it so **each site's best feature becomes available everywhere** — fortepan.hu's map view, kronofoto's storytelling tools (Exhibit, Krono360), fortepan.eu's multi-tenant admin — shared across all OPAN properties instead of siloed per site. This is the "super platform" Bettina's Option 1 gestures at, but reached by growing fortepan.us outward rather than by adopting fortepan.eu's CakePHP code wholesale.

*(This framing is Tac's proposed alternative angle, not yet raised with Bettina or the fortepan.eu/hu teams — presented here for discussion, not as a decided plan.)*

**Open item, needs further discussion**: this whole comparison has focused on the photo-archive core (photo admin, albums, timeline). None of the three options above have addressed the **CMS** — the rest of each site's pages that aren't photo/album/timeline features (about pages, static/editorial content, etc.). Worth flagging now so it doesn't get lost, but not yet scoped.

## Update: access agreement + a decentralized alternative to "one unified backend"

**Access**: Bettina has verbal agreement that the fortepan.eu/hu team will share their backend code. It's still closed source and may not go fully public, but as director she can get us visibility into it. This is a real step forward on the access blocker above — though still verbal, not yet exercised, so treat it as "in progress" rather than "resolved."

**A lighter-weight alternative to a single shared backend**: rather than migrating everyone onto one platform, each team (fortepan.us, .hu, .eu) could keep its own repo/fork, and the shared goal becomes **publishing packages that are easily integrated** across those independently-owned codebases — not one shared deployment. Concretely: the timeline and PhotoStory tools could be written as **PHP libraries** that any site's PHP backend installs, requiring only a defined API endpoint to integrate against. No shared database, no shared deploy, no forced migration to a common framework — the API contract is the integration point, and each site keeps full ownership of its own hosting and backend choice. This pairs naturally with the JS-widget idea above (JS widgets for frontend-only integration on any site, PHP libraries for tighter backend integration on PHP sites) and with the earlier API-boundary principle.
