# Auto import: Optimize imports

> Source: https://www.jetbrains.com/help/idea/creating-and-optimizing-imports.html
> Collected: 2026-09-22
> Published: 2026-08-17

## Optimize imports

The Optimize Imports feature helps you remove unused imports and organize import statements in the current file or in all files in a directory at once according to the rules specified in Settings | Editor | Code Style | <language> | Imports.

### Optimize all imports

1. Select a file or a directory in the Project tool window (View | Tool Windows | Project).
2. Do any of the following:

   * In the main menu, go to Code | Optimize Imports (or press `Ctrl+Alt+O`).
   * From the context menu, select Optimize Imports.

3. (If you've selected a directory) Choose whether you want to optimize imports in all files in the directory, or only in locally modified files (if your project is under version control), and click Run.

### Remove unused imports

1. Place the caret at the unused import statement and press `Alt+Enter` or use the Intention action button.

Unused statements are greyed out by default.

2. From the list of suggestions, select Remove unused imports.
