# Web Security Headers

Copy-paste configurations for the HTTP security headers every site should send, with an explanation of what each one prevents and how to roll it out without breaking the site. Apache `.htaccess`, Nginx, Cloudflare and Next.js.

By [Felipe Guedes](https://fgxdev.com). These are the headers I check first in every application-security audit, and the ones most sites get wrong.

## The headers

| Header | Prevents | Value to start with |
|---|---|---|
| `Content-Security-Policy` | Cross-site scripting, injected scripts, data exfiltration | `default-src 'self'; script-src 'self'; style-src 'self'; img-src 'self' data:; font-src 'self'; connect-src 'self'; object-src 'none'; base-uri 'self'; form-action 'self'; frame-ancestors 'none'; upgrade-insecure-requests` |
| `Strict-Transport-Security` | Downgrade to HTTP, cookie theft on open networks | `max-age=31536000; includeSubDomains; preload` (only after HTTPS works everywhere) |
| `X-Content-Type-Options` | MIME sniffing turning uploads into scripts | `nosniff` |
| `X-Frame-Options` | Clickjacking on old browsers (CSP `frame-ancestors` for modern ones) | `DENY` |
| `Referrer-Policy` | Leaking URLs with tokens to third parties | `strict-origin-when-cross-origin` |
| `Permissions-Policy` | Pages or injected scripts using camera, microphone, location | `camera=(), microphone=(), geolocation=(), payment=(), usb=()` |
| `Cross-Origin-Opener-Policy` | Cross-window attacks | `same-origin` |
| `Cache-Control` | Serving stale HTML or caching private responses | `no-cache` for HTML, long `max-age` for versioned assets |

Configurations: [`apache/.htaccess`](apache/.htaccess) · [`nginx/security.conf`](nginx/security.conf) · [`cloudflare/README.md`](cloudflare/README.md) · [`nextjs/next.config.mjs`](nextjs/next.config.mjs)

## Rolling out CSP without breaking the site

1. Start with `Content-Security-Policy-Report-Only` and a `report-to` endpoint, or read the browser console on every page.
2. Remove inline scripts and styles from your templates. Move them to files. For the rare unavoidable inline script, use a nonce generated per request, never `'unsafe-inline'`.
3. Inventory every third-party origin (analytics, fonts, embeds). Each one is a trust decision; list the ones you keep in `script-src`, `style-src`, `img-src`, `connect-src`, `frame-src`.
4. Switch to enforcing. Keep the report endpoint.
5. `frame-ancestors 'none'` unless the site is meant to be embedded. Then list the exact origins.

Note: JSON-LD blocks (`<script type="application/ld+json">`) are data, not executed scripts, and are not blocked by `script-src`.

## Rolling out HSTS

- Only after every subdomain serves HTTPS. `includeSubDomains` with one HTTP-only subdomain locks users out of it.
- Start with `max-age=300`, confirm, then raise to a year. `preload` is a commitment: removal from the browser preload list takes months.

## Verifying

- Browser devtools, Network tab, response headers of the document and of an asset.
- `curl -I https://example.com/` from outside your network.
- Check the CSP with the console open on every page type; violations are logged there.

## Em português

Configurações prontas dos cabeçalhos de segurança HTTP para Apache, Nginx, Cloudflare e Next.js, com o que cada um previne e como colocar em produção sem quebrar o site. Auditoria de segurança em [fgxdev.com/pt/auditoria-de-software-e-seguranca](https://fgxdev.com/pt/auditoria-de-software-e-seguranca/).

## License

MIT.
