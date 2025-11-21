---
applyTo: "**"
description: "Next.js 16 development guidelines and best practices"
name: "nextjs-guidelines"
---

# Next.js 16 Development Guidelines

## Version Requirements

- **Next.js**: 16.0.0+
- **React**: 19.2+
- **Node.js**: 20.9+ (minimum, LTS recommended)
- **TypeScript**: 5.1.0+
- **Browsers**: Chrome 111+, Edge 111+, Firefox 111+, Safari 16.4+

## Automatic Optimizations (Always Enabled)

These Next.js optimizations are enabled by default. Understand them to leverage the framework properly:

- **Server Components by default**: All components are Server Components unless marked with `"use client"`
- **Code-splitting automatic**: Automatic code-splitting by route segments
- **Prefetching**: Routes visible in viewport are prefetched automatically (enhanced in v16 with incremental prefetching and layout deduplication)
- **Static Rendering**: Build-time rendering and caching
- **Aggressive Caching**: Data, components, and assets are cached by default
- **Turbopack by default**: Next.js 16 uses Turbopack for both `next dev` and `next build` by default

## Turbopack (Default in v16)

Next.js 16 uses Turbopack by default for both development and production builds.

### Key Changes

- **No flags needed**: Remove `--turbopack` or `--turbo` from your package.json scripts
- **Configuration location**: Move `experimental.turbopack` to top-level `turbopack` in next.config
- **Webpack opt-out**: Use `--webpack` flag if you need to continue using Webpack

Example configuration:

```tsx
// Next.js 16 - turbopack at top level
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  turbopack: {
    // Turbopack options
  },
};

export default nextConfig;
```

### File System Caching (beta)

Enable filesystem caching for faster compile times across restarts:

```tsx
const nextConfig: NextConfig = {
  experimental: {
    turbopackFileSystemCacheForDev: true,
  },
};
```

### Resolve Alias Fallback

If you have client code importing Node.js modules (causing "Module not found: Can't resolve 'fs'" errors):

```tsx
const nextConfig: NextConfig = {
  turbopack: {
    resolveAlias: {
      fs: {
        browser: "./empty.ts", // Load empty module in browser
      },
    },
  },
};
```

### Sass Imports

Turbopack does NOT support legacy tilde (`~`) prefix. Remove it:

```scss
// BAD (Webpack)
@import "~bootstrap/dist/css/bootstrap.min.css";

// GOOD (Turbopack)
@import "bootstrap/dist/css/bootstrap.min.css";
```

## Routing and Rendering

### Layouts

- Use Layouts to share UI across pages
- Enable partial rendering on navigation
- Place shared logic in layouts, not in individual pages

### Link Component

- ALWAYS use `<Link>` for client-side navigation
- Never use `<a>` tags for internal routes
- Prefetching is automatic unless you set `prefetch={false}`

Example:

```tsx
import Link from "next/link";

<Link href="/dashboard">Dashboard</Link>;
```

### Error Handling

- Create `error.tsx` for route-specific error boundaries
- Create `not-found.tsx` for 404 handling
- Add `app/global-error.tsx` for uncaught errors (fallback global)
- Add `app/global-not-found.tsx` for unmatched routes

### Client vs Server Components

- **Server Components (default)**: Use for data fetching, accessing backend resources
- **Client Components ("use client")**: Use ONLY when you need:
  - State (`useState`, `useReducer`)
  - Effects (`useEffect`)
  - Event handlers (`onClick`, `onChange`)
  - Browser APIs (`window`, `localStorage`)
  - Custom hooks

**CRITICAL: Check your "use client" boundaries**

- Don't add `"use client"` unnecessarily - it increases bundle size
- Move `"use client"` as deep as possible in the component tree
- Keep interactive components small and isolated

BAD:

```tsx
// app/layout.tsx
"use client"; // DON'T - makes entire app client-side
```

GOOD:

```tsx
// app/ui/search.tsx
"use client"; // Only this component is client-side
```

### Dynamic APIs (Breaking Change in v16)

**CRITICAL CHANGE**: Starting with Next.js 16, these APIs are now fully async and MUST be awaited:

- `cookies()` - now returns Promise
- `headers()` - now returns Promise
- `draftMode()` - now returns Promise
- `params` prop - now Promise in layout.js, page.js, route.js, default.js, opengraph-image, twitter-image, icon, apple-icon
- `searchParams` prop - now Promise in page.js
- `id` parameter - now Promise in sitemap and image generation functions

**IMPORTANT**: These APIs opt the ENTIRE route into Dynamic Rendering. Wrap them in `<Suspense>` boundaries when possible to limit impact.

Example:

```tsx
import { cookies } from "next/server";

export default async function Page() {
  // MUST await in Next.js 16+
  const cookieStore = await cookies();
  const token = cookieStore.get("token");

  return <div>Token: {token?.value}</div>;
}
```

For params and searchParams:

```tsx
// Next.js 16 - params and searchParams are now async
export default async function Page({
  params,
  searchParams,
}: {
  params: Promise<{ slug: string }>;
  searchParams: Promise<{ query: string }>;
}) {
  const { slug } = await params;
  const { query } = await searchParams;

  return (
    <div>
      Slug: {slug}, Query: {query}
    </div>
  );
}
```

**Type Helper**: Use `npx next typegen` to generate type helpers:

- `PageProps<'/path/[slug]'>` for pages
- `LayoutProps<'/path'>` for layouts
- `RouteContext<'/api/route'>` for route handlers

## Data Fetching and Caching

### Caching APIs (Updated in v16)

#### revalidateTag

`revalidateTag` now accepts a cacheLife profile as second argument:

```tsx
"use server";
import { revalidateTag } from "next/cache";

export async function updateArticle(articleId: string) {
  // Mark data as stale - readers see stale data while it revalidates
  revalidateTag(`article-${articleId}`, "max");
}
```

Use for content where slight delay is acceptable (blogs, docs, catalogs).

#### updateTag (New in v16)

Provides read-your-writes semantics - users see changes immediately:

```tsx
"use server";
import { updateTag } from "next/cache";

export async function updateUserProfile(userId: string, profile: Profile) {
  await db.users.update(userId, profile);

  // Expire cache and refresh immediately
  updateTag(`user-${userId}`);
}
```

Use for interactive features where users expect instant updates (forms, settings).

#### refresh (New in v16)

Refresh the client router from within a Server Action:

```tsx
"use server";
import { refresh } from "next/cache";

export async function markNotificationAsRead(notificationId: string) {
  await db.notifications.markAsRead(notificationId);

  // Refresh the notification count in header
  refresh();
}
```

#### cacheLife and cacheTag (Stable in v16)

No longer need `unstable_` prefix:

```tsx
// Remove this:
import { unstable_cacheLife as cacheLife, unstable_cacheTag as cacheTag } from "next/cache";

// Use this:
import { cacheLife, cacheTag } from "next/cache";
```

### Prefer Server Components for Data Fetching

- Fetch data in Server Components whenever possible
- Reduces client-side JavaScript
- Data never exposed to the client

Example:

```tsx
// Server Component (default)
async function getData() {
  const res = await fetch("https://api.example.com/data");
  return res.json();
}

export default async function Page() {
  const data = await getData();
  return <div>{data.title}</div>;
}
```

### Route Handlers

- Use Route Handlers (`route.ts`) to access backend from Client Components
- **NEVER call Route Handlers from Server Components** - this creates unnecessary server requests
- Server Components can directly access your backend

BAD:

```tsx
// Server Component
async function Page() {
  // DON'T - unnecessary request to your own API
  const res = await fetch("/api/data");
}
```

GOOD:

```tsx
// Server Component
async function Page() {
  // DO - direct backend access
  const data = await db.getData();
}
```

### Streaming

- Use `loading.tsx` for automatic loading UI
- Use React Suspense for granular loading states
- Stream UI progressively to the client

### Parallel Data Fetching

- Reduce waterfalls by fetching data in parallel
- Use `Promise.all()` when appropriate

Example:

```tsx
async function Page() {
  const [user, posts] = await Promise.all([getUser(), getPosts()]);

  return <div>...</div>;
}
```

### Data Caching

**Verify caching behavior**:

- By default, `fetch` requests are cached in Static Rendering
- In Dynamic Rendering, `fetch` is NOT cached by default
- For non-fetch requests (database, CMS), use `unstable_cache`

Example with cache control:

```tsx
// Cache for 1 hour
fetch("https://api.example.com", {
  next: { revalidate: 3600 },
});

// Never cache
fetch("https://api.example.com", {
  cache: "no-store",
});
```

### Static Assets

- Use `public/` directory for static assets (images, fonts)
- Automatic caching by Next.js

## UI and Accessibility

### Forms and Validation

- Use Server Actions for form submissions
- Handle validation server-side
- Return errors in a structured way

Example:

```tsx
// app/actions.ts
"use server";

export async function submitForm(formData: FormData) {
  const email = formData.get("email");

  if (!email) {
    return { error: "Email is required" };
  }

  // Process form
  return { success: true };
}
```

### Font Optimization

- Use `next/font` for automatic font optimization
- Fonts are hosted with static assets (no external requests)
- Reduces layout shift

Example:

```tsx
import { Inter } from "next/font/inter";

const inter = Inter({ subsets: ["latin"] });

export default function Layout({ children }) {
  return <body className={inter.className}>{children}</body>;
}
```

### Image Optimization

- ALWAYS use `<Image>` from `next/image`
- Automatic WebP conversion
- Lazy loading by default
- Prevents layout shift

Example:

```tsx
import Image from "next/image";

<Image
  src="/hero.jpg"
  alt="Hero image"
  width={800}
  height={600}
  priority // for above-the-fold images
/>;
```

### Script Optimization

- Use `<Script>` from `next/script`
- Automatic defer for third-party scripts

Example:

```tsx
import Script from "next/script";

<Script src="https://example.com/analytics.js" strategy="lazyOnload" />;
```

### Accessibility

- Use `eslint-plugin-jsx-a11y` (already configured)
- Test with screen readers
- Ensure semantic HTML
- Add ARIA labels when needed

## Security

### Proxy (Renamed from Middleware in v16)

The `middleware` filename is deprecated. Rename to `proxy`:

```bash
# Rename your file
mv middleware.ts proxy.ts
```

Rename the function export:

```tsx
// proxy.ts
export function proxy(request: Request) {
  // Your logic
}
```

**IMPORTANT**: The `edge` runtime is NOT supported in `proxy`. The runtime is `nodejs` and cannot be configured.

Configuration flags also renamed:

```tsx
const nextConfig: NextConfig = {
  skipProxyUrlNormalize: true, // was skipMiddlewareUrlNormalize
};
```

### Tainting

- Use tainting to prevent sensitive data exposure to client

Example:

```tsx
import { experimental_taintObjectReference } from "react";

experimental_taintObjectReference("Do not pass user object to client", user);
```

### Server Actions

- Validate authorization in Server Actions
- Never trust client input
- Sanitize and validate all data

Example:

```tsx
"use server";

export async function deletePost(id: string) {
  const session = await getSession();

  if (!session) {
    throw new Error("Unauthorized");
  }

  // Proceed with delete
}
```

### Environment Variables

- Add `.env.*` to `.gitignore`
- Only prefix public variables with `NEXT_PUBLIC_`
- Server-side secrets should NEVER be prefixed

Example:

```bash
# Server-only (secure)
DATABASE_URL=...

# Client-side (exposed)
NEXT_PUBLIC_API_URL=...
```

### Content Security Policy

- Consider implementing CSP for production
- Protect against XSS, clickjacking, code injection

## Metadata and SEO

### Metadata API

- Use Metadata API for SEO
- Add titles, descriptions, Open Graph data

Example:

```tsx
import { Metadata } from "next";

export const metadata: Metadata = {
  title: "Page Title",
  description: "Page description for SEO",
};
```

### Open Graph Images

- Create `opengraph-image.tsx` for social sharing
- Automatically generated at build time

### Sitemaps and Robots

- Generate `sitemap.xml` for search engines
- Configure `robots.txt` for crawling rules

## Type Safety

- ALWAYS use TypeScript
- Enable TypeScript plugin in Next.js
- Define types for all props, state, and API responses
- Never use `any`

## Before Production

### Partial Pre-Rendering (PPR) - v16 Changes

**Breaking Change**: The experimental PPR flag has been removed and replaced with `cacheComponents`:

```tsx
// Remove this (Next.js 15):
const nextConfig = {
  experimental: {
    ppr: true,
  },
};

// Use this (Next.js 16):
const nextConfig = {
  cacheComponents: true,
};
```

**Important**: If you're using PPR in Next.js 15 canary, stay on that version. The migration path is different and requires specific steps.

### React 19.2

Next.js 16 uses React 19.2 with new features:

- **View Transitions**: Animate elements that update inside a Transition
- **useEffectEvent**: Extract non-reactive logic from Effects
- **Activity**: Render "background activity" with display: none

### React Compiler Support (Stable in v16)

React Compiler is now stable (not experimental):

```tsx
const nextConfig: NextConfig = {
  reactCompiler: true, // Moved from experimental
};
```

Install the plugin:

```bash
pnpm install -D babel-plugin-react-compiler
```

**Note**: Expect higher compile times with this enabled.

### next/image Changes (Breaking Changes in v16)

#### Local Images with Query Strings

Now require `images.localPatterns.search` configuration:

```tsx
const nextConfig: NextConfig = {
  images: {
    localPatterns: [
      {
        pathname: "/assets/**",
        search: "?v=1",
      },
    ],
  },
};
```

#### minimumCacheTTL Default

Changed from 60 seconds to 4 hours (14400 seconds):

```tsx
// To restore old behavior:
const nextConfig: NextConfig = {
  images: {
    minimumCacheTTL: 60,
  },
};
```

#### imageSizes Default

Value `16` removed from default array. To restore:

```tsx
const nextConfig: NextConfig = {
  images: {
    imageSizes: [16, 32, 48, 64, 96, 128, 256, 384],
  },
};
```

#### qualities Default

Changed from all qualities to only `[75]`. To support multiple:

```tsx
const nextConfig: NextConfig = {
  images: {
    qualities: [50, 75, 100],
  },
};
```

#### Local IP Restriction

Local IP optimization blocked by default:

```tsx
const nextConfig: NextConfig = {
  images: {
    dangerouslyAllowLocalIP: true, // Only for private networks
  },
};
```

#### Maximum Redirects

Changed from unlimited to 3 redirects maximum:

```tsx
const nextConfig: NextConfig = {
  images: {
    maximumRedirects: 0 // Disable redirects
    // or
    maximumRedirects: 5 // Increase for edge cases
  }
}
```

#### Deprecated: next/legacy/image

Use `next/image` instead of `next/legacy/image`.

#### Deprecated: images.domains

Use `images.remotePatterns` instead:

```tsx
// BAD (deprecated)
module.exports = {
  images: {
    domains: ["example.com"],
  },
};

// GOOD
module.exports = {
  images: {
    remotePatterns: [
      {
        protocol: "https",
        hostname: "example.com",
      },
    ],
  },
};
```

### Parallel Routes Requirement (Breaking Change in v16)

All parallel route slots now REQUIRE explicit `default.js` files. Builds will fail without them.

```tsx
// app/@modal/default.tsx
import { notFound } from "next/navigation";

export default function Default() {
  notFound();
}

// Or return null:
export default function Default() {
  return null;
}
```

### ESLint Flat Config

`@next/eslint-plugin-next` now defaults to ESLint Flat Config format (aligning with ESLint v10).

Consider migrating from `.eslintrc` to flat config format. See [ESLint migration guide](https://eslint.org/docs/latest/use/configure/migration-guide).

### Concurrent Dev and Build

`next dev` now outputs to `.next/dev` (separate from build output `.next`).

For Turbopack tracing:

```bash
npx next internal trace .next/dev/trace-turbopack
```

### Removals in v16

#### AMP Support

All AMP APIs removed:

- `amp` configuration
- `next/amp` hook
- `export const config = { amp: true }`

#### next lint Command

Removed. Use ESLint directly:

```bash
# Use this instead:
pnpm eslint .
```

The `eslint` config option in next.config is also removed.

#### Runtime Configuration

`serverRuntimeConfig` and `publicRuntimeConfig` removed. Use environment variables:

```tsx
// BAD (removed):
module.exports = {
  serverRuntimeConfig: { dbUrl: process.env.DATABASE_URL },
  publicRuntimeConfig: { apiUrl: "/api" },
};

// GOOD - Server-side:
export default async function Page() {
  const dbUrl = process.env.DATABASE_URL;
  // Use directly in Server Components
}

// GOOD - Client-side:
// .env.local
NEXT_PUBLIC_API_URL = "/api";

// component:
const apiUrl = process.env.NEXT_PUBLIC_API_URL;
```

For runtime values, use `connection()` before reading `process.env`:

```tsx
import { connection } from "next/server";

export default async function Page() {
  await connection();
  const config = process.env.RUNTIME_CONFIG;
  return <p>{config}</p>;
}
```

#### devIndicators Options

Removed:

- `appIsrStatus`
- `buildActivity`
- `buildActivityPosition`

#### experimental.dynamicIO

Renamed to `cacheComponents`:

```tsx
// Remove:
module.exports = {
  experimental: { dynamicIO: true },
};

// Use:
module.exports = {
  cacheComponents: true,
};
```

#### unstable_rootParams

Removed. Alternative API coming in future minor release.

### Core Web Vitals

- Run Lighthouse in incognito mode
- Measure LCP, FID, CLS
- Use `useReportWebVitals` to send metrics to analytics

### Bundle Analysis

- Use `@next/bundle-analyzer` to identify large bundles
- Check dependency sizes before adding:
  - [Import Cost](https://marketplace.visualstudio.com/items?itemName=wix.vscode-import-cost)
  - [Package Phobia](https://packagephobia.com/)
  - [Bundle Phobia](https://bundlephobia.com/)
  - [bundlejs](https://bundlejs.com/)

### Build and Test

- ALWAYS run `next build` locally before deploying
- Test with `next start` in production mode
- Fix all build errors before pushing

## Common Pitfalls

### DON'T

- Don't call Route Handlers from Server Components
- Don't add `"use client"` to entire layouts or pages
- Don't use Dynamic APIs unnecessarily (they opt routes into dynamic rendering)
- Don't forget to wrap dynamic content in Suspense
- Don't use `<a>` tags for internal navigation
- Don't ignore TypeScript errors
- Don't skip `next build` before deploying
- **Don't forget to await Dynamic APIs in v16** (cookies, headers, params, searchParams, etc.)
- Don't use `middleware.ts` - rename to `proxy.ts`
- Don't use `unstable_cacheLife` or `unstable_cacheTag` - they're now stable
- Don't use `next/legacy/image` - use `next/image`
- Don't use `images.domains` - use `images.remotePatterns`
- Don't rely on `next lint` - use ESLint directly

### DO

- Use Server Components by default
- Add `"use client"` only when necessary and as deep as possible
- Use `<Link>` for all internal navigation
- Wrap Dynamic APIs in Suspense when appropriate
- Optimize images with `<Image>`
- Use parallel data fetching with `Promise.all()`
- Verify caching behavior of your requests
- Run Lighthouse and check Core Web Vitals
- Analyze bundle size regularly
- **Always await Dynamic APIs in v16** (cookies, headers, params, searchParams, draftMode)
- Use `updateTag` for immediate updates, `revalidateTag` for background updates
- Use `refresh` to refresh client router from Server Actions
- Rename `middleware.ts` to `proxy.ts` in v16
- Remove `unstable_` prefix from cacheLife and cacheTag
- Add `default.js` to all parallel route slots
- Use `npx next typegen` to generate type helpers for async params/searchParams

## Upgrading to Next.js 16

Run the codemod:

```bash
npx @next/codemod@canary upgrade latest
```

The codemod handles:

- Updating next.config.js for turbopack
- Migrating from next lint to ESLint CLI
- Migrating middleware to proxy convention
- Removing unstable\_ prefix from stabilized APIs
- Removing experimental_ppr config

Manual steps:

- Update all Dynamic API usages to await them
- Add default.js to parallel routes
- Update TypeScript types for async params/searchParams
- Replace runtime config with environment variables
- Update image configuration if needed

## References

- [Next.js 16 Upgrade Guide](https://nextjs.org/docs/app/guides/upgrading/version-16)
- [Next.js Production Checklist](https://nextjs.org/docs/app/guides/production-checklist)
- [Next.js Caching](https://nextjs.org/docs/app/guides/caching)
- [Server and Client Components](https://nextjs.org/docs/app/getting-started/server-and-client-components)
- [React 19.2 Announcement](https://react.dev/blog/2025/10/01/react-19-2)
