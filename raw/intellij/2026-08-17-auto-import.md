# Auto import

> Source: https://www.jetbrains.com/help/idea/creating-and-optimizing-imports.html
> Collected: 2026-09-22
> Published: 2026-08-17

This page describes Java imports. For more information about imports in Kotlin, refer to Packages and Imports.

If you are using a class, a static method, or a static field that you have not imported yet, the IDE shows you a tooltip prompting you to add a missing import statement so that you do not have to add it manually. Press `Alt+Enter` to accept the suggestion.

If there's more than one possible source of import, pressing `Alt+Enter` will open the list of suggestions.

## Disable import tooltips

When tooltips are disabled, unresolved references are underlined and marked with the red bulb icon. To view the list of suggestions, click this icon (or press `Alt+Enter`) and select Import class.

## Exclude a class or a package on the fly

1. Press `Alt+Enter` on a missing class to open the list of import suggestions.

2. Click the right arrow next to a package and select an item (a class or an entire package) that you want to exclude.

3. In the Exclude from auto-import and completion section of the Auto Import dialog, select whether you want to exclude items from the current project or from all projects, and apply the changes.
