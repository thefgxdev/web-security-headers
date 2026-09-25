# Cloudflare

Two ways to add the headers at the edge, so they apply regardless of the origin.

## Transform Rules (no code)

Rules → Transform Rules → Modify Response Header → Add a rule per header, "Set static", for all incoming requests to the zone. Use the values from the main README. Keep HSTS in Cloudflare's SSL/TLS → Edge Certificates → HSTS settings instead, which also handles preload.

## Worker (when you need logic, like nonces or per-path policies)

```js
export default {
  async fetch(request, env, ctx) {
    const response = await fetch(request);
    const headers = new Headers(response.headers);
    headers.set('Content-Security-Policy', "default-src 'self'; script-src 'self'; style-src 'self'; img-src 'self' data:; font-src 'self'; connect-src 'self'; object-src 'none'; base-uri 'self'; form-action 'self'; frame-ancestors 'none'; upgrade-insecure-requests");
    headers.set('X-Content-Type-Options', 'nosniff');
    headers.set('X-Frame-Options', 'DENY');
    headers.set('Referrer-Policy', 'strict-origin-when-cross-origin');
    headers.set('Permissions-Policy', 'camera=(), microphone=(), geolocation=(), payment=(), usb=()');
    headers.set('Cross-Origin-Opener-Policy', 'same-origin');
    return new Response(response.body, { status: response.status, statusText: response.statusText, headers });
  },
};
```

## Also enable

- SSL/TLS mode **Full (strict)**, so the origin is verified too.
- **Always Use HTTPS** and **Automatic HTTPS Rewrites**.
- **Bot Fight Mode** and rate limiting rules on login and API paths.
- WAF managed rules on the free plan block the obvious.
