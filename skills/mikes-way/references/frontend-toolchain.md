# Frontend toolchain

Mike's frontend toolchain guidance covers Vite+, lint and format presets, framework plugins, Tailwind, shadcn/ui, and editor settings. Apply only the parts relevant to the project's stack and installed versions.

For a new frontend, follow the Vite+ scaffolding preference in [stack preferences](stack-preferences.md#toolchain).

## Vite+ configuration

This is Mike's starting `vite.config.ts` for a React and Tailwind v4 app. Keep it when the installed versions support it and the repo has not chosen different rules. Remove the framework-specific presets, plugins, and comments that do not apply. Ultracite 7.12 or newer ships the `shadcn` preset used below; on older versions, upgrade rather than writing the plugin entry and rules by hand.

```ts
// OpenAPI client projects only.
import { heyApiPlugin } from "@hey-api/vite-plugin";
import tailwindcss from "@tailwindcss/vite";
// TanStack Router projects only.
import { tanstackRouter } from "@tanstack/router-plugin/vite";
import reactVite from "@vitejs/plugin-react";
import fmt from "ultracite/oxfmt";
import antiSlop from "ultracite/oxlint/anti-slop";
import core from "ultracite/oxlint/core";
import jsPlugins, { jsPluginSettings } from "ultracite/oxlint/js-plugins";
import reactUltracite from "ultracite/oxlint/react";
import shadcn from "ultracite/oxlint/shadcn";
// TanStack projects only, including the matching presets and plugin below.
import tanstack from "ultracite/oxlint/tanstack";
import tanstackJsPlugins from "ultracite/oxlint/tanstack/js-plugins";
// Vitest projects only.
import vitest from "ultracite/oxlint/vitest";
import { defineConfig, lazyPlugins } from "vite-plus";

// dotnet build rewrites openapi.json on every build, so formatting it would
// leave a diff after every build.
const openApiDocument = "openapi.json";

// https://vite.dev/config/
export default defineConfig({
  fmt: {
    ...fmt,
    ignorePatterns: [...(fmt.ignorePatterns ?? []), openApiDocument],
    singleAttributePerLine: true,
  },
  lint: {
    extends: [
      core,
      reactUltracite,
      jsPlugins,
      antiSlop,
      shadcn,
      // Vitest projects only.
      vitest,
      // TanStack projects only.
      tanstack,
      tanstackJsPlugins,
    ],
    ignorePatterns: [...(core.ignorePatterns ?? []), openApiDocument],
    jsPlugins: [
      ...(jsPlugins.jsPlugins ?? []),
      {
        name: "vite-plus",
        specifier: "vite-plus/oxlint-plugin",
      },
      {
        name: "stylistic",
        specifier: "@stylistic/eslint-plugin",
      },
      {
        name: "eslint-js",
        specifier: "oxlint-plugin-eslint",
      },
    ],
    options: {
      typeAware: true,
      typeCheck: true,
    },
    overrides: [
      {
        files: ["src/**"],
        rules: {
          "no-restricted-imports": [
            "error",
            {
              patterns: [
                {
                  message:
                    "Use the @/* alias for imports outside the parent folder.",
                  regex: "^\\.\\./\\.\\./",
                },
              ],
            },
          ],
        },
      },
    ],
    rules: {
      "eslint-js/no-restricted-syntax": [
        "error",
        {
          message: "Reserve export {} for multiple exports.",
          selector: "ExportNamedDeclaration[source=null][specifiers.length=1]",
        },
        {
          message: "Use React.* for React types instead of named type imports.",
          selector:
            "ImportDeclaration[source.value='react'][importKind='type'] > ImportSpecifier, ImportDeclaration[source.value='react'] > ImportSpecifier[importKind='type']",
        },
      ],
      // Remove unused imports on save.
      "no-unused-vars": [
        "error",
        {
          fix: {
            imports: "safe-fix",
            variables: "off",
          },
        },
      ],
      // Reduce max cognitive complexity (Ultracite defaults to 20 to match Biome).
      "sonarjs/cognitive-complexity": ["error", 13],
      // Spacing rules for code legibility
      "stylistic/jsx-newline": [
        "error",
        {
          allowMultilines: true,
          prevent: true,
        },
      ],
      "stylistic/padding-line-between-statements": [
        "error",
        {
          blankLine: "always",
          next: "*",
          prev: [
            "multiline-block-like",
            "multiline-const",
            "multiline-let",
            "multiline-expression",
          ],
        },
        {
          blankLine: "always",
          next: [
            "multiline-block-like",
            "multiline-const",
            "multiline-let",
            "multiline-expression",
          ],
          prev: "*",
        },
        {
          blankLine: "never",
          next: ["singleline-const", "singleline-let"],
          prev: ["singleline-const", "singleline-let"],
        },
        {
          blankLine: "always",
          next: { selector: "Program > *" },
          prev: { selector: "Program > *" },
        },
        {
          blankLine: "any",
          next: "import",
          prev: "import",
        },
      ],
      "vite-plus/prefer-vite-plus-imports": "error",
    },
    settings: jsPluginSettings,
  },
  plugins: lazyPlugins(() => [
    // OpenAPI client projects only; see Hey API below.
    heyApiPlugin({
      config: {
        input: openApiDocument,
        output: {
          entryFile: false,
          path: "src/api",
          postProcess: [{ args: ["fmt", "{{path}}"], command: "vp" }],
        },
        plugins: [
          { name: "@hey-api/sdk", validator: true },
          { dates: { local: true, offset: true }, name: "zod" },
          { name: "@tanstack/react-query", queryKeys: { tags: true } },
        ],
      },
    }),
    // TanStack Router projects only.
    tanstackRouter({
      autoCodeSplitting: true,
      target: "react",
    }),
    reactVite({ compiler: true }),
    tailwindcss(),
  ]),
  resolve: {
    tsconfigPaths: true,
  },
  // Omit in projects committed from Visual Studio; see below.
  staged: {
    "*": "vp check --fix",
  },
});
```

The restricted-imports pattern allows `./` and `../` imports. Imports starting with `../../` must use `@/*`, or the project's equivalent source alias. Keep the TanStack imports, their `lint.extends` entries, and the router plugin only in TanStack projects. The `tanstack` and `js-plugins` presets already relax the filename-case rules inside `routes/` directories, so TanStack Router's file-based routes need no extra override.

`staged` runs `vp check --fix` on staged files through Vite+'s pre-commit hook; run `vp hooks status` to confirm the hook is installed. Do not add a separate lint-staged setup beside it. Omit `staged` in projects that also contain a .NET solution, or any project Mike commits from Visual Studio. Visual Studio doesn't surface a failing client-side lint hook clearly, so a lint error looks like a broken commit. Run `vp check` in CI or on demand for those projects instead.

### Path aliases

Enable `resolve.tsconfigPaths` and let Vite read the `paths` from `tsconfig.json`. Do not add a manual `resolve.alias` entry for `@/*` or duplicate the alias in a second place; `tsconfig.json` is the only place aliases are declared. Keep `resolve.alias` for cases tsconfig cannot express, such as remapping a dependency.

### Oxc React Compiler

Mike uses the Rust-based React Compiler through `@vitejs/plugin-react`, not the Babel plugin. Install `oxc-transform-react` as a dev dependency in the app package and pass `compiler: true` to the React plugin, as shown above:

```sh
vp add -D oxc-transform-react
```

Remove `babel-plugin-react-compiler`, `@babel/*` packages, Babel plugin options on the React plugin, and Babel configuration files that existed only for the replaced compiler. Do not leave the Babel compiler active alongside Oxc. Preserve Babel configuration that another build path still needs. Verify the installed React plugin supports the `compiler` option and run the build. See the [Oxc React Compiler guide](https://oxc.rs/docs/guide/usage/transformer/react-compiler).

### Hey API

Generate the API client with Hey API's Vite plugin, as configured above:

- `entryFile: false` skips the `index.ts` barrel, so app code imports from the generated file that owns each piece, such as `@/api/@tanstack/react-query.gen`, `@/api/types.gen`, `@/api/zod.gen`, and `@/api/client.gen`.
- `postProcess` runs `vp fmt` over the output, so regenerating doesn't undo `vp check --fix` on Hey API's non-`.gen` files.
- `validator: true` validates both requests and responses with the Zod plugin.
- `dates` accepts .NET's ISO date-times with and without an offset.
- `queryKeys.tags` lets you invalidate queries by OpenAPI tag.

The native TypeScript types are the source of truth; Zod only supplies runtime schemas. Don't enable the Zod plugin's `types.infer`: the SDK then types responses with the Zod types and loses the status-code map ([hey-api#3290](https://github.com/hey-api/hey-api/issues/3290); fix in progress in [#4454](https://github.com/hey-api/hey-api/pull/4454)). Revisit once that ships.

Hey API throws the raw response body without the status. To detect a 401, register `client.interceptors.error.use((cause, response) => …)` on `client` from `@/api/client.gen` and check `response.status`.

Ultracite's `core` and `oxfmt` presets already ignore `**/*.gen.*`; lint and format the rest of `src/api`. Hey API's `src/api/client/index.ts` re-exports `TDataShape`, so add a `lint.overrides` entry that turns off only `anti-slop/no-shape-in-symbol-names` for that file:

```ts
{
  files: ["src/api/client/index.ts"],
  rules: { "anti-slop/no-shape-in-symbol-names": "off" },
},
```

The config ignores the .NET build-time `openapi.json` in both tools, after each preset's own patterns. Without a build-time document, drop `openApiDocument` and the ignore entries, and point `input` at the server's OpenAPI URL.

`vp check` skips ignored files, so it never sees type errors in generated code. When generated code is committed, make the check script type-check the app too, using the app's tsconfig:

```json
"check": "vp check --fix && tsc --noEmit -p tsconfig.app.json"
```

### Tailwind

Use `@tailwindcss/vite` as a dev dependency in the app package rather than the PostCSS plugin:

```sh
vp add -D @tailwindcss/vite
```

## Lint plugin dependencies

Install the plugins referenced by the selected Ultracite presets as direct dev dependencies in the package that owns `vite.config.ts`. Oxlint resolves their specifiers from the project, so extending the presets alone is not enough. For the configuration above:

```sh
vp add -D @shadcn/lint @stylistic/eslint-plugin eslint-plugin-github eslint-plugin-sonarjs oxlint-plugin-eslint oxlint-plugin-react-doctor
```

The `js-plugins` preset needs `eslint-plugin-github`, `eslint-plugin-sonarjs`, and `oxlint-plugin-react-doctor`. The `shadcn` preset needs `@shadcn/lint`. The `stylistic` and `eslint-js` entries need `@stylistic/eslint-plugin` and `oxlint-plugin-eslint`. The `anti-slop` preset bundles its own plugin. Keep `jsPluginSettings` on the root `lint.settings`; Oxlint does not merge settings from extended configs. Oxlint loads the preset plugins through `extends`, but dependency analyzers such as Knip only read the root `jsPlugins`, so the config spreads `jsPlugins.jsPlugins` into the root array. If Knip still reports `@shadcn/lint` as unused, spread `shadcn.jsPlugins` there too. Install any plugins required by the TanStack presets only when the project uses TanStack. Check the installed preset's plugin specifiers and supported versions, then run the project's lint command to verify they load. See [Ultracite's releases](https://github.com/haydenbleasel/ultracite/releases).

## Tailwind design-system linting

For Tailwind v4 projects, extend `ultracite/oxlint/shadcn` as shown above. It registers [shadcn's lint plugin](https://github.com/shadcn-ui/lint), which works with custom Tailwind components and themes; shadcn/ui is not required. Omit the preset and the `@shadcn/lint` dependency for projects without Tailwind v4.

Check that Node.js is at least 20.19 and Vite+'s bundled Oxlint is at least 1.80. Use the project's existing lint command, such as `vp lint` or `vp check`, to verify the plugin loads. Keep Vite+'s lint configuration in `vite.config.ts`; do not add a parallel `.oxlintrc.json` just for this plugin.

The preset enables these rules at `error`. Keep them; adjust policy through narrow exceptions rather than turning rules off.

| Rule | Policy and scope |
| --- | --- |
| [no-restyle](https://github.com/shadcn-ui/lint/blob/main/docs/rules/no-restyle.md) | `allow: ["layout"]`. Callers can set margin, size, flex, positioning, transforms, and text alignment on recognized components. Padding, gap, color, typography, and shape changes need component variants or an accepted component contract. |
| [no-raw-colors](https://github.com/shadcn-ui/lint/blob/main/docs/rules/no-raw-colors.md) | Use declared theme colors rather than raw palette classes or undeclared tokens. Also checks literal intrinsic SVG color attributes. Stays on inside design-system sources. `white`, `black`, `transparent`, `current`, and `inherit` pass. Arbitrary color values belong to no-arbitrary-values. |
| [no-arbitrary-values](https://github.com/shadcn-ui/lint/blob/main/docs/rules/no-arbitrary-values.md) | `allow: ["layout"]`. `inline-[320px]` passes; `p-[13px]`, `rounded-[10px]`, and `bg-[#333]` need tokens, scale values, or narrow exceptions. Arbitrary variants and CSS-variable shorthands are not arbitrary values. |
| [no-inline-styles](https://github.com/shadcn-ui/lint/blob/main/docs/rules/no-inline-styles.md) | No `style` props on recognized components. Use classes and variants; pass dynamic values through CSS variables the component exposes. Stays on inside design-system sources. |
| [no-unknown-classes](https://github.com/shadcn-ui/lint/blob/main/docs/rules/no-unknown-classes.md) | Checks whether the installed Tailwind v4 can generate a class using the project's theme, utilities, variants, and plugins. Stays on inside design-system sources. Add exact exceptions for external classes the app actually loads; an allowance does not generate CSS. |
| [require-static-classes](https://github.com/shadcn-ui/lint/blob/main/docs/rules/require-static-classes.md) | Applies to recognized components and forwarding wrappers. Use complete literal classes, conditional choices, and readable class-helper arguments. Imported class values, unknown function results, and interpolated names such as `bg-${color}` cannot be checked. Plain elements are outside this rule. |

The preset already disables `no-restyle`, `no-arbitrary-values`, and `require-static-classes` inside `**/components/ui/**` so components can define internals and call their own variant functions. For a design-system directory elsewhere, add an override with the same shape for the actual path. Same-file constants can be read one hop deep. Wrappers may forward their received `className`; their authored defaults still get checked. Register custom class helpers only when their arguments describe their output. Do not assume that require-static-classes proves every class in the application is static.

Recognition and theme discovery are part of enforcement. Use `components.json` when present; verify the component aliases and theme CSS path it names. Without it, the plugin looks for `components/ui` or `src/components/ui` beside the nearest `package.json`. For a custom design system, set `settings.shadcn.ui` or `componentImports` on the root `lint.settings`, merged with `jsPluginSettings`. Verify the discovered stylesheet imports Tailwind and includes the project's tokens through its import graph. Scope component recognition per app in a workspace and preserve each app's theme discovery. Fix theme-loading warnings before treating a clean result as full coverage; no-unknown-classes falls back to a less precise grammar when Tailwind or the theme cannot load. See [settings](https://github.com/shadcn-ui/lint#settings) and [discovery and analysis limits](https://github.com/shadcn-ui/lint/blob/main/docs/how-it-works.md).

Exceptions apply to each rule independently. Allowing a color or arbitrary value does not make it acceptable to `no-restyle`. Component `contracts` match resolved names; a contract inherits omitted top-level keys, and only the last matching contract applies. `require-static-classes` has no `allow`, `deny`, or `contracts`. Avoid `deny` alone unless the intention is to allow every other match. Keep exceptions narrow and supported by styles the app actually loads. Review suggested color replacements against the design; proximity in light-mode color values does not establish the right semantic token.

See the upstream [setup guide](https://github.com/shadcn-ui/lint/blob/main/SETUP.md), [rules](https://github.com/shadcn-ui/lint/blob/main/docs/rules.md), and [design-system configuration](https://github.com/shadcn-ui/lint/blob/main/docs/design-systems.md).

## Editor settings

### TanStack Router

When TanStack Router is present, merge these into the project's `.vscode/settings.json` without replacing unrelated settings:

```json
{
  "files.readonlyInclude": {
    "**/routeTree.gen.ts": true
  },
  "files.watcherExclude": {
    "**/routeTree.gen.ts": true
  },
  "search.exclude": {
    "**/routeTree.gen.ts": true
  }
}
```
