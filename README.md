# Shared themes

This repository contains daisyUI, VS Code, and Ghostty themes. Install it as a Git dependency pinned to a version tag, then use the files in the relevant app-specific directory.

## Repository structure

- `daisyui/<theme-name>/` contains one daisyUI 5 theme for apps using Tailwind CSS v4, including that theme's CSS file. These CSS files are compiled into the app's build.
- `vscode/<extension>/` contains a VS Code theme extension in its native structure.
- `ghostty/themes/` contains Ghostty theme files in Ghostty's native format.

The root package includes and exports all three directories, so the pinned Git dependency delivers the native assets as well as the daisyUI CSS. VS Code themes intended to be installed as extensions need their own extension metadata within the VS Code directory.

## Install in an app

Install the version you want from GitHub:

```sh
npm install github:pimvandenbroek/themes#v1.0.0
```

Replace `v1.0.0` with the desired release tag. The dependency is recorded in the app's `package.json` and lockfile, so builds use the pinned version.

The daisyUI peer dependency is optional; apps using daisyUI themes must install daisyUI themselves.

Add the Tailwind, daisyUI, and theme imports to the app's main CSS file:

```css
@import "tailwindcss";
@plugin "daisyui";
@import "themes/daisyui/silo-terminal/silo-terminal.css";
```

Set the theme on the document (or on a subtree) with the matching daisyUI theme name:

```html
<html data-theme="silo-terminal">
```

The shared theme is not marked as a default, so it does not change the app's default theme. To make it the default, configure daisyUI in the existing plugin declaration:

```css
@plugin "daisyui" {
  themes: silo-terminal --default, light;
}
```

If `@plugin "daisyui"` is already declared, add the `themes` option to that declaration rather than declaring the plugin twice. If Tailwind cannot resolve the package import, try its explicit path:

```css
@import "../node_modules/themes/daisyui/silo-terminal/silo-terminal.css";
```

## Use multiple themes

Install the repository once, then import every theme the app will use:

```css
@import "themes/daisyui/silo-terminal/silo-terminal.css";
@import "themes/daisyui/another-theme/another-theme.css";
```

Each imported theme is available by its declared daisyUI `name`. Set `data-theme` to switch themes on the document or on an individual subtree:

```html
<html data-theme="silo-terminal">
  <section data-theme="another-theme">
    This section uses the other theme.
  </section>
</html>
```

To add a daisyUI theme, create `daisyui/<theme-name>/<theme-name>.css` and use a unique daisyUI theme name. Add VS Code themes under `vscode/<extension>/` and Ghostty themes under `ghostty/`, keeping each in its native format.

## Install the VS Code theme

The VS Code extension source is under `vscode/silo-terminal-theme/`. From an app that has installed this package, build and install the extension:

```sh
cd node_modules/themes/vscode/silo-terminal-theme
npx @vscode/vsce package
code --install-extension silo-terminal-theme-1.1.0.vsix
```

Then select **Silo Terminal** from VS Code's **Preferences: Color Theme** command. The generated `.vsix` is not checked in; it is built from the extension source.

## Install a Ghostty theme

Copy a theme file into Ghostty's user themes directory:

```sh
mkdir -p ~/.config/ghostty/themes
cp node_modules/themes/ghostty/themes/silo-terminal ~/.config/ghostty/themes/
```

Set the theme name in Ghostty's configuration:

```ini
theme = silo-terminal
```

The other included Ghostty theme is `wakelicious`; copy that file and set `theme = wakelicious` to use it instead.

## Build and deployment

Git dependencies are fetched during installation. Since npm uses Git to install them, make sure Git is available in the build environment. For example, in an Alpine-based Docker build:

```dockerfile
RUN apk add --no-cache git
RUN npm ci
```

For Debian or Ubuntu slim, install Git with `apt-get` before running `npm ci`. A multi-stage Docker build can keep Git out of the final runtime image. The theme is CSS compiled during the app build, so the final image does not need the theme package separately.

For a private repository, CI and Docker builds need authenticated Git access. Use a deploy key or token passed as a build secret; do not put credentials in build arguments or image layers.

## Releases

To release the collection, bump the root `package.json` version and push that change to `main`. The workflow compares it with the previous commit, then creates the matching `v<version>` tag and a GitHub Release with DaisyUI and Ghostty archives and a built VS Code `.vsix`. Unchanged versions and existing releases are skipped. A tag without a release is reused only if it points to the exact commit being released; existing tags are never replaced.

Apps can pin the Git dependency to the generated tag, for example `npm install github:pimvandenbroek/themes#v1.1.0`. The VS Code extension's own version in `vscode/silo-terminal-theme/package.json` identifies the VSIX; the root package version tags the full theme collection.

## Available themes

| File | Theme name | Description |
|---|---|---|
| `daisyui/silo-terminal/silo-terminal.css` | `silo-terminal` | Dark theme derived from the VS Code "Silo Terminal" theme |

Ghostty themes are in `ghostty/themes/`: `silo-terminal` and `wakelicious`. The VS Code **Silo Terminal** extension source is in `vscode/silo-terminal-theme/`.

## Updating themes

Git tags version the whole repository, including DaisyUI and Ghostty themes. Update the version in `vscode/silo-terminal-theme/package.json` when changing the VS Code extension; that version identifies the generated VSIX. To consume a repository change, install the new tag in each app that needs it:

```sh
npm install github:pimvandenbroek/themes#v1.0.1
```

Apps remain on their pinned version until explicitly updated. To use another daisyUI theme from this repository, import its CSS file and set `data-theme` to the theme's declared `name`.
