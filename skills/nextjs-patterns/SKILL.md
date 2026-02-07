---
name: nextjs-patterns
description: Next.js App Router patterns. Auto-loaded in Next.js projects.
---

# Next.js Patterns

## App Router
- Server Components by default — "use client" only for interactivity
- Colocate page-specific components in route directory
- Shared components in src/components/
- Use loading.tsx and error.tsx for route-level states

## Data Fetching
- Server Components: fetch directly
- Prefer server actions for mutations over API routes
- revalidatePath/revalidateTag for cache invalidation
- API routes only for webhooks and external integrations

## Styling
- Tailwind CSS utility-first
- cn() helper (clsx + tailwind-merge) for conditional classes
- cva for component variants
- No inline styles or CSS modules

## File Naming
- page.tsx, layout.tsx, loading.tsx, error.tsx, not-found.tsx
- Components: PascalCase (UserCard.tsx)
- Utilities: camelCase (formatDate.ts)
