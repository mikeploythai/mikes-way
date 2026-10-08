# Stack preferences

These are the tools Mike likes working with. When the user is figuring out how to build something, suggest the relevant ones and explain why you'd use them. Ask before committing them to a new service, dependency, or architecture.

Don't replace a working stack just to match this list, or install everything here at the start of a project. Use what the app actually needs. Check the current docs, compatibility, maintenance, and pricing before adding something.

## General

Prefer full-stack TypeScript, but keep the setup proportional to what we're building:

- For a quick mockup, plain HTML, CSS, and JavaScript are fine. Add the Tailwind CLI, a small server, htmx, or Alpine.js if you need them.
- For a site that's mostly content, use Astro. Keep the content in the project unless people need a CMS to edit it. Sanity and self-hosted Payload are the preferences there.
- For an app without SSR, start with Vite and Convex.
- If the app needs a traditional server, use Vite with Elysia or Hono. Elysia is the current preference. Expose OpenAPI and generate the client with Hey API's Vite plugin and its TanStack Query integration.
- Use TanStack Start or Next.js App Router if the app needs SSR or benefits from keeping the frontend and backend in one framework.

Default new web apps and mockups to no indexing until the user says they should be discoverable. Add a `robots.txt` that disallows all crawlers, plus a `noindex, nofollow` meta tag or `X-Robots-Tag` response header. Remove them when indexing is wanted. Robots directives are not access control.

### Toolchain

Use [Vite+](https://viteplus.dev/guide/) for the web toolchain. Let it manage Node.js, pnpm, Vite, Vitest, Oxlint, and Oxfmt, and use the `vp` commands. Stick with pnpm unless the project already uses or requires something else.

For a new frontend, Mike prefers scaffolding with Vite+ and pnpm:

```sh
vp create vite@latest --package-manager pnpm
```

Immediately after initialization, run this from the newly created app directory:

```sh
pnpm up --latest
```

Temporary, as of October 2026: Hey API 0.99.0 crashes under TypeScript 7 ([hey-api#4235](https://github.com/hey-api/hey-api/issues/4235)). In projects that use Hey API, pin `typescript` to `^6.0.3` after `pnpm up --latest` and say why in the PR, because `pnpm up --latest` moves it back to 7. Remove the pin and this note once a stable fix ships.

Use pnpm workspaces for a monorepo. Add Turborepo if you need its task dependencies, caching, or remote execution. Make the dev command start the server and wait for its OpenAPI document before starting the frontend. When Hey API reads the document from a URL, it needs the server running to generate the client. If you're using Drizzle, start Drizzle Studio too.

Keep linting and formatting under one `check` script. Add the database and UI shortcuts if those tools are in the project:

```json
{
  "scripts": {
    "check": "vp check --fix",
    "db:studio": "drizzle-kit studio",
    "ui:add": "shadcn add"
  }
}
```

For end-to-end tests, have the subagent(s) use the browser and try the flow first. Use Playwright only when the flow has to run unattended, such as in CI. Use Vitest only to test a system in isolation. Follow the [testing rules](../SKILL.md#testing) for both.

### Frontend toolchain

Read [frontend toolchain guidance](frontend-toolchain.md) when setting up or changing Vite+, linting, or styling integrations. It covers Mike's `vite.config.ts`, the Hey API client setup, required lint-plugin dev dependencies, TanStack-only presets, `resolve.tsconfigPaths` for aliases, and the `@tailwindcss/vite` dev dependency preference. Use the Oxc React Compiler through `reactVite({ compiler: true })` with `oxc-transform-react` as a dev dependency, and remove the Babel compiler setup it replaces.

For Tailwind v4 projects, extend Ultracite's `shadcn` preset in `vite.config.ts`, following the compatibility and configuration guidance in [frontend toolchain guidance](frontend-toolchain.md#tailwind-design-system-linting). It works without shadcn/ui. Keep its six rules on, with the layout allowances and design-system source override it ships. Preserve other existing rule policies and use narrow, accepted exceptions.

When a Vite app needs local HTTPS, use `vite-plugin-mkcert`. Add it as a dev dependency and add `mkcert()` to the Vite plugins. Prefer it over loading the ASP.NET dev cert in `vite.config.ts`, because shelling out to `dotnet` fails `sonarjs/no-os-command-from-path`. Its default certificate covers only `localhost`, IPs, and `server.host`, so pass a dev hostname through `hosts`, such as `mkcert({ hosts: ["app.dev.localhost"] })`. Use the HTTPS origin for MSAL React development and match it in the app's registered redirect URI.

### Frontend choices

Start with React, React Compiler, and TanStack Router. Use shadcn/ui with Base UI for components. Use React Aria when the app has major accessibility requirements.

```sh
pnpm dlx shadcn@latest init --preset b5defoKzi --template vite
pnpm dlx shadcn@latest init --preset b5defoKzi --base aria --template vite
```

When using TanStack Router, prefer file-based routing. Nest groups with several routes in folders, and add `route.tsx` when the group needs a shared layout, loader, or other parent behavior:

```text
folderA/
├── page1.$id.tsx
├── page2.tsx
└── route.tsx
folderB.onlyPage.$id.tsx
```

Dot notation is fine for a nest with only one page, as in the `folderB` example. Apply the route filename lint override and project-level VS Code settings in [frontend toolchain guidance](frontend-toolchain.md).

#### Shadcn CLI

In a shadcn project, inspect the installed components and project configuration before writing a new component. The CLI can find documentation and registry components:

```sh
pnpm dlx shadcn@latest info --json
pnpm dlx shadcn@latest docs button dialog select
pnpm dlx shadcn@latest search @shadcn -q "sidebar"
pnpm dlx shadcn@latest view @shadcn/button
```

`docs` returns resolved documentation, example, and API-reference URLs. Read those pages before implementing or debugging the component. `info` identifies the framework, component base, Tailwind paths, aliases, and installed components. `search`, also available as `list`, finds registry items; `view` inspects their metadata and source.

Use the project's package runner and pass `--cwd <path>` for the app workspace when needed. Use the project's existing component base; `docs` supports `--base base|radix|aria` and `--json`. Consult the [CLI reference](https://ui.shadcn.com/docs/cli#docs) for current options.

Preview component changes with `pnpm dlx shadcn@latest add button --dry-run`. The add command's `--diff <file>` and `--view <file>` options inspect individual files without writing changes. Use `add button --diff button.tsx` to compare an existing component with upstream and preserve local customizations when updating it. Use `add --diff` instead of the standalone `diff` command. Plain `add`, `init`, `apply`, `migrate`, and `eject` modify the project; use them when the requested work calls for those changes. See the [upstream CLI guide](https://github.com/shadcn-ui/ui/blob/main/skills/shadcn/cli.md).

Use `pnpm ui:add <component>` when that shortcut exists, or `pnpm dlx shadcn@latest add <component>`, then adapt its source. Write one from scratch only when the catalog has nothing that fits the needed interaction.

#### Other frontend libraries

For the rest, these are the preferences. Add them as you need them:

- TanStack Query for data fetching, Pacer for timing and rate control, and Hotkeys for keyboard shortcuts.
- TanStack Table for data grids, Virtual for long lists, and Charts for data visualization.
- TanStack Form for new forms. React Hook Form is fine if it's already in the project or has something the app needs.
- Zod for runtime validation and T3 Env for typed environment variables.
- dnd kit for drag and drop, React Dropzone for file drops, React Email for email templates, and Tiptap for rich text.
- Sonner, Vaul, and cmdk when the component base does not already cover toasts, drawers, or command menus.
- Jotai when React context isn't satisfactory.

Prefer Hugeicons or Lucide for icons. Use Tailwind CSS with proper design tokens, and let shadcn's lint plugin enforce the design system; see [Tailwind design-system linting](frontend-toolchain.md#tailwind-design-system-linting). Prefer logical properties and utilities over physical ones, as described in the Codestyle section of [Mike's way](../SKILL.md).

Use Fontsource for app fonts. Instrument Sans and Cascadia Code are the current favorites. Other fonts to consider are Inter, Space Mono, JetBrains Mono, Space Grotesk, Manrope, Figtree, DM Mono, Bricolage Grotesque, Mona Sans, Unbounded, Roboto Flex, Google Sans, and Google Sans Flex. Pick something that suits the app. Don't use the same font for everything just because it's on this list.

### Backend and platform choices

Prefer Convex for the backend. Check what it and its components already do before writing auth, email, rate limiting, scheduled or durable work, presence, aggregates, migrations, or blob storage yourself.

Prefer Better Auth. Use Clerk if the user wants managed auth. Use Resend for email, though Nodemailer is fine for something small or an integration that needs it.

For a traditional server, use Elysia or Hono and generate the frontend client from OpenAPI with Hey API's Vite plugin. Use Drizzle if you need an ORM. For data:

- Use Cloudflare D1 for SQLite, or local Node.js SQLite for isolated development data. Use Durable Objects when the app needs their coordination model.
- Use Neon or PlanetScale for Postgres, after checking the cost with the user.
- Use PlanetScale for MySQL, after checking the cost with the user.
- Cloudflare R2 for blobs, with UploadThing as a managed alternative.

Supabase is fine too. Convex is the preference.

For AI, consider TanStack AI if you're already using TanStack. It can stream over HTTP. The Vercel AI SDK is also a good option, especially if it has the provider or UI support you need. If the stream comes from a Convex HTTP action or a non-TypeScript server, test that the client can actually read it. Use OpenRouter if Cloudflare's AI tools aren't a better choice for the app.

Prefer Cloudflare for infrastructure. One account can cover hosting, domains, secrets, Workers, Workers AI, AI Gateway, D1, Durable Objects, and R2. Vercel is fine if it's a better choice for the app.

When asked to prepare an app for Cloudflare deployment, target Cloudflare Workers and make the project ready for deployment from Cloudflare's web dashboard. Use Wrangler tooling and configuration when useful, but don't run `wrangler deploy` unless Mike explicitly asks.

Start with PostHog for observability. Use Sentry if you need more help diagnosing errors. For simple analytics, self-hosted Umami with Postgres or One Dollar Stats is enough.

## Mike's work environment

This part is mostly for Mike's day job, where production runs on on-prem IIS and Microsoft SQL Server. There's still .NET Framework, with a move toward modern .NET. It might help someone working in a similar setup, but don't assume every project has these constraints.

Use a Vite frontend and generate the OpenAPI client with Hey API's Vite plugin. A small server can use `.ashx` handlers and `NSwag.Generation`. On .NET Framework, prefer Microsoft ASP.NET Web API with OWIN if the app supports it. On modern .NET, use minimal APIs. Build the frontend into `wwwroot` if that's where the IIS app serves it from: set `build.outDir: "../wwwroot"` and `build.emptyOutDir: true`, and gitignore `wwwroot`.

On modern .NET, generate the OpenAPI document at build time with `Microsoft.AspNetCore.OpenApi` and `Microsoft.Extensions.ApiDescription.Server`. Set `OpenApiDocumentsDirectory` to `client` and `OpenApiGenerateDocumentsOptions` to `--file-name openapi`, and point Hey API at that file, so the client builds without a running server. Skip startup-only work, such as migrations or seeding, while the generator runs: `Assembly.GetEntryAssembly()?.GetName().Name == "GetDocument.Insider"`. Follow the Hey API setup in [frontend toolchain guidance](frontend-toolchain.md#hey-api).

Write endpoints so the document is accurate:

- Name each endpoint with `.WithName(...)` for stable operationIds.
- Return typed results.
- Put DataAnnotations on records as `[property: …]`.
- Make request types `public` so `AddValidation()` generates validators for them.
- Set JSON number handling to strict so `int` isn't emitted as `integer | string`.

In development, proxy API paths from Vite to Kestrel:

- Target `https://localhost:<port>`, because Node on Windows can't resolve `*.dev.localhost`.
- Set `changeOrigin: false` so OIDC redirect URIs and cookies stay on the Vite origin.
- Set `secure: false`; it only affects the loopback hop.
- Type entries with `ProxyOptions` from `vite-plus`. Move them into a shared constant only when proxying many paths.
- Fix `server.port` with `strictPort: true`, because dev origins are registered redirect URIs.

Use MSAL React in the browser, `Microsoft.Identity.Web` on the server, and the Microsoft Graph SDK for Microsoft 365 integration. Prefer Azure Key Vault for secrets. The fallback is gitignored `secrets.config` for `<appSettings>` and `database.config` for connection strings.

Production uses on-prem SQL Server, but do all data work in SQL Server Express LocalDB during development so it doesn't affect the production database. LocalDB runs the same engine as production, so the migration stays plain T-SQL. When Mike is ready, have a subagent create a SQL Server migration based on the changes made in LocalDB, and check it against a fresh LocalDB or test SQL Server before handing it back for review. Creating the migration doesn't mean running it against production.

Prefer `Microsoft.Data.SqlClient` for SQL Server, and add Dapper if it makes the queries or mapping easier. Use `System.Text.Json` if the target framework supports what you need.

For AI, prefer Microsoft Semantic Kernel with Microsoft Foundry or OpenRouter. Stream responses over HTTP with server-sent events if you only need server-to-client updates. You can use TanStack AI's React client if the server sends the stream format it expects.

If Web Forms is unavoidable, keep it close to plain web development. Disable ViewState, render markup in `.aspx`, and use `.aspx.cs` only to load data or add markup through `asp:Literal` and `asp:Placeholder`. Don't add other server or custom controls. Use `.ashx` for API endpoints, htmx with Alpine.js or jQuery for browser behavior, jsDelivr for CDN dependencies, and a local Tailwind CLI build. If Bootstrap is unavoidable, check the current Bootstrap 6 prerelease before using it. Set up `Microsoft.AspNet.FriendlyUrls.Core`.
