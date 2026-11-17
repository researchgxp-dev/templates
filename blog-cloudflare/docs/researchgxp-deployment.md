# ResearchGxP Masthead replacement

Prepared on 17 November 2026. This source replaces the public theme at
https://blogs.researchgxp.com while retaining its existing EmDash installation.
The deployment retains the existing Cloudflare build connection.

## Verified installation

- Worker: `templates` in account `ea05afda8e7e1e0c472efd02a84c6a7d`.
- Current source: `researchgxp-dev/templates`, `blog-cloudflare` directory.
- Current source commit: `57f15c7d328964110a0a5c09a44db18c32d82e1f`.
- Existing admin: Research GxP; passkey sign-in was verified in Chrome.
- Existing CMS reports EmDash 1.5.0. This copy pins EmDash and its Cloudflare
  adapter to 1.5.0, rather than using Masthead's older 0.38.0 dependency pins.
- Masthead upstream commit: `251c20d0617818678c615ebe1bd4a9a15b1139f5`.
- Hosting is Cloudflare Workers. The account currently uses Workers Free.

## Required resources

| Binding | Resource | Identifier |
| --- | --- | --- |
| `DB` | D1 `researchgxp-blog` | `95443d4e-55bd-4569-afa8-92dc00b30771` |
| `MEDIA` | R2 `researchgxp-blog-media` | Bucket name |
| `SESSION` | KV `templates-session` | `35aa4fe6490f4b9cb3abebd3f1547e11` |
| `IMAGES` | Cloudflare Images | Added by the Astro Cloudflare adapter |
| `ASSETS` | Built static assets | Added by the Astro Cloudflare adapter |

Do not reset, replace, or re-seed the production database. Existing users,
passkeys, content, site settings, and stored site secrets remain in D1. Keep the
existing session namespace and Worker secrets. The public origin remains
`https://blogs.researchgxp.com`, preserving the passkey relying-party domain.

EmDash sessions use the `SESSION` store and an opaque session cookie. There is
no session-signing secret to invent. `EMDASH_ENCRYPTION_KEY` encrypts plugin
settings; retain the existing production value if one is configured. The local
`.env` is deliberately excluded from this package and from version control.

## Validation

- `astro check`: 25 files, zero errors, warnings, or hints.
- `astro build`: completed using the Cloudflare adapter and EmDash 1.5.0.
- `wrangler deploy --dry-run`: passed without deploying.
- Upload size: 20,363.19 KiB before compression; 5,451.66 KiB compressed.
- This is about 19.89 MiB uncompressed. Cloudflare's limits page, updated
  13 November 2026, lists 64 MiB uncompressed for both Free and Paid, with no
  compressed-size cap. The original Masthead README's 3 MiB limit is outdated.
  The account remains on Workers Free; no billing upgrade is authorized.
- The compiled JavaScript and JSON contain no instance of the local encryption
  key. Generated local development secret files are excluded from this package.
- `git diff --check`: passed.

## Remaining deployment work

1. Keep the account on Workers Free, as explicitly requested by the owner.
   Verify deployment acceptance and runtime CPU use rather than relying on the
   upstream README's older bundle-size limit.
2. Publish this source into the existing `blog-cloudflare` directory in
   `researchgxp-dev/templates`. Git retains the previous theme at the recorded
   commit for rollback; no additional Cloudflare Git connection is needed.
3. In Cloudflare Workers Builds, retain the `blog-cloudflare` root directory, install with
   `pnpm install --frozen-lockfile`, build with `pnpm typecheck && pnpm build`,
   and deploy with `pnpm exec wrangler deploy`.
4. Set the build environment variable `EMDASH_SITE_URL` to
   `https://blogs.researchgxp.com`. Use Node.js 22.12 or newer. The same runtime
   variable and existing resource bindings are recorded in `wrangler.jsonc`.
5. The homepage handles the existing blog schema without querying a missing
   `placement` field. It selects lead and top stories from the latest posts
   until that optional field exists. No production schema migration or seed
   import is required to install the theme. To enable the optional newspaper
   fields later, add only missing fields in Content Types > Posts:

   | Field | Type | Label |
   | --- | --- | --- |
   | `kicker` | string | Kicker |
   | `dek` | text | Dek |
   | `dateline` | string | Dateline |
   | `featured_image_caption` | string | Image caption |
   | `featured_image_credit` | string | Image credit |
   | `correction` | text | Correction or editor's note |
   | `placement` | select | Home page placement |

   For those optional additions, set `placement` options to `lead`, `top`, and `standard`, with `standard` as
   its default. Keep fields optional and retain every existing field. The seed
   only bootstraps an empty installation; changing it does not update the live
   content schema. Do not import the fictional sample newspaper into this blog.
6. Verify the homepage, archive, an article when content exists, the media
   library, and the authenticated admin after deployment. The existing site is
   in staging; retain its search visibility until the owner requests going live.

## Rollback

Before production deployment, record the active Worker version in Cloudflare.
The observed version was `ca5ce392`. A theme-only rollback selects that prior
Worker deployment and restores the source from the recorded Git commit. Optional
fields added for Masthead can remain in the database. Do not delete any user
or content records as part of a theme rollback.
