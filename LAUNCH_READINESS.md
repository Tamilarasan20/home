# Launch Readiness — Working Items

Work items required to take the Loraloop marketing site to a production-ready
launch state. Each item is grounded in a specific gap found in the current
codebase; file references point at the evidence.

Status legend: `[ ]` not started · `[~]` in progress · `[x]` done

---

## 1. SEO & discoverability

| # | Work item | Use case | How it helps |
|---|-----------|----------|--------------|
| 1 | Per-route SEO metadata (title, description, canonical) | Only `AgentLandingPage.tsx` sets a document title, covering the 4 agent routes. Home, Pricing, Solution, Blog index, Tools, About, Contact, the 5 persona pages and the legal pages all inherit the single static title in `index.html` | Search engines currently see ~17 routes as near-duplicates. Unique titles and descriptions make each page independently rankable and lift click-through from results pages |
| 2 | Extend `prerender.mjs` beyond blog posts | The script deliberately stops after blog generation, so every marketing route is served as an empty `<div id="root">` | Crawlers and social scrapers that do not execute JavaScript see real content, making the pages that actually sell the product indexable — not just the blog |
| 3 | Add `public/robots.txt` | No robots file exists | Directs crawlers to the indexable surface and advertises the sitemap, instead of leaving discovery to chance |
| 4 | Generate `sitemap.xml` at build time | No sitemap exists; `prerender.mjs` already enumerates every blog slug, so the data is in hand | Gets the full blog archive plus marketing routes discovered in days rather than weeks |
| 5 | Ship the missing `og-image.png` | `prerender.mjs` points every prerendered page at `/og-image.png`, but `public/` contains only `blog-thumbnails/` | Every share on LinkedIn, X or Slack currently renders a broken preview image. One asset fixes social preview across all pages |
| 6 | Organization + WebSite JSON-LD on the homepage | Structured data today is Article, FAQ and Breadcrumb only, emitted for blog posts | Enables a brand knowledge panel and the sitelinks search box for branded queries |

## 2. Performance (Core Web Vitals)

| # | Work item | Use case | How it helps |
|---|-----------|----------|--------------|
| 7 | Optimise the hero image | `src/assets/video_img.png` is a 408 KB PNG imported by both `imports/LandingPage/LandingPage.tsx` and `imports/Home-1/Home-10-5453.tsx` | It is the LCP element on the landing page. WebP/AVIF encoding, explicit width/height and a preload hint materially cut mobile LCP and eliminate layout shift |
| 8 | Trim font loading | `index.html` loads 5 families across two CDNs (Fontshare and Google Fonts) | Fewer render-blocking round trips and less flash of unstyled text; families left over from the Satoshi / General Sans migration can be dropped |

## 3. Analytics & conversion measurement

| # | Work item | Use case | How it helps |
|---|-----------|----------|--------------|
| 9 | Fire named conversion events | `trackEvent()` is exported from `src/lib/mixpanel.ts` but has zero call sites; only Mixpanel autocapture is active | Without explicit events on waitlist submit, pricing plan click and billing-term toggle, funnel drop-off is unmeasurable — pricing changes ship without feedback |
| 10 | Move configuration to env vars and add `.env.example` | The Mixpanel token is hardcoded as a source fallback in `src/lib/mixpanel.ts`, and no template documents `VITE_SUPABASE_URL`, `VITE_SUPABASE_ANON_KEY`, `VITE_APP_URL` or `GEMINI_API_KEY` | Contributors and new environments stop guessing required configuration, and staging traffic stops contaminating production analytics |

## 4. Lead capture

| # | Work item | Use case | How it helps |
|---|-----------|----------|--------------|
| 11 | Real contact form on `/contact` | `pages/Contact.tsx` offers only a `mailto:` link | Captures enquiries directly into Supabase rather than losing them to a mail client many visitors never open |
| 12 | Consolidate the application URL | Three sources disagree: `components/WaitlistForm.tsx` hardcodes `app.loraloop.com/signup`, `lib/supabase.ts` defaults `APP_URL` to `loraloop-main-api.vercel.app`, and `vercel.json` rewrites `/app` to a third target | A single env-driven constant prevents signup CTAs silently pointing at the wrong environment |

## 5. Quality, accessibility & hygiene

| # | Work item | Use case | How it helps |
|---|-----------|----------|--------------|
| 13 | Decide the fate of i18n | Six locale files are bundled, only `components/Header.tsx` calls `useTranslation`, and the language picker was removed in `bd79ebf` | Finish the translation coverage or remove the machinery. As it stands the bundle carries unused weight and implies multi-language support the site cannot deliver |
| 14 | Add missing image alt text | 5 of the 20 `<img>` tags under `src/app` have no `alt` attribute | Accessibility compliance, plus image-search traffic |
| 15 | Add `typecheck` and `lint` scripts | `package.json` defines only `dev` and `build` | Type errors surface locally instead of during a Vercel deploy, shortening the fix loop |
| 16 | Add a CI workflow | There is no `.github/` directory | Every pull request gets a build check; today a broken build can merge to `main` unnoticed |
| 17 | Delete the root `.cjs` codemod scripts | Twelve one-off migration scripts sit at the repository root (`update_*.cjs`, `fix_mobile_pricing.cjs`, `cleanup_script.cjs`), all already applied | Removes dead code that reads like maintained tooling |
| 18 | Delete stray root images | `video_img.png` and `app_logo.png` at the root are byte-identical duplicates of the `src/assets/` copies that are actually imported; `thank_you.png` (194 KB) is imported nowhere | Reclaims roughly 620 KB and removes ambiguity about which copy is live |

---

## Suggested sequence

1. **Items 5, 3, 4, 1** — broken social previews and absent crawler entry points
   are actively costing traffic every day they remain.
2. **Item 9** — instrument the funnel before making further pricing decisions,
   so their impact is measurable.
3. **Item 7** — the largest single performance win available.
4. **Items 15, 16** — put the guardrails in place before the remaining changes land.
5. Everything else, in any order.
