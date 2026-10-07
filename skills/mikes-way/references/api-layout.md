# API layout

Use a layered layout for standalone APIs. This is the standard "package by layer" approach, kept to three folders: `endpoints/`, `services/`, and `data/`. The other common approach is "package by feature", also called vertical slices, where each folder holds one feature's routes, logic, and data together.

## When it applies

Use this layout for standalone API services of small to medium size with a handful of contributors. Examples include ASP.NET Core minimal APIs, Hono, Express, and Fastify.

## When it doesn't

Full-stack framework apps use feature folders that follow the framework's own conventions. This includes Next.js, TanStack Start, Remix or React Router in framework mode, SvelteKit, Nuxt, Rails, and Laravel.

Always follow an existing project's established structure over this default unless the user asks to change it.

## Layout

- `endpoints/` holds every route. In .NET, name the folder `Endpoints/`. In Node, `routes/` is fine if the project already uses that term. Use one file per area, named `<area>-endpoints`, such as `AppEndpoints.cs` and `GroupEndpoints.cs` in .NET. Each file exposes one registration function for its area. In .NET that is an extension method such as `public static void MapApps(this RouteGroupBuilder api)`. In Hono it is a function that mounts the area's routes on the app or router passed to it. Put request and response types at the top of the endpoints file that serves them, because they are the API contract.
- `services/` holds everything the endpoints call. That includes domain logic, integrations such as a Graph or payment client, background senders, auth handlers, and small shared helpers that would otherwise end up buried in an endpoints file, such as ID normalization, error classification, and claim names. Name each file for what it is, such as `session-service` or `audit-log`. Don't use generic buckets like `utils`, `helpers`, or `common`.
- `data/` holds the database context or client, the entities or schema, and the migrations.
- The entry point, such as `Program.cs`, `index.ts`, or `server.ts`, wires up services and keeps an explicit list of route registrations, so the whole route table is visible in one place. For example, `var admin = app.MapAdminApi(); admin.MapApps(); admin.MapGroups(); admin.MapSessions();`. When a feature's startup code gets long, such as auth and cookie setup or OIDC server setup, move it to an extension method in `services/<area>-setup`. The route list stays in the entry point.
- The endpoints file that creates a route group owns that group's shared concerns, such as the auth policy, filters like "writes must be JSON", and OpenAPI metadata.

## Rules

- Only `endpoints/` registers routes. You can check this with a search. In .NET, `rg "\.Map(Get|Post|Put|Delete|Methods|Group)\(" -g "*.cs"` should only match endpoint files and the entry point's root and fallback routes. In Node, `rg "\b(app|router)\.(get|post|put|patch|delete|route|use)\(" -g "*.ts"` should only match endpoint files and the entry point.
- Dependencies point one way, from endpoints to services to data. Services never import endpoints. Data imports nothing from the other two folders.
- Tests mirror endpoint files, with one test file per endpoint file, such as `AppTests` and `GroupTests`. Write integration tests that go through the real app, following the testing rules in this skill. Put shared test fixtures in a `support` folder or at the test project root.
- Namespaces and modules follow folders. In .NET, that means `<App>.Endpoints`, `<App>.Services`, and `<App>.Data`.

## Don't

- Don't use feature folders or per-feature `Endpoints/` subfolders in a small API. A subfolder per area usually holds one file.
- Don't use one file per endpoint (REPR) at this size. It produces dozens of tiny files that all lean on the same local helpers.
- Don't use auto-discovery or registration libraries, such as Carter, FastEndpoints scanning, reflection-based `IEndpoint`, or file-system routing in a plain API. They hide the route table.
- Don't add a mediator layer, such as MediatR, or split a small service into separate Infrastructure and Application projects. They add indirection a small service doesn't need.

## When to switch to feature folders

Switch an API to feature folders when one area has grown enough files that pairing names across `endpoints/` and `services/` gets tedious. A rough signal is an area with four or more service files, or an API past roughly 10,000 hand-written lines. Also switch when several teams own different areas. Move one area at a time.

## Examples

An ASP.NET Core minimal API:

```text
service/
  Program.cs            wiring and the explicit Map list
  Endpoints/            AdminEndpoints.cs, AppEndpoints.cs, AuditEndpoints.cs, GroupEndpoints.cs, SessionEndpoints.cs
  Services/             AppAccess.cs, AuditLog.cs, AuthSetup.cs, BackchannelLogoutService.cs, ObjectIds.cs, SessionService.cs
  Data/                 ApplicationDbContext.cs, entities, Migrations/
tests/
  AppTests.cs, AuditTests.cs, GroupTests.cs, SessionTests.cs, IdpFactory.cs
```

A Hono API:

```text
src/
  index.ts              wiring and the explicit route list
  routes/               app-endpoints.ts, group-endpoints.ts, session-endpoints.ts
  services/             audit-log.ts, payment-client.ts, session-service.ts
  data/                 db.ts, schema.ts, migrations/
tests/
  app.test.ts, group.test.ts, session.test.ts, support/
```

## Sources

- Microsoft's [route handlers in minimal APIs](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis/route-handlers) shows endpoints organized in static classes and grouped with `MapGroup`.
- The [dotnet/eShop Catalog API](https://github.com/dotnet/eShop/tree/main/src/Catalog.API) splits each service into `Apis/`, `Services/`, and `Infrastructure/`.
- Jimmy Bogard's [vertical slice architecture](https://www.jimmybogard.com/vertical-slice-architecture/) describes the package-by-feature approach this layout is contrasted with.
