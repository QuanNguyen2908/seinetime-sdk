# seinetime Analytics Browser — Developer Guide

> SDK: `@nguyenquan241208/seinetime-sdk`  
> Purpose: Send analytics data from web apps to **seinetime Customer Insight Platform (AIP)**.

---

## Table of Contents

- [Overview](#overview)
- [Quickstart](#quickstart)
  - [Using via AIP Portal (recommended)](#using-via-aip-portal-recommended)
  - [Using as an NPM Package](#using-as-an-npm-package)
- [Browser Standalone Usage (Script Snippet)](#html-usage-script-snippet)
  - [Full Snippet Example](#full-snippet-example)
- [API Reference](#api-reference)
  - [`SeineTimeSDK.load`](#analyticsbrowserload)
  - [`identify`](#identify)
  - [`track`](#track)
  - [`delivery`](#delivery)
- [Lazy / Delayed Loading & Consent](#lazy--delayed-loading--consent)
- [Error Handling](#error-handling)
- [Examples with Frameworks](#examples-with-frameworks)
  - [React](#react)
  - [Vue](#vue)
- [TypeScript Support (for snippet users)](#typescript-support-for-snippet-users)
- [Best Practices](#best-practices)
- [Troubleshooting](#troubleshooting)

---

## Overview

**seinetime Analytics Browser** is a lightweight JavaScript SDK to instrument your website or SPA and forward events to **AIP**.  
You can integrate it in two ways:

1. **AIP Portal snippet** – easiest, managed and auto-updated script.
2. **NPM package** – import it into your build and control initialization explicitly.

---

## Quickstart

### Using via AIP Portal (recommended)

1. Sign in to **[AIP Portal](https://aip-portal.seinetime.io)** and create an **Integration** (Source).  
2. Use **Browser Standalone snippet** and paste it before `</head>` on your site.  
3. Start tracking with `window.analytics`.

---

### Using as an NPM Package

```bash
# npm
npm install @nguyenquan241208/seinetime-sdk

# yarn
yarn add @nguyenquan241208/seinetime-sdk

# pnpm
pnpm add @nguyenquan241208/seinetime-sdk
```

```ts
import { SeineTimeSDK } from '@nguyenquan241208/seinetime-sdk'

// Boot the SDK and identify the current user
const analytics = SeineTimeSDK.load(
  { secretKey: '<YOUR_SECRET_KEY>' },
  {
    attributeType: 'email',          // e.g., "email" | "phone" | "userId"
    attributeValue: 'user@site.com', // actual user identifier value
  }
)

// Send events
analytics.track('App Started')
analytics.identify('User Logged In')

// Attach to UI interactions
document.body?.addEventListener('click', () => {
  analytics.track('document body clicked!')
})
```

---

## Browser Standalone Usage (Script Snippet)

Below is a **template** showing where to put the snippet and how to use it.
Here is a **complete snippet integration** for HTML environments, including stub methods and async loading of the SDK:

```html
<!doctype html>
<html>
  <head></head>
  <body>
    <script>
      window.addEventListener('load', () => {
        window.analyticsSecretKey = 'test'
        console.log(AnalyticsNext)
      })
    </script>

    <script type="text/javascript">
      (function() {
        var globalAnalyticsKey = "analytics";
        var analytics = window[globalAnalyticsKey] = window[globalAnalyticsKey] || [];

        if (analytics.initialize) return;
        if (analytics.invoked) {
          if (window.console && console.error) {
            console.error("Segment snippet included twice.");
          }
          return;
        }

        analytics.invoked = true;
        analytics.methods = [
          "trackSubmit","trackClick","trackLink","trackForm","pageview",
          "identify","reset","group","track","ready","alias","debug","page",
          "screen","once","off","on","addSourceMiddleware",
          "addIntegrationMiddleware","setAnonymousId","addDestinationMiddleware","register"
        ];

        analytics.factory = function(e) {
          return function() {
            if (window[globalAnalyticsKey].initialized) {
              return window[globalAnalyticsKey][e].apply(window[globalAnalyticsKey], arguments);
            }
            var args = Array.prototype.slice.call(arguments);
            if (["track","screen","alias","group","page","identify"].indexOf(e) > -1) {
              var c = document.querySelector("link[rel='canonical']");
              args.push({
                __t: "bpc",
                c: c && c.getAttribute("href") || undefined,
                p: location.pathname,
                u: location.href,
                s: location.search,
                t: document.title,
                r: document.referrer
              });
            }
            args.unshift(e);
            analytics.push(args);
            return analytics;
          };
        };

        for (var i = 0; i < analytics.methods.length; i++) {
          var key = analytics.methods[i];
          analytics[key] = analytics.factory(key);
        }

        analytics.load = function(key, options) {
          var t = document.createElement("script");
          t.type = "text/javascript";
          t.async = true;
          t.setAttribute("data-global-segment-analytics-key", globalAnalyticsKey);
          t.src = "https://cdn.jsdelivr.net/gh/QuanNguyen2908/seinetime-sdk@main/standalone.js"; // Replace with actual path or CDN
          var first = document.getElementsByTagName("script")[0];
          first.parentNode.insertBefore(t, first);
          analytics._loadOptions = options;
        };

        analytics._secretKey = "YOUR_WRITE_KEY";
        analytics.SNIPPET_VERSION = "5.2.1";

        analytics.load({ secretKey: "YOUR_WRITE_KEY" },
        { 
          attributeType: "attributeType",
          attributeValue: "attributeValue"
        });
        analytics.page();
      })();
    </script>
  </body>
</html>
```

**Notes**
- This snippet queues API calls until the SDK is fully loaded.  
- Replace `"YOUR_WRITE_KEY"` with your actual **secret/write key** from AIP Portal.  
- Ensure the `https://cdn.jsdelivr.net/gh/QuanNguyen2908/seinetime-sdk@main/standalone.js` file is accessible (via CDN or hosted locally).
- `secretKey` (string, required): Your AIP Source key. Keep it safe.
- `attributeType` (string, optional): The identity attribute type (e.g., `email`, `userId`, `phone`).
- `attributeValue` (string, optional): The identity value for the current user.
- Start tracking with `window.analytics`.
---

## API Reference

### `SeineTimeSDK.load`

Bootstraps the SDK and fetches destinations. Should be called **once**.

```ts
SeineTimeSDK.load(
  { secretKey: string },
  { attributeType?: string, attributeValue?: string }
): AnalyticsBrowserInstance
```

**Parameters**
- `secretKey` (string, required): Your AIP Source key. Keep it safe.
- `attributeType` (string, optional): The identity attribute type (e.g., `email`, `userId`, `phone`).
- `attributeValue` (string, optional): The identity value for the current user.

---

### `identify`

Associate the current session with a known user.

```ts
analytics.identify(
  eventName?: string,   // optional — semantic label, e.g., 'User Logged In'
  properties?: object,  // optional — custom traits (e.g., plan, role, teamId)
  options?: object      // optional — extra options (e.g., region)
)
```

---

### `track`

Record a behavioral event.

```ts
analytics.track(
  eventName: string,    // required — e.g., 'Product Viewed'
  properties?: object,  // optional — context, e.g., { sku: 'A-123', price: 19.99 }
  options?: object      // optional — extra options
)
```

---

### `delivery`

Send a delivery/attribution style event.

```ts
analytics.delivery(
  code: string,
  secretKey: string,
  options?: object
)
```

---

## Lazy / Delayed Loading & Consent

```ts
import { SeineTimeSDK } from '@nguyenquan241208/seinetime-sdk'

export const analytics = new SeineTimeSDK()

analytics.identify('Preload Identify', { stage: 'pre-consent' })

analytics.load(
  { secretKey: '<YOUR_SECRET_KEY>' },
  { attributeType: 'email', attributeValue: 'user@site.com' }
)
```

---

## Error Handling

```ts
const analytics = new SeineTimeSDK()
analytics
  .load(
    { secretKey: 'YOUR_SECRET_KEY' },
    { attributeType: 'email', attributeValue: 'user@site.com' }
  )
  .catch((err) => {
    console.error('Analytics init failed:', err)
  })
```

---

## Examples with Frameworks

### React

```tsx
import { SeineTimeSDK } from '@nguyenquan241208/seinetime-sdk'

export const analytics = SeineTimeSDK.load(
  { secretKey: '<YOUR_SECRET_KEY>' },
  { attributeType: 'email', attributeValue: 'user@site.com' }
)

export default function App() {
  return (
    <div>
      <button onClick={() => analytics.track('hello world')}>
        Track
      </button>
    </div>
  )
}
```

### Vue

```ts
// services/analytics.ts
import { SeineTimeSDK } from '@nguyenquan241208/seinetime-sdk'

export const analytics = SeineTimeSDK.load(
  { secretKey: '<YOUR_SECRET_KEY>' },
  { attributeType: 'email', attributeValue: 'user@site.com' }
)
```

```vue
<!-- App.vue -->
<template>
  <button @click="track">Track</button>
</template>

<script setup lang="ts">
import { analytics } from './services/analytics'

function track() {
  analytics.track('Hello world', { page: 'home' })
}
</script>
```

---

## TypeScript Support (for snippet users)

```ts
import type { AnalyticsSnippet } from '@nguyenquan241208/seinetime-sdk'

declare global {
  interface Window {
    analytics: AnalyticsSnippet
  }
}
```

---

## Best Practices

- **Consistent Event Names**
- **Stable Identities**
- **Minimize PII**
- **Consent First**
- **Enrich Gracefully**

---

## Troubleshooting

- **Nothing appears in AIP** → Check key and network.
- **`window.analytics` undefined** → Ensure snippet loaded before use.
- **Duplicate events** → Ensure `.load()` is called once.

---

**© seinetime — @nguyenquan241208/seinetime-sdk**
