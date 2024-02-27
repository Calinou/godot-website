---
title: "Godot Visual Studio Code 2.0 extension arrives with major improvements"
excerpt: "The official Godot Visual Studio Code extension has been updated with features including scene tree previewing, a code formatter and more, while remaining compatible with Godot 3.x and 4.x."
categories: ["news"]
author: Hugo Locurcio
image: /storage/blog/covers/vscode-extension-2.0.webp
date: 2024-03-04 14:00:00
---

The [Godot Tools Visual Studio Code extension](https://marketplace.visualstudio.com/items?itemName=geequlim.godot-tools) has just reached version 2.0, bringing dozens of new features and bug fixes! The extension retains full compatibility with Godot 3.x **and** 4.x, with most of the new features being available with both versions.

This work has been the fruit of contributors since 2022, in particular DaelonSuzuka who was central in this effort of updating this extension for Godot 4!

## New features

### Inlay hints implementation

*Inlay hints* are a feature of Visual Studio Code that lets plugins display text within a file, without this text actually being part of the file's contents. The main use case of this is displaying inferred types when using [static typing in GDScript](https://docs.godotengine.org/en/stable/tutorials/scripting/gdscript/static_typing.html).

Inlay hints are now [supported as an **experimental** feature](https://github.com/godotengine/godot-vscode-plugin/pull/589) in the extension, which must be enabled in the Visual Studio Code's Settings menu for the Godot Tools extension. Note that inlay hints can take a few seconds to appear after opening a file. They also require the LSP connection to be available, so the editor must be opened or you.

| ![Inlay hints example](/storage/blog/vscode-extension-2.0/inlay-hints.webp)
|:-:|
| *Inlay hints are displayed in gray within inferred type operators (`var name := value` syntax)* |

### Support for headless LSP mode

Until now, you always had to keep the Godot editor open for the extension's autocompletion and debugging features to work. As soon as the editor was closed, you'd get an error dialog in Visual Studio Code telling you that it can't connect to the Godot editor (which is the LSP server). This added significant friction to quick project changes where you don't need to edit scenes.

If you enable the [**Headless** option](https://github.com/godotengine/godot-vscode-plugin/pull/488) in the Visual Studio Code's Settings menu for the Godot Tools extension, you'll be able to benefit from all the LSP features without having to open the Godot editor. This works using Godot's `--headless` (4.x) and `--no-window` (3.x) [command line arguments](https://docs.godotengine.org/en/stable/tutorials/editor/command_line_tutorial.html), which are simultaneously passed to Godot for backwards compatibility. Additionally, a random LSP port is given using the `--lsp-port` command line argument so that multiple Visual Studio Code instances can be used on different Godot projects at the same time.

Note that this feature will make the extension automatically start Godot, instead of having you open the Godot editor yourself. If you have multiple projects in the same workspace, the first project found in alphabetical order will be opened in headless mode by the extension. As a result, this option is disabled by default to give you more flexibility in multi-project workspaces.

### New scene preview panel

The new [scene preview panel](https://github.com/godotengine/godot-vscode-plugin/pull/413) allows viewing the node tree in your code editor when opening a scene file:

<video autoplay loop muted playsinline title="Demonstration of the scene preview panel">
  <source src="/storage/blog/vscode-extension-2.0/scene-preview-panel.mp4" type="video/mp4">
</video>

By right-clicking a node, you can choose to open its associated script as well:

<video autoplay loop muted playsinline title="Demonstration of the scene preview panel's Open Script context menu action">
  <source src="/storage/blog/vscode-extension-2.0/scene-preview-panel-open-script.mp4" type="video/mp4">
</video>

### Revamped support for Godot shader files (`.gdshader`)

Support for shader file syntax highlighting was [rewritten to be more complete](https://github.com/godotengine/godot-vscode-plugin/pull/360):

| ![Redesigned Godot shader highlighting](/storage/blog/vscode-extension-2.0/gdshader-highlighting-comparison.webp)
|:-:|
| *Redesigned Godot shader highlighting. **Top:** Before - **Bottom:** After* |

Shader files embedded in text-based scene and resource files are now highlighted as well:

| ![Godot shader highlighting embedded in resource](/storage/blog/vscode-extension-2.0/gdshader-highlighting-embedded-resource.webp)
|:-:|
| *Godot shader highlighting embedded in resource* |

### New GDScript code formatter

A new [code formatter](https://github.com/godotengine/godot-vscode-plugin/pull/529) is available in the extension. It can be used to make GDScript code comply with the [style guide](https://docs.godotengine.org/en/stable/tutorials/scripting/gdscript/gdscript_styleguide.html) by the press of a keyboard shortcut (<kbd>Ctrl + Shift + I</kbd> by default). By enabling Visual Studio Code's **Format on Save** option, you can have your code be automatically formatted every time it's saved as well.

This formatter is a standalone implementation, so it doesn't require Godot to be open or even installed to work. It also doesn't rely on [gdformat](https://github.com/Scony/godot-gdscript-toolkit) either, so you don't need to install any external utilities.

### Improved syntax highlighting

Syntax highlighting is now closer than ever to the built-in script editor. This was made possible by the following pull requests:

- [Add highlighting support for %Unique nodes in NodePaths](https://github.com/godotengine/godot-vscode-plugin/pull/403)
- [Fix `func` keyword highlighting](https://github.com/godotengine/godot-vscode-plugin/pull/398)
- [Fix OS singleton being incorrectly highlighted as a constant](https://github.com/godotengine/godot-vscode-plugin/pull/402)
- [Fix incorrect highlighting in dictionary literals](https://github.com/godotengine/godot-vscode-plugin/pull/419)
- [Fix various highlighting errors](https://github.com/godotengine/godot-vscode-plugin/pull/407)
- [Fix various syntax highlighting problems](https://github.com/godotengine/godot-vscode-plugin/pull/441)
- [Various highlighting/formatting fixes](https://github.com/godotengine/godot-vscode-plugin/pull/559)

### Rewritten documentation manager

The documentation manager is the part of the extension that retrieves and displays class reference information obtained using the [LSP](https://microsoft.github.io/language-server-protocol/) exposed by the Godot editor. This way, the extension doesn't have to bundle the entire class reference and can display up-to-date descriptions for Godot versions released after the extension.

It now uses a *virtual file* approach to displaying documentation, which relies on a custom file handler for "files" with a `.gddoc` file extension. Thanks to Visual Studio Code's behavior on these files, this allows reopening documentation tabs when an editor session is reopened:

| ![Documentation being reloaded when the Visual Studio Code window is reopened](/storage/blog/vscode-extension-2.0/new-docs-manager-reload-editor.webp)
|:-:|
| *Documentation being reloaded when the Visual Studio Code window is reopened* |

Additionally, this allows the minimap to show up in documentation pages:

| ![Minimap in documentation visible on the right side of the editor](/storage/blog/vscode-extension-2.0/new-docs-manager-minimap.webp)
|:-:|
| *Minimap in documentation visible on the right side of the editor* |

### Rewritten debugger with Godot 4 support

The debugger has been [rewritten from scratch](https://github.com/godotengine/godot-vscode-plugin/pull/452) for Godot 4 support while preserving Godot 3 support.

### Various quality-of-life improvements

#### Open Type Documentation context menu option

There is now an [**Open Type Documentation** context menu option](https://github.com/godotengine/godot-vscode-plugin/pull/405) when right-clicking text in a GDScript file:

<video autoplay loop muted playsinline title="Demonstration of the Open Type Documentation context menu option">
  <source src="/storage/blog/vscode-extension-2.0/open-type-documentation.mp4" type="video/mp4">
</video>

#### Restructured settings

[Settings have been restructured and renamed](https://github.com/godotengine/godot-vscode-plugin/pull/376) to better follow the Visual Studio Code naming guidelines:

| ![Setting renames comparison](/storage/blog/vscode-extension-2.0/setting-renames.webp)
|:-:|
| ***Left:** Before - **Right:** After* |

#### Improved startup performance

Lastly, [the extension's startup performance has been improved](https://github.com/godotengine/godot-vscode-plugin/pull/408).

## Support

Godot is a non-profit, open source game engine developed by hundreds of contributors in their free time, as well as a handful of part or full-time developers hired thanks to [generous donations from the Godot community](https://fund.godotengine.org/). A big thank you to everyone who has contributed [their time](https://github.com/godotengine/godot/blob/master/AUTHORS.md) or [their financial support](https://github.com/godotengine/godot/blob/master/DONORS.md) to the project!

If you'd like to support the project financially and help us secure our future hires, you can do so using the [Godot Development Fund](https://fund.godotengine.org/) platform managed by [Godot Foundation](https://godot.foundation/). There are also several [alternative ways to donate](/donate) which you may find more suitable.
