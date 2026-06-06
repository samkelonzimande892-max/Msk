# Web App Template (tRPC + Manus Auth + Database)

This template gives you a React 19 + Tailwind 4 + Express 4 + tRPC 11 stack with Manus OAuth already wired. Procedures are your contracts, types flow end to end, and authentication "just works".

---

## Quick Facts

- **tRPC-first:** define procedures in `server/routers.ts`, consume them with `trpc.*` hooks.
- **Superjson out of the box:** return Drizzle rows directly—`Date` stays a `Date`.
- **Auth baked in:** `/api/oauth/callback` handles Manus OAuth, `protectedProcedure` injects `ctx.user`.
- **Gateway-ready:** all RPC traffic is under `/api/trpc`, making it easy to route at the edge.

---

## Build Loop (Four Touch Points)

1. Update schema in `drizzle/schema.ts`, then run `pnpm db:push`.
2. Add database helpers in `server/db.ts` (return raw results).
3. Add or extend procedures in `server/routers.ts`, then wire the UI with `trpc.*.useQuery/useMutation`.
4. Build frontend experience according to `Frontend Workflow`
5. Cover your changes with Vitest specs inside `server/*.test.ts` (see `server/auth.logout.test.ts`) and run `pnpm test`.

That's it—no manual REST routes, no Axios client, no shared contract files.

---

## Key Files

```
server/auth.logout.test.ts → Reference sample vitest test file
drizzle/schema.ts → Database tables & types
server/db.ts → Query helpers (reuse across procedures)
server/routers.ts → tRPC procedures (auth + features)
client/src/App.tsx → Routes wiring & layout shells
client/src/lib/trpc.ts → tRPC client binding
client/src/pages/ → Feature UI that calls trpc hooks
```

Framework plumbing (OAuth, context, Vite bridge) lives under `server/_core`.

---

## File Structure

```
client/
  public/         ← Small configuration files ONLY (favicon.ico, robots.txt). DO NOT put images/media here.
  src/
    pages/        ← Page-level components
    components/   ← Reusable UI & shadcn/ui
    contexts/     ← React contexts
    hooks/        ← Custom hooks
    lib/trpc.ts   ← tRPC client
    App.tsx       ← Routes & layout
    main.tsx      ← Providers
    index.css     ← global style
drizzle/          ← Schema & migrations
server/
  db.ts           ← Query helpers
  routers.ts      ← tRPC procedures
storage/          ← S3 helpers
shared/           ← Shared constants & types
```

Only touch the files under "←" markers. Anything under `server/_core` or other tooling directories is framework-level—avoid editing unless you are extending the infrastructure.

### ⚠️ Handling Images & Media

**DO NOT** store images, videos, or large assets in `client/public/` or `client/src/assets/`. Local media files will cause deployment timeouts.

**Required workflow:**
1. Upload assets using the CLI: `manus-upload-file --webdev path/to/image.png`
2. Use the returned storage path directly in your code: `<img src="/manus-storage/image_a1b2c3d4.png" />`
3. Store the original local file in `/home/ubuntu/webdev-static-assets/` (outside the project directory)

Only small configuration files like `favicon.ico`, `robots.txt`, and `manifest.json` belong in `client/public/`.

Files in `client/public` are available at the root of your site—reference them with absolute paths (`/robots.txt`, etc.) from HTML templates, JSX, or meta tags.

---

## Authentication Flow

- Manus OAuth completes at `/api/oauth/callback` and drops a session cookie.
- Each request to `/api/trpc` builds context via `server/_core/context.ts`, making the current user available as `ctx.user`.
- Wrap protected logic in `protectedProcedure`; public access uses `publicProcedure`.
- Frontend reads auth state with `trpc.auth.me.useQuery()` and invokes `trpc.auth.logout.useMutation()`—no cookie plumbing required.

---

## Environment Variables

Available pre-defined system envs:
- `DATABASE_URL`: MySQL/TiDB connection string
- `JWT_SECRET`: Session cookie signing secret
- `VITE_APP_ID`: Manus OAuth application ID
- `OAUTH_SERVER_URL`: Manus OAuth backend base URL
- `VITE_OAUTH_PORTAL_URL`: Manus login portal URL (frontend)
- `OWNER_OPEN_ID`, `OWNER_NAME`: Owner's info
- `BUILT_IN_FORGE_API_URL`: Manus built-in apis (includes llm, storage, data_api, notification, etc...)
- `BUILT_IN_FORGE_API_KEY`: Bearer token used by Manus built-in apis (server-side)
- `VITE_FRONTEND_FORGE_API_KEY`: Bearer token for frontend access to Manus built-in apis
- `VITE_FRONTEND_FORGE_API_URL`: Manus built-in apis URL for frontend

Do not edit these directly in code or commit `.env` files.
The envs above are system envs, when use env in website code, refer `server/_core/env.ts` for available list.

---

## Frontend Workflow

1. Choose a design style before you write any frontend code according to Design Guide (color, font, shadow, art style). Remember to edit `client/src/index.css` for global theming and add needed font using google font cdn in `client/index.html`.
2. Design the layout and navigation structure based on app purpose. Establish navigation in App.tsx accordingly:
  - **Personal tools & internal dashboards** (finance trackers, task managers, admin panels, personal finance apps, analytics): Use DashboardLayout with sidebar navigation for consistent experience.
  - **Public-facing products** (marketing sites, e-commerce, communities): Design custom navigation (top nav, contextual nav) and landing page to attract users.
3. Start by updating `client/src/pages/Home.tsx` (the landing page shell) using shadcn/ui components to introduce links, CTAs, or feature entry points. 
4. Create or update additional components under `client/src/pages/FeatureName.tsx`, continuing to leverage shadcn/ui + Tailwind for consistent styling.
5. Register the route (or navigation entry) in `client/src/App.tsx`.
6. Read data with `const { data, isLoading } = trpc.feature.useQuery(params);`.
7. Mutate data with `trpc.feature.useMutation()`. Use optimistic updates for list operations, toggles, and profile edits. For critical operations (payments, auth), use `invalidate` with loading states.
8. Use `useAuth()` for current user state, login URL from `getLoginUrl()`, and avoid direct cookie handling.
9. Handle loading/empty/error states in the UI—tRPC already surfaces typed responses and errors.

---

## Frontend Development Guidelines

**tRPC & Data Management:**
- Use `trpc.*.useQuery/useMutation` for all backend calls—never introduce Axios/fetch wrappers.
- **Use optimistic updates for instant feedback**: ideal for adding/editing/deleting list items, toggling states, updating profiles. Use `onMutate` to update cache, `onError` to rollback (The onMutate/onError/onSettled pattern). For critical operations (payments, auth), prefer `invalidate` with explicit loading states.
- When using `invalidate` as fallback: call `trpc.useUtils().feature.invalidate()` in mutation's `onSuccess`.
- Auth state comes from `useAuth()`; do not manipulate cookies manually.

**UI & Styling:**
- Prefer shadcn/ui components for interactions to keep a modern, consistent look; import from `@/components/ui/*` (e.g., `button`, `card`, `dialog`).
- Compose Tailwind utilities with component variants for layout and states; avoid excessive custom CSS. Use built-in `variant`, `size`, etc. where available.
- Preserve design tokens: keep the `@layer base` rules in `client/src/index.css`. Utilities like `border-border` and `font-sans` depend on them.
- Consistent design language: use spacing, radius, shadows, and typography via tokens. Extract shared UI into `components/` for reuse instead of copy‑paste.
- Accessibility and responsiveness: keep visible focus rings and ensure keyboard reachability; design mobile‑first with thoughtful breakpoints.
- Theming: Choose dark/light theme to start with for ThemeProvider according to your design style (dark or light bg), then manage colors pallette with CSS variables in `client/src/index.css` instead of hard‑coding to keep global consistency.
- Micro‑interactions and empty states: add motion, empty states, and icons tastefully to improve quality without distracting from content.
- Navigation: For internal tools/admin panels, use persistent sidebar. For public-facing apps, design navigation based on content structure (top nav, side nav, or contextual)—ensure clear escape routes from all pages.
- Placeholder UI elements: When adding structural placeholders (nav items, table actions) for not-yet-implemented features, show toast on click ("Feature coming soon"). Inform user which elements are placeholders when presenting work.

**React Best Practices:**
- Never call setState/navigation in render phase → wrap in `useEffect`

**Customized Defaults:**
This template customizes some Tailwind/shadcn defaults for simplified usage:
- `.container` is customized to auto-center and add responsive padding (see `index.css`). Use directly without `mx-auto`/`px-*`. For custom widths, use `max-w-*` with `mx-auto px-4`.
- `.flex` is customized to have `min-width:0` and `min-height:0` by default
- `button` variant `outline` uses transparent background (not `bg-background`). Add bg color class manually if needed.

---

## 🎨 Design Guide

When generating frontend UI, avoid generic patterns that lack visual distinction:
- Avoid generic full-page centered layouts—prefer asymmetric/sidebar/grid structures for landing pages and dashboards
- Avoid applying dashboard/sidebar patterns to public-facing apps (forums, communities, e-commerce)—reserve those for internal tools
- When user provides vague requirements, make creative design decisions (choose specific color palette, typography, layout approach)
- Prioritize visual diversity: combine different design systems (e.g., one color scheme + different typography + another layout principle)
- For landing pages: prefer asymmetric layouts, specific color values (not just "blue"), and textured backgrounds over flat colors
- For dashboards: use defined spacing systems, soft shadows over borders, and accent colors for hierarchy

---

## Animation Guide

Bake motion taste in from the first line of code. Snappy, physically intuitive interactions are not a polish pass — they are part of the initial build.
- Decide whether to animate at all: keyboard-initiated actions (command palettes, shortcuts) must be instant — never animate them. High-frequency interactions (hover, list nav) should be minimal. Reserve richer motion for occasional events (modals, drawers, toasts) and rare delight moments (onboarding).
- Keep UI animations under 300ms. A 180ms dropdown feels significantly better than a 400ms one. Typical ranges: button press 100–160ms, tooltips 125–200ms, dropdowns 150–250ms, modals/drawers 200–500ms.
- Use strong custom easings, not the weak CSS defaults. Default to a snappy ease-out for entering/exiting UI: `--ease-out: cubic-bezier(0.23, 1, 0.32, 1);`. For moving/morphing use `--ease-in-out: cubic-bezier(0.77, 0, 0.175, 1);`. NEVER use `ease-in` for UI animations — it feels sluggish.
- Buttons must feel responsive: add `transform: scale(0.97)` on `:active` with a ~160ms ease-out transition so the UI confirms it heard the user.
- Never animate from `scale(0)` — nothing in the real world appears from nothing. Start from `scale(0.95)` combined with `opacity: 0`.
- Origin-aware popovers/dropdowns: scale in from the trigger point (e.g. `transform-origin: var(--radix-popover-content-transform-origin)`). Modals are the exception and stay centered.
- Prefer CSS transitions over @keyframes for dynamic UI state. Transitions can be interrupted and reversed smoothly mid-flight; keyframes restart from zero and feel broken when interrupted.
- Only animate `transform` and `opacity` for motion — they run on the GPU and skip layout/paint. Avoid animating `width`, `height`, `padding`, `margin`, `top/left` unless absolutely necessary.
- Stagger grouped entrances by 30–80ms per item to create a cascading reveal instead of a wall of motion.
- Asymmetric timing for deliberate actions: hold-to-confirm should be slow and linear on press (e.g. 2s linear), but release/cancel should snap back fast (~200ms ease-out).
- Respect `prefers-reduced-motion`: gate non-essential motion behind `@media (prefers-reduced-motion: no-preference)`.

---

## Feature Checklist

- [ ] Tables updated in `drizzle/schema.ts`, migrations pushed (`pnpm db:push`)
- [ ] Query helper added in `server/db.ts` (returns raw Drizzle rows)
- [ ] Procedure created in `server/routers.ts` (choose `public` vs `protected`)
- [ ] UI calls the procedure via `trpc.*.useQuery/useMutation`
- [ ] Success + error paths verified in the browser

---

## Pre-built Components

Before implementing UI features, check if these components already exist:

Dashboard & Layout:
- `client/src/components/DashboardLayout.tsx` - Full dashboard layout with sidebar navigation, auth handling, and user profile. Use this for any admin panel or dashboard-style app instead of building from scratch.
- `client/src/components/DashboardLayoutSkeleton.tsx` - Loading skeleton for dashboard during auth checks

Chat & Messaging:
- `client/src/components/AIChatBox.tsx` - Full-featured chat interface with message history, streaming support, and markdown rendering. Use this for any chat/conversation UI instead of building from scratch.

Maps:
- `client/src/components/Map.tsx` - Google Maps integration with proxy authentication. Provides MapView component with onMapReady callback for initializing Google Maps services (Places, Geocoder, Directions, Drawing, etc.). All map functionality works directly in the browser.

When implementing features that match these categories, MUST evaluate the component first to decide whether to use or customize it.

---

## Internal Tools & Admin Panels

For certain app types, this template provides DashboardLayout—a standardized sidebar pattern.

**Use DashboardLayout for:**
- Admin/management dashboards
- Personal productivity apps (task managers, note-taking)
- Analytics/monitoring tools

**Do NOT use for:**
- Public content platforms (forums, blogs, social networks)
- E-commerce storefronts
- Marketing/landing sites

**Layout & Navigation**
- Use `DashboardLayout` component from `client/src/components/DashboardLayout.tsx` and remove any page-level headers to avoid duplication.
- When use DashboardLayout, read its content before making changes and preserve its core structure by default.

**Role-based Access Control**
When building apps with distinct access levels (e.g., e-commerce with public home, user account, admin panel):
- The `user` table includes a `role` field (enum: `admin` | `user`) for identity separation
- Use `ctx.user.role` in procedures to gate admin-only operations
- Wrap admin-only backend logic in `adminProcedure`
- Frontend can conditionally render navigation/routes based on `useAuth().user?.role`

Example procedure pattern:
```ts
adminOnlyProcedure: protectedProcedure.use(({ ctx, next }) => {
  if (ctx.user.role !== 'admin') throw new TRPCError({ code: 'FORBIDDEN' });
  return next({ ctx });
}),
```

**Managing Admins**
- To promote a user to admin, update the `role` field directly in the database via the system UI or SQL
- If you need additional roles beyond `admin`/`user`, extend the enum in `drizzle/schema.ts` and push the migration

---

## LLM Integration

Use the preconfigured LLM helpers. Credentials are injected from the platform (no manual setup required).

```ts
import { invokeLLM } from "./server/_core/llm";

/**
 * Simple chat completion
 * type Role = "system" | "user" | "assistant" | "tool" | "function";
 * type TextContent = {
 *   type: "text";
 *   text: string;
 * };
 *
 * type ImageContent = {
 *   type: "image_url";
 *   image_url: {
 *     url: string;
 *     detail?: "auto" | "low" | "high";
 *   };
 * };
 *
 * type FileContent = {
 *   type: "file_url";
 *   file_url: {
 *     url: string;
 *     mime_type?: "audio/mpeg" | "audio/wav" | "application/pdf" | "audio/mp4" | "video/mp4" ;
 *   };
 * };
 *
 * export type Message = {
 *   role: Role;
 *   content: string | Array<ImageContent | TextContent | FileContent>
 * };
 *
 * Supported parameters:
 * messages: Array<{
 *   role: 'system' | 'user' | 'assistant' | 'tool',
 *   content: string | { tool_call: { name: string, arguments: string } }
 * }>
 * tool_choice?: 'none' | 'auto' | 'required' | { type: 'function', function: { name: string } }
 * tools?: Tool[]
 */
const response = await invokeLLM({
  messages: [
    { role: "system", content: "You are a helpful assistant." },
    { role: "user", content: "Hello, world!" },
  ],
});
```

Tips
- Always call llm functions from server-side code (e.g., inside tRPC procedures), to avoid exposing your API key.
- LLM calls deduct from this project's credit balance.
- All models support streaming, but `invokeLLM()` doesn't expose `stream` — modify the helper to pass `stream: true` and parse the SSE response if you need it. When proxying SSE, listen on `res` close (not `req`) and guard with a `finished` flag, or the upstream gets aborted after the first event.
- LLM responses often contain markdown. Use `<Streamdown>{content}</Streamdown>` (imported from `streamdown`) to render markdown content with proper formatting and streaming support.

### Listing Available Models

```ts
import { listLLMModels } from "./server/_core/llm";

const { data } = await listLLMModels();
const ids = data.map(m => m.id);
```

Returns OpenAI-standard model metadata for each available ID. From the project shell you can also peek at it directly: `curl "$BUILT_IN_FORGE_API_URL/v1/models" -H "Authorization: Bearer $BUILT_IN_FORGE_API_KEY"`.

**Combine with `invokeLLM`** to discover IDs at runtime instead of hardcoding:

```ts
import { invokeLLM, listLLMModels } from "./server/_core/llm";

const { data } = await listLLMModels();
const model = data.find(m => m.id.startsWith("claude-"))?.id;

const response = await invokeLLM({
  model,
  messages: [{ role: "user", content: "Hello" }],
});
```

### Thinking / Reasoning

`invokeLLM()` forwards `thinking` and `reasoning` extension params unchanged (no defaults). Per model family:

- OpenAI gpt-5 family — `reasoning: { effort: "minimal" | "low" | "medium" | "high" }`
- Anthropic claude family — `thinking: { type: "enabled", budget_tokens: 2048 }`
- Google gemini family — `thinking: { budget_tokens: 1024 }`

```ts
await invokeLLM({
  model: "claude-sonnet-4-6",
  messages: [...],
  thinking: { type: "enabled", budget_tokens: 2048 },
});

await invokeLLM({
  model: "gpt-5",
  messages: [...],
  reasoning: { effort: "low" },
});
```

For the exact shape per model, check `capabilities.thinking_example` from the `/models` catalog (see Tips above).

### Structured Responses (JSON Schema)

Ask the model to return structured JSON via `response_format`:

```ts
import { invokeLLM } from "./server/_core/llm";

const structured = await invokeLLM({
  messages: [
    { role: "system", content: "You are a helpful assistant designed to output JSON." },
    { role: "user", content: "Extract the name and age from the following text: \"My name is Alice and I am 30 years old.\"" },
  ],
  response_format: {
    type: "json_schema",
    json_schema: {
      name: "person_info",
      strict: true,
      schema: {
        type: "object",
        properties: {
          name: { type: "string", description: "The name of the person" },
          age: { type: "integer", description: "The age of the person" },
        },
        required: ["name", "age"],
        additionalProperties: false,
      },
    },
  },
});

// The model responds with JSON content matching the schema.
// Access via `structured.choices[0].message.content` and JSON.parse if needed.
```
The helpers mirror the Python SDK semantics but produce JavaScript-first code, keeping credentials inside the server and ensuring every environment has access to the same token.

---

## Voice Transcription Integration

Use the preconfigured voice transcription helper that converts speech to text using Whisper API, no manual setup required.

Example usage:
```ts
import { transcribeAudio } from "./server/_core/voiceTranscriptio
