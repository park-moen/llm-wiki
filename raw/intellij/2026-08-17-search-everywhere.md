# Search everywhere

> Source: https://www.jetbrains.com/help/idea/searching-everywhere.html
> Collected: 2026-09-22
> Published: 2026-08-17

You can find any item in the project or outside of it by its name. You can search for files, actions, classes, symbols, settings, UI elements, and anything in Git from a single entry point.

## Search everywhere

1. In the main menu, go to Navigate | Search Everywhere or press `Shift` twice to open the search window. By default, IntelliJ IDEA displays the list of recent files.

Pressing double `Shift` again or `Alt+N` for mnemonics will select the Include non-project items checkbox and the list of search results will extend to external items.

If you switch to other tabs, select the All Places to extend the search results to non-project items.

2. Start typing your query. You can use synonyms in your search. For example, typing `toggle presentation mode` to search for the presentation mode action will display `Enter Presentation Mode` in results.

IntelliJ IDEA lists all the found results where your query is found. Press `Ctrl+Down` to jump to the bottom of the list for `more...` items or `Ctrl+Up` to return to the top of the search results.

Press `Tab` to switch the context of your search to classes, files, symbols, actions, and so on.

You can use the following shortcuts to open the search window with the needed scope right from the start:

* `Ctrl+N`: finds a class by name.
* `Ctrl+Shift+N`: finds any file or directory by name (supports CamelCase and snake_case).

If you have a directory or a file that you excluded from your project, IntelliJ IDEA will not include it in the search process.

* `Ctrl+Alt+Shift+N`: finds a symbol.

In IntelliJ IDEA, a symbol is any code element such as method, field, class, constant, and so on.

* `Ctrl+Shift+A`: finds an action by name. You can find any action even if it doesn't have a mapped shortcut or appear in the menu. For example, Emacs actions, such as kill rings, sticky selection, or hungry backspace.

To narrow down your search, click the Filter button on the window toolbar and select the appropriate option.

For example, when you search for files, you can exclude some file types from your search.

To see the results of your search in the Find tool window, click the Open in Find Tool Window button on the window toolbar. This button is disabled when you search in the Actions tab.

## Search for settings and plugins

You can search for a list of settings, their options, and plugins that you can quickly access, enable, or disable.

1. Press `Shift` twice to open the search window and type `/`. IntelliJ IDEA lists the available groups of settings.

2. Select the one you need and press `Enter`.

As a result, IntelliJ IDEA gives you quick access to the selected setting and its options.

You can also search for plugins and enable or disable them. Type `/plugins` in the search field, in the list of the search results use ON/OFF control keys to enable or disable the needed plugin.

Other tags include `/appearance`, `/system`, `/inspections`, `/registry`, `/intentions`, `/templates`, and `/vcs`.

## Search for actions

You can search for actions. For example, you can search for a VCS action and access its dialog.

1. Press `Shift` twice to open the search window.

2. In the search field, type, for example, `push`.

IntelliJ IDEA displays the Push action in the Actions section together with the `Ctrl+Shift+K` shortcut, which lets you access the Push dialog.

If the action doesn't have a shortcut, you can assign it without leaving the Search Everywhere window.

After typing the action name in the search results, select it, press `Alt+Enter` and in the dialog that opens specify a new shortcut.

## Customize the Search Everywhere window shortcut

If you need to customize the assigned shortcut (double `Shift`) for the Search Everywhere window, use the following steps:

1. Press `Ctrl+Alt+S` to open settings and then select Keymap.

2. In the search field, start typing Search Everywhere. In the search results, locate the option under the Navigate node and double-click it.

3. From the context menu that opens, select the action you want to perform (add, remove, or edit the shortcut), then click OK.
