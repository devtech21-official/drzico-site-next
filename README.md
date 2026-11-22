# Dr. Md. Isma Azam (Zico): website (Next.js)

Next.js 16 (App Router) site, exported as plain static files and hosted on Hostinger.
Live: https://neurosurgeonzico.com

## How it works

1. You push to `main` (or `next`) on GitHub.
2. GitHub Actions (`.github/workflows/deploy.yml`) runs `npm run build`:
   - `scripts/prepare.mjs` makes the WebP image sizes for new or changed photos and the finder data.
   - `next build` renders every page to HTML in `out/`.
   - `scripts/postbuild.mjs` removes the React browser runtime (the pages need none), checks every page
     (no em dash, every page has a title), writes `sitemap.xml` and writes `out/.htaccess` with the
     security headers and a strict Content Security Policy.
3. The finished `out/` folder is pushed to the `deploy` branch. Hostinger deploys that branch.

The server only ever receives HTML, CSS, JS and images. There is no Node.js, database or admin login on
the server, so there is nothing there to hack into.

## Editing

| What | Where |
|---|---|
| Doctor details, phone numbers, services, case numbers, FAQ, before/after cases, videos, patient photos | `content/site-data.json` |
| Page layout and text of each page | `app/(en)/.../page.tsx`, Bangla article in `app/(bn)/...` |
| Header, footer, menu, contact button | `components/Shell.tsx` |
| Styles | `public/assets/css/style.css` |
| Site behaviour (menu, carousels, finder, WhatsApp form, videos) | `public/assets/js/main.js` |
| Photos | `public/assets/img/...` (same folders as before). Just add the file with the right name; the build makes the WebP sizes. |
| Blog posts (later from WordPress) | `lib/blog.ts` |
| Server rules and security headers | `config/htaccess.template` |

## Run on your computer

```
npm ci
npm run build      # result in out/
npm run serve      # preview out/ (clean URLs need the .htaccess, so use Apache/Hostinger for a full test)
npm run dev        # live editing at http://localhost:3000
```

## Google indexing

The site is open to Google (no noindex header, `robots.txt` allows everything and points to the sitemap).
After a push, submit `https://neurosurgeonzico.com/sitemap.xml` in Google Search Console.
To hide the site from Google again, add this line inside `<IfModule mod_headers.c>` in
`config/htaccess.template`: `Header always set X-Robots-Tag "noindex, nofollow"`

## Security

- Strict Content Security Policy: only this site's own scripts run (inline scripts by SHA-256 hash),
  frames only from YouTube, no plugins, the site cannot be framed by other sites.
- HSTS, nosniff, Referrer-Policy, Permissions-Policy, Cross-Origin-Opener-Policy.
- Only GET/HEAD requests; hidden files, source maps and build folders return 403.
- HTTPS and non-www are forced with 301 redirects.
- No secrets in the code. The YouTube key is a browser key: keep it restricted in Google Cloud to this
  domain and the YouTube Data API v3 only.
- `npm audit`: 0 vulnerabilities at the time of the conversion.

## Rollback

The old static site is kept on the `static-v1` branch. In Hostinger, deploy that branch to go back.
