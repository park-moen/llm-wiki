# Database versioning

> Source: https://www.jetbrains.com/help/idea/database-versioning.html
> Collected: 2026-09-22
> Published: 2026-08-13

## DDL by Entity

IntelliJ IDEA allows you to convert entities into DDL statements. You can generate:

* An initialization script for a single entity right from the editor.
* Initialization scripts for multiple entities: create a database schema based on the entire model or selected entities.
* Differential DDL: update an already existing database to the valid state in accordance with entity mappings.

This feature is useful if you want to avoid using the automatic scripts generation enabled by `hbm2ddl` or `ddl-auto` properties. By using this feature, you can fully control DDL before execution, set up proper Java -> DB types mapping, map fields with attribute converters and Hibernate types, generate drop statements, and many more.

### Generate DDL for single entity

IntelliJ IDEA provides an action to generate DDL statements for a specific entity.

1. With your entity source code opened in the editor, click (Show Entity actions) in the gutter and select Generate DDL. Or place the caret at the class name, press `Alt+Enter` to invoke context actions, and select Generate DDL.

Alternatively, in the Project tool window, right-click your entity class file and select New | DDL by Entity.

2. In the Generate DDL by <Entity name> window that opens, configure the options to save the DDL statement:

   * DB type: select a database management system for which you want to generate the DDL statement.
   * Save as: select how to save the DDL statement: File, Scratch File, Clipboard, or Database Console.
   * Directory: if the DDL statement is saved as a file, select the file location.
   * File name: if the DDL statement is saved as a file, select the file name.
   * DB connection: if the DDL statement is saved as a console command, select the database connection where you want to run this statement.

3. Preview the statement and click OK to save it.

### Generate database initialization script

1. Open an SQL file or a query console. In the toolbar, click the DDL action.
2. In the Init DDL dialog that opens, click Model and select a persistence unit.

To generate a DDL script for particular entities, under Scope, select Selected Entities.

3. Click OK. IntelliJ IDEA will analyze the difference between Source and Target and show the DDL Preview dialog.
4. Preview the DDL statement and click Save to insert the resulting DDL script into your query console or SQL file.
