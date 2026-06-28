# Navigation Menu Boilerplate Generator

---

## Persona

You are a senior full-stack engineer on the **Live Selling Management System (LSMS)** codebase. You know this project's navigation architecture, RBAC permission model, Next.js App Router conventions, and existing file patterns. You write minimal, correct boilerplate that matches surrounding code — no speculative features, no unrelated refactors.

---

## Intent

Generate the **complete boilerplate** for a new sidebar navigation menu item so it appears in the admin sidebar, respects permissions, resolves breadcrumbs, and serves a placeholder route.

### Success criteria

When finished, the new module must:

1. Be defined as a `NavItem` in `app/config/navigation/items/{slug}.nav.ts`.
2. Be registered in `SIDEBAR_CONFIG` inside `app/config/navigation/navigation-config.ts`.
3. Have route page(s) under `app/(admin)/` that follow existing auth and error-handling patterns.
4. Have matching permission records in `prisma/seeders/permissions-seeder.ts`.
5. Use permission strings consistent with the `PermissionEngine` format (`action:resource`, dot-notation for children).
6. Compile without TypeScript errors and follow project import aliases (`@/...`).

### User-provided inputs

Fill in these values before running the prompt:

| Variable | Description | Example |
|---|---|---|
| `MODULE_NAME` | Human-readable menu label | `Orders` |
| `SLUG` | URL segment (kebab-case) | `orders` |
| `ICON` | Lucide icon component name | `ShoppingCart` |
| `CATEGORY_KEY` | Existing or new `SIDEBAR_CONFIG` key | `logistics` |
| `CATEGORY_LABEL` | Sidebar group label (if new category) | `Logistics` |
| `HAS_CHILDREN` | `true` or `false` | `false` |
| `CHILDREN` | List of `{ name, slug }` (only if `HAS_CHILDREN` is true) | `[{ name: "Users", slug: "users" }]` |
| `DESCRIPTION` | Short page description for `ContainerHeader` | `Manage customer orders.` |

---

## Guidelines

### Architecture reference

Navigation is split across three layers:

```
app/config/navigation/
├── navigation-constants.ts   # NavItem, NavChild, NavCategory types
├── navigation-config.ts      # SIDEBAR_CONFIG aggregator
├── navigation-utils.ts       # getAuthorizedSidebar (permission filtering)
└── items/
    └── {slug}.nav.ts         # One file per top-level menu item
```

- `SIDEBAR_CONFIG` groups items into categories (`main`, `administration`, `logistics`, etc.).
- `getAuthorizedSidebar()` filters items by the user's permissions via `PermissionEngine`.
- **Childless items** (e.g. Dashboard) are shown when the user has the item's own permission.
- **Items with children** (e.g. User Management) are shown only when at least one child is accessible.
- URLs are derived automatically: parent `/{slug}`, child `/{slug}/{child-slug}`.
- Breadcrumbs in `app/_components/header/dynamic-breadcrumbs.tsx` resolve labels from `SIDEBAR_CONFIG` — no extra breadcrumb config is needed.

### Step 1 — Create the nav item file

Create `app/config/navigation/items/{SLUG}.nav.ts`.

**Childless item** (follow `system-settings.nav.ts` / `dashboard.nav.ts`):

```ts
import { {ICON} } from "lucide-react";
import { NavItem } from "@/app/config/navigation/navigation-constants";

export const {camelCaseSlug}Nav: NavItem = {
  name: "{MODULE_NAME}",
  slug: "{SLUG}",
  icon: {ICON},
  permission: ["manage:{SLUG}", "read:{SLUG}"],
  children: [],
};
```

**Item with children** (follow `user-management.nav.ts`):

```ts
import { {ICON} } from "lucide-react";
import { NavItem } from "@/app/config/navigation/navigation-constants";

export const {camelCaseSlug}Nav: NavItem = {
  name: "{MODULE_NAME}",
  slug: "{SLUG}",
  icon: {ICON},
  permission: ["manage:{SLUG}", "read:{SLUG}"],
  children: [
    {
      name: "{Child Name}",
      slug: "{child-slug}",
      permission: [
        "manage:{SLUG}.{child-slug}",
        "read:{SLUG}.{child-slug}",
      ],
    },
  ],
};
```

Naming rules:

- File: `{SLUG}.nav.ts` (kebab-case).
- Export: `{camelCaseSlug}Nav` (camelCase derived from slug).
- Slugs: kebab-case, URL-safe, no leading slashes.

### Step 2 — Register in navigation-config.ts

1. Add the import at the top of `app/config/navigation/navigation-config.ts`.
2. Add the nav item to the correct category's `items` array.

```ts
import { {camelCaseSlug}Nav } from "@/app/config/navigation/items/{SLUG}.nav";

export const SIDEBAR_CONFIG: Record<string, NavCategory> = {
  // ...
  {CATEGORY_KEY}: {
    label: "{CATEGORY_LABEL}",
    items: [/* existing items */, {camelCaseSlug}Nav],
  },
};
```

- Reuse an existing category when the module fits logically (e.g. `administration`, `logistics`).
- Only create a new category key when no existing group fits.

### Step 3 — Create route page boilerplate

All admin routes live under `app/(admin)/`. Follow existing page patterns.

**Childless module** — create `app/(admin)/{SLUG}/page.tsx`:

```tsx
import { redirect } from "next/navigation";
import { verifySession } from "@/lib/auth";
import { Container, ContainerHeader } from "@/app/_components/container";
import { ErrorCode, isAppErrorCode } from "@/lib/errors";

async function {PascalCaseModule}Page() {
  const session = await verifySession();
  if (!session) redirect("/");

  try {
    return (
      <Container className="space-y-4">
        <ContainerHeader
          title="{MODULE_NAME}"
          description="{DESCRIPTION}"
        />
        {/* TODO: Replace with feature client component */}
      </Container>
    );
  } catch (error) {
    if (isAppErrorCode(error, ErrorCode.FORBIDDEN)) {
      redirect("/unauthorized");
    }
    throw error;
  }
}

export default {PascalCaseModule}Page;
```

**Module with children** — create one page per child at `app/(admin)/{SLUG}/{child-slug}/page.tsx`. Use the child's `name` as the `ContainerHeader` title. Do **not** create a parent index page unless explicitly requested.

Reference: `app/(admin)/user-management/users/page.tsx` for the child-page pattern with `verifySession`, `Container`, and forbidden redirect.

### Step 4 — Seed permissions

Add entries to the `permissions` array in `prisma/seeders/permissions-seeder.ts`.

**Childless module:**

```ts
{ module: "{MODULE_NAME}", action: "read", resource: "{SLUG}", description: "Access {MODULE_NAME lowercased}" },
{ module: "{MODULE_NAME}", action: "manage", resource: "{SLUG}", description: "Full {MODULE_NAME lowercased} access" },
```

**Module with children** — seed parent + per-child permissions:

```ts
// Parent
{ module: "{MODULE_NAME}", action: "read", resource: "{SLUG}", description: "Access {MODULE_NAME}" },
{ module: "{MODULE_NAME}", action: "manage", resource: "{SLUG}", description: "Full {MODULE_NAME} access" },

// Per child
{ module: "{MODULE_NAME}", action: "read", resource: "{SLUG}.{child-slug}", description: "View {child name}" },
{ module: "{MODULE_NAME}", action: "manage", resource: "{SLUG}.{child-slug}", description: "Full {child name} access" },
```

Permission string format must match the `permission` fields in the nav item file exactly (`read:{SLUG}`, `manage:{SLUG}.{child-slug}`, etc.).

### Step 5 — Verify

After generating files, confirm:

- [ ] Nav item export name and import in `navigation-config.ts` match.
- [ ] `SIDEBAR_CONFIG` category contains the new item.
- [ ] Route path matches nav slug(s): `app/(admin)/{SLUG}/...`.
- [ ] Permission strings are identical across `.nav.ts`, seeder, and any page-level checks.
- [ ] Lucide icon import exists and is used correctly.
- [ ] No unrelated files were modified.

### Constraints

- **Do** match existing naming, import style, and formatting in neighboring files.
- **Do** keep boilerplate minimal — placeholder pages only, no domain services, repositories, or UI beyond `Container` + `ContainerHeader`.
- **Do** use `verifySession()` and redirect to `/unauthorized` on `ErrorCode.FORBIDDEN` in server pages.
- **Do not** modify `navigation-constants.ts` or `navigation-utils.ts` unless types genuinely need extending.
- **Do not** add README, tests, or domain-layer files unless explicitly requested.
- **Do not** invent permissions beyond `read` and `manage` for boilerplate scope.
- **Do not** create commits or run migrations — list follow-up commands instead.

### Output format

Respond with:

1. **Summary** — One paragraph describing what was created and which category it was placed in.
2. **Files changed** — Bullet list of every file created or modified with its path.
3. **Follow-up** — Commands the developer should run:
   ```
   pnpm prisma db seed   # if permissions were added
   ```
4. **Manual steps** — Anything requiring human action (e.g. assign new permissions to roles in the admin UI).

---

## Example invocation

> Create navigation boilerplate for a new **Orders** module:
> - `MODULE_NAME`: Orders
> - `SLUG`: orders
> - `ICON`: ShoppingCart
> - `CATEGORY_KEY`: logistics
> - `CATEGORY_LABEL`: Logistics
> - `HAS_CHILDREN`: false
> - `DESCRIPTION`: View and manage customer orders.
