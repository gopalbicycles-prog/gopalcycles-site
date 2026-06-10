---
title: 'How GopalDesk uses Cloudflare Workers to keep Traccar credentials secure'
description: 'Most fleet dashboards expose backend credentials in the browser. Here is how GopalDesk routes all Traccar requests through a Cloudflare Worker so keys never reach the frontend.'
pubDate: 'Jun 01 2025'
heroImage: '../../assets/blog-placeholder-1.jpg'
---

One of the first design decisions we made with GopalDesk was that Traccar credentials should never appear in browser JavaScript. If you self-host Traccar and let your frontend call it directly, your admin password is visible to anyone who opens DevTools.

The fix is a Cloudflare Worker that sits between your dashboard and Traccar. The browser calls `https://your-worker.workers.dev/api/devices` — the Worker adds the `Authorization` header from a stored secret, forwards the request to your Oracle Cloud instance, and returns the response. The browser never sees the credential.

## Why this matters for rental operators

Rental fleet dashboards are often accessed from shared computers, tablets at a front desk, or field tablets carried by staff. Any of those surfaces is a potential credential leak if you hardcode Traccar auth into the page.

The Worker approach also lets you add rate limiting, CORS rules, and audit logging in one place — without touching the dashboard frontend.

## The Worker pattern

```js
export default {
  async fetch(request, env) {
    const url = new URL(request.url);
    url.hostname = env.TRACCAR_HOST;
    url.protocol = 'https:';

    const proxied = new Request(url.toString(), request);
    proxied.headers.set(
      'Authorization',
      'Basic ' + btoa(`${env.TRACCAR_USER}:${env.TRACCAR_PASS}`)
    );
    return fetch(proxied);
  }
};
```

`TRACCAR_USER` and `TRACCAR_PASS` live in Cloudflare's encrypted secret store — not in your code or your repo.

## Manual refresh as a UX decision

GopalDesk defaults to manual map refresh rather than polling. For field operators on limited mobile data plans, continuous polling adds up. A refresh-on-demand pattern keeps data consumption predictable and the dashboard fast on slower connections.
