---
layout: docu
title: CLI API
---

## Installation

The DuckDB CLI (Command Line Interface) is a single, dependency-free executable. It is precompiled for Windows, Mac, and Linux for both the stable version and for nightly builds produced by GitHub Actions. Please see the [installation page](../installation) under the CLI tab for download links.

The DuckDB CLI is based on the SQLite command line shell, so CLI-client-specific functionality is similar to what is described in the [SQLite documentation](https://www.sqlite.org/cli.html) (although DuckDB's SQL syntax follows PostgreSQL conventions).


## Getting Started

Once the CLI executable has been downloaded, unzip it and save it to any directory. Navigate to that directory in a terminal and enter the command `duckdb` to run the executable. If in a PowerShell or POSIX shell environment, use the command `./duckdb` instead.

The executable can be configured in many ways when started. Some common configurations include: 
* `-csv`, to set the output mode to CSV
* `-json` to set the output mode to JSON
* `-readonly` to open the database in read-only mode

> DuckDB has two options for concurrent access: Either one process runs which can both read and write to the database, or multiple processes can read from the database but no processes can write (`-readonly`). See [concurrency in DuckDB](/faq#how-does-duckdb-handle-concurrency) for more details.

To see additional command line options to use when starting the CLI, use the command `duckdb -help`.

> DuckDB has a [tldr page](https://github.com/tldr-pages/tldr/blob/main/pages/common/duckdb.md). If you have [tldr](https://github.com/tldr-pages/tldr) installed, you can display it by running `tldr duckdb`.

Frequently-used configurations can be stored in the file `~/.duckdbrc`. See the [Configuring the CLI](#configuring-the-cli) below for further information on these options.

By default, the CLI will open a temporary in-memory database. To open or create a persistent database, simply include a path as a command line argument like `duckdb path/to/my_database.duckdb`. This path can point to an existing database or to a file that does not yet exist and DuckDB will open or create a database at that location as needed. The file may have any arbitrary extension, but `.db` or `.duckdb` are two common choices. You will see a prompt like the below, with a `D` on the final line.

```text
v0.9.2 3c695d7ba9
Enter ".help" for usage hints.
Connected to a transient in-memory database.
Use ".open FILENAME" to reopen on a persistent database.
D
```

Once the CLI has been opened, enter a SQL statement followed by a semicolon, then hit enter and it will be executed. Results will be displayed in a table in the terminal. If a semicolon is omitted, hitting enter will allow for multi-line SQL statements to be entered.

```sql
SELECT 'quack' AS my_column;
```

<div class="narrow_table"></div>

| my_column |
|-----------|
| quack     |

```sql
SELECT
    'nicely formatted quack' AS my_column,
    'excited quacking' AS another_column;
```

<div class="narrow_table"></div>

|       my_column        |  another_column  |
|------------------------|------------------|
| nicely formatted quack | excited quacking |

The CLI supports all of DuckDB's rich SQL syntax including `SELECT`, `CREATE`, and `ALTER` statements, etc.

To exit the CLI, press `Ctrl`-`D` if your platform supports it. Otherwise press `Ctrl`-`C`. If using a persistent database, it will automatically checkpoint (save the latest edits to disk) and close. This will remove the .WAL file (the Write-Ahead-Log) and consolidate all of your data into the single file database.

## Special Commands (Dot Commands)

In addition to SQL syntax, special dot commands may be entered that are specific to the CLI client. To use one of these commands, begin the line with a period (`.`) immediately followed by the name of the command you wish to execute. Additional arguments to the command are entered, space separated, after the command. If an argument must contain a space, either single or double quotes may be used to wrap that parameter. Dot commands must be entered on a single line, and no whitespace may occur before the period. No semicolon is required at the end of the line. To see available commands, use the `.help` command:

```text
.help
.bail on|off             Stop after hitting an error.  Default OFF
.binary on|off           Turn binary output on or off.  Default OFF
.cd DIRECTORY            Change the working directory to DIRECTORY
.changes on|off          Show number of rows changed by SQL
.check GLOB              Fail if output since .testcase does not match
.clone NEWDB             Clone data into NEWDB from the existing database
.columns                 Column-wise rendering of query results
.constant ?COLOR?        Sets the syntax highlighting color used for constant values
.constantcode ?CODE?     Sets the syntax highlighting terminal code used for constant values
.databases               List names and files of attached databases
.dump ?TABLE?            Render database content as SQL
.echo on|off             Turn command echo on or off
.excel                   Display the output of next command in spreadsheet
.exit ?CODE?             Exit this program with return-code CODE
.explain ?on|off|auto?   Change the EXPLAIN formatting mode.  Default: auto
.fullschema ?--indent?   Show schema and the content of sqlite_stat tables
.headers on|off          Turn display of headers on or off
.help ?-all? ?PATTERN?   Show help text for PATTERN
.highlight [on|off]      Toggle syntax highlighting in the shell on/off
.import FILE TABLE       Import data from FILE into TABLE
.indexes ?TABLE?         Show names of indexes
.keyword ?COLOR?         Sets the syntax highlighting color used for keywords
.keywordcode ?CODE?      Sets the syntax highlighting terminal code used for keywords
.lint OPTIONS            Report potential schema issues.
.log FILE|off            Turn logging on or off.  FILE can be stderr/stdout
.maxrows COUNT           Sets the maximum number of rows for display. Only for duckbox mode.
.maxwidth COUNT          Sets the maximum width in characters. 0 defaults to terminal width. Only for duckbox mode.
.mode MODE ?TABLE?       Set output mode
.nullvalue STRING        Use STRING in place of NULL values
.once ?OPTIONS? ?FILE?   Output for the next SQL command only to FILE
.open ?OPTIONS? ?FILE?   Close existing database and reopen FILE
.output ?FILE?           Send output to FILE or stdout if FILE is omitted
.parameter CMD ...       Manage SQL parameter bindings
.print STRING...         Print literal STRING
.prompt MAIN CONTINUE    Replace the standard prompts
.quit                    Exit this program
.read FILE               Read input from FILE
.rows                    Row-wise rendering of query results (default)
.schema ?PATTERN?        Show the CREATE statements matching PATTERN
.separator COL ?ROW?     Change the column and row separators
.sha3sum ...             Compute a SHA3 hash of database content
.shell CMD ARGS...       Run CMD ARGS... in a system shell
.show                    Show the current values for various settings
.system CMD ARGS...      Run CMD ARGS... in a system shell
.tables ?TABLE?          List names of tables matching LIKE pattern TABLE
.testcase NAME           Begin redirecting output to 'testcase-out.txt'
.timer on|off            Turn SQL timer on or off
.width NUM1 NUM2 ...     Set minimum column widths for columnar output
```

Note that the above list of methods is extensive, and DuckDB supports only a subset of the commands that are displayed. Please file a [GitHub issue](https://github.com/duckdb/duckdb/issues) if a command that is central to your workflow is not yet supported.

As an example of passing an argument to a dot command, the `.help` text may be filtered by passing in a text string as the second argument.

```text
.help sh
```

```text
.sha3sum ...             Compute a SHA3 hash of database content
.shell CMD ARGS...       Run CMD ARGS... in a system shell
.show                    Show the current values for various settings
```


## Syntax Highlighting

By default the shell includes support for syntax highlighting. Syntax highlighting can be disabled using the `.highlight off` command.

The colors of the syntax highlighting can also be configured using the following commands.

```text
.constant
Error: Expected usage: .constant [red|green|yellow|blue|magenta|cyan|white|brightblack|brightred|brightgreen|brightyellow|brightblue|brightmagenta|brightcyan|brightwhite]
```
```text
.keyword
Error: Expected usage: .keyword [red|green|yellow|blue|magenta|cyan|white|brightblack|brightred|brightgreen|brightyellow|brightblue|brightmagenta|brightcyan|brightwhite]
```
```text
.keywordcode
Error: Expected usage: .keywordcode [terminal_code]
```
```text
.constantcode
Error: Expected usage: .constantcode [terminal_code]
```

## Auto-Complete

The shell offers context-aware auto-complete of SQL queries. Auto-complete is triggered by pressing the tab character. The shell auto-completes four different groups: (1) keywords, (2) table names + table functions, (3) column names + scalar functions, and (4) file names. The shell looks at the position in the SQL statement to determine which of these auto-completions to trigger. For example:

```sql
S -> SELECT

SELECT s -> student_id

SELECT student_id F -> FROM


SELECT student_id FROM g -> grades

SELECT student_id FROM 'd -> data/

SELECT student_id FROM 'data/ -> data/grades.csv
```

## Output Formats

The `.mode` command may be used to change the appearance of the tables returned in the terminal output. In addition to customizing the appearance, these modes have additional benefits. This can be useful for presenting DuckDB output elsewhere by redirecting the terminal output to a file, for example (see "Writing Results to a File" section below). Using the `insert` mode will build a series of SQL statements that can be used to insert the data at a later point. The `markdown` mode is particularly useful for building documentation!

<div class="narrow_table"></div>

|    mode    |                  description                 |
|------------|----------------------------------------------|
| ascii      | Columns/rows delimited by 0x1F and 0x1E      |
| box        | Tables using unicode box-drawing characters  |
| csv        | Comma-separated values                       |
| column     | Output in columns.  (See .width)             |
| duckbox    | Tables with extensive features               |
| html       | HTML `<table>` code                          |
| insert     | SQL insert statements for TABLE              |
| json       | Results in a JSON array                      |
| jsonlines  | Results in a NDJSON                          |
| latex      | LaTeX tabular environment code               |
| line       | One value per line                           |
| list       | Values delimited by "\|"                     |
| markdown   | Markdown table format                        |
| quote      | Escape answers as for SQL                    |
| table      | ASCII-art table                              |
| tabs       | Tab-separated values                         |
| tcl        | TCL list elements                            |
| trash      | No output                                    |

```sql
.mode markdown
SELECT 'quacking intensifies' AS incoming_ducks;
```

```text
|    incoming_ducks    |
|----------------------|
| quacking intensifies |
```

The output appearance can also be adjusted with the `.separator` command. If using an export mode that relies on a separator (`csv` or `tabs` for example), the separator will be reset when the mode is changed. For example, `.mode csv` will set the separator to a comma (`,`). Using `.separator "|"` will then convert the output to be pipe separated.

```sql
.mode csv
SELECT 1 AS col_1, 2 AS col_2
UNION ALL
SELECT 10 AS col1, 20 AS col_2;
```

```csv
col_1,col_2
1,2
10,20
```

```sql
.separator "|"
SELECT 1 AS col_1, 2 AS col_2
UNION ALL
SELECT 10 AS col1, 20 AS col_2;
```

```csv
col_1|col_2
1|2
10|20
```

## Prepared Statements

The DuckDB CLI supports executing prepared statements in addition to normal `SELECT` statements. To create a prepared
statement, use the `PREPARE` statement
```sql
PREPARE S1 AS SELECT * FROM my_table WHERE my_column < $1 OR my_column > $2;
```
To run the prepared statement with parameters, use the `EXECUTE` statement
```sql
EXECUTE S1(42, 101);
```

## Querying the Database Schema

All DuckDB clients support [querying the database schema with SQL](../sql/information_schema), but the CLI has additional dot commands that can make it easier to understand the contents of a database.
The `.tables` command will return a list of tables in the database. It has an optional argument that will filter the results according to a [`LIKE` pattern](../sql/functions/patternmatching#like).

```sql
CREATE TABLE swimmers AS SELECT 'duck' AS animal;
CREATE TABLE fliers AS SELECT 'duck' AS animal;
CREATE TABLE walkers AS SELECT 'duck' AS animal;
.tables
```

```text
fliers    swimmers  walkers
```

For example, to filter to only tables that contain an "l", use the `LIKE` pattern `%l%`.

```sql
.tables %l%
```

```text
fliers   walkers
```

The `.schema` command will show all of the SQL statements used to define the schema of the database.

```text
.schema
```

```sql
CREATE TABLE fliers(animal VARCHAR);
CREATE TABLE swimmers(animal VARCHAR);
CREATE TABLE walkers(animal VARCHAR);
```

## Opening Database Files

In addition to connecting to a database when opening the CLI, a new database connection can be made by using the `.open` command. If no additional parameters are supplied, a new in-memory database connection is created. This database will not be persisted when the CLI connection is closed.

```text
.open
```

The `.open` command optionally accepts several options, but the final parameter can be used to indicate a path to a persistent database (or where one should be created). The special string `:memory:` can also be used to open a temporary in-memory database.

```text
.open persistent.duckdb
```

One important option accepted by `.open` is the `--readonly` flag. This disallows any editing of the database. To open in read only mode, the database must already exist. This also means that a new in-memory database can't be opened in read only mode since in-memory databases are created upon connection.

```text
.open --readonly preexisting.duckdb
```

## Writing Results to a File

By default, the DuckDB CLI sends results to the terminal's standard output. However, this can be modified using either the `.output` or `.once` commands. Pass in the desired output file location as a parameter. The `.once` command will only output the next set of results and then revert to standard out, but `.output` will redirect all subsequent output to that file location. Note that each result will overwrite the entire file at that destination. To revert back to standard output, enter `.output` with no file parameter.

In this example, the output format is changed to markdown, the destination is identified as a markdown file, and then DuckDB will write the output of the SQL statement to that file. Output is then reverted to standard output using `.output` with no parameter.

```sql
.mode markdown
.output my_results.md
SELECT 'taking flight' AS output_column;
.output
SELECT 'back to the terminal' AS displayed_column;
```

The file `my_results.md` will then contain:

```text
| output_column |
|---------------|
| taking flight |
```

The terminal will then display:

```text
|   displayed_column   |
|----------------------|
| back to the terminal |
```

A common output format is CSV, or comma separated values. DuckDB supports [SQL syntax to export data as CSV or Parquet](../sql/statements/copy#copy-to), but the CLI-specific commands may be used to write a CSV instead if desired.

```sql
.mode csv
.once my_output_file.csv
SELECT 1 AS col_1, 2 AS col_2
UNION ALL
SELECT 10 AS col1, 20 AS col_2;
```

The file my_output_file.csv will then contain:

```csv
col_1,col_2
1,2
10,20
```
<!-- TODO: Document .output -e and -x -->

By passing special options (flags) to the `.once` command, query results can also be sent to a temporary file and automatically opened in the user's default program. Use either the `-e` flag for a text file (opened in the default text editor), or the `-x` flag for a CSV file (opened in the default spreadsheet editor). This is useful for more detailed inspection of query results, especially if there is a relatively large result set. The `.excel` command is equivalent to `.once -x`.

```sql
.once -e
SELECT 'quack' AS hello;
```

The results then open in the default text file editor of the system, for example:

<img src="/images/cli_docs_output_to_text_editor.jpg" alt="cli_docs_output_to_text_editor" title="Output to text editor" style="width:293px;"/>

## Import Data from CSV

DuckDB supports [SQL syntax to directly query or import CSV files](../data/csv), but the CLI-specific commands may be used to import a CSV instead if desired. The `.import` command takes two arguments and also supports several options. The first argument is the path to the CSV file, and the second is the name of the DuckDB table to create. Since DuckDB requires stricter typing than SQLite (upon which the DuckDB CLI is based), the destination table must be created before using the `.import` command. To automatically detect the schema and create a table from a CSV, see the [`read_csv_auto` examples in the import docs](../data/csv).

In this example, a CSV file is generated by changing to CSV mode and setting an output file location:

```sql
.mode csv
.output import_example.csv
SELECT 1 AS col_1, 2 AS col_2 UNION ALL SELECT 10 AS col1, 20 AS col_2;
```
<!-- TODO: If automatic table creation is added, document it! -->

Now that the CSV has been written, a table can be created with the desired schema and the CSV can be imported. The output is reset to the terminal to avoid continuing to edit the output file specified above. The `--skip N` option is used to ignore the first row of data since it is a header row and the table has already been created with the correct column names.

```sql
.mode csv
.output
CREATE TABLE test_table (col_1 INT, col_2 INT);
.import import_example.csv test_table --skip 1
```

Note that the `.import` command utilizes the current `.mode` and `.separator` settings when identifying the structure of the data to import. The `--csv` option can be used to override that behavior.

```sql
.import import_example.csv test_table --skip 1 --csv
```

## Reading SQL from a File

The DuckDB CLI can read both SQL commands and dot commands from an external file instead of the terminal using the `.read` command. This allows for a number of commands to be run in sequence and allows command sequences to be saved and reused.

The `.read` command requires only one argument: the path to the file containing the SQL and/or commands to execute. After running the commands in the file, control will revert back to the terminal. Output from the execution of that file is governed by the same `.output` and `.once` commands that have been discussed previously. This allows the output to be displayed back to the terminal, as in the first example below, or out to another file, as in the second example.

In this example, the file `select_example.sql` is located in the same directory as duckdb.exe and contains the following SQL statement:
```sql
SELECT
    *
FROM generate_series(5);
```

To execute it from the CLI, the `.read` command is used.

```text
.read select_example.sql
```

The output below is returned to the terminal by default. The formatting of the table be adjusted using the `.output` or `.once` commands.

```text
| generate_series |
|-----------------|
| 0               |
| 1               |
| 2               |
| 3               |
| 4               |
| 5               |
```

Multiple commands, including both SQL and dot commands, can also be run in a single `.read` command. In this example, the file `write_markdown_to_file.sql` is located in the same directory as duckdb.exe and contains the following commands:

```sql
.mode markdown
.output series.md
SELECT
    *
FROM generate_series(5);
```

To execute it from the CLI, the `.read` command is used as before.

```text
.read write_markdown_to_file.sql
```

In this case, no output is returned to the terminal. Instead, the file `series.md` is created (or replaced if it already existed) with the markdown-formatted results shown here:

```text
| generate_series |
|-----------------|
| 0               |
| 1               |
| 2               |
| 3               |
| 4               |
| 5               |
```

<!-- The edit function does not appear to work -->

## Configuring the CLI

The various dot commands above can be used to configure the CLI. On start-up, the CLI reads and executes all commands in the file `~/.duckdbrc`. This allows you to store the configuration state of the CLI. 
This file is passed to a `.read` command at startup, so any series of dot commands and SQL commands may be included. 
You may also point to a different initialization file using the `-init` switch.

As an example, a file in the same directory as the DuckDB CLI named `select_example` will change the DuckDB prompt to be a duck head and run a SQL statement.
Note that the duck head is built with unicode characters and does not always work in all terminal environments (like Windows, unless running with WSL and using the Windows Terminal).

```sql
-- Duck head prompt
.prompt '⚫◗ '
-- Example SQL statement
SELECT 'Begin quacking!' AS "Ready, Set, ...";
```

To invoke that file on initialization, use this command:

```text
$ ./duckdb -init select_example
```

This outputs:
```text
-- Loading resources from /home/<user>/.duckdbrc
┌─────────────────┐
│ Ready, Set, ... │
│     varchar     │
├─────────────────┤
│ Begin quacking! │
└─────────────────┘
v<version> <git hash>
Enter ".help" for usage hints.
Connected to a transient in-memory database.
Use ".open FILENAME" to reopen on a persistent database.
⚫◗
```

## Non-interactive Usage

To read/process a file and exit immediately, pipe the file contents in to `duckdb`:

```bash
$ ./duckdb < select_example.sql
```

```text
| generate_series |
|-----------------|
| 0               |
| 1               |
| 2               |
| 3               |
| 4               |
| 5               |
```

To execute a command with SQL text passed in directly from the command line, call `duckdb` with two arguments: the database location (or `:memory:`), and a string with the SQL statement to execute.

```sql
./duckdb :memory: "SELECT 42 AS the_answer"
```

```text
┌────────────┐
│ the_answer │
│   int32    │
├────────────┤
│         42 │
└────────────┘
```

## Loading Extensions

The CLI does not use the SQLite shell's `.load` command. Instead, directly execute DuckDB's SQL `install` and `load` commands as you would other SQL statements. See the [Extension docs](../extensions/overview) for details.

```text
INSTALL fts;
LOAD fts;
```

<!-- SQL parameters do not appear to work -->

## Reading from stdin and Writing to stdout

When in a Unix environment, it can be useful to pipe data between multiple commands. 
DuckDB is able to read data from stdin as well as write to stdout using the file location of stdin (`/dev/stdin`) and stdout (`/dev/stdout`) within SQL commands, as pipes act very similarly to file handles.

This command will create an example CSV:

```sql
COPY (SELECT 42 AS woot UNION ALL SELECT 43 AS woot) TO 'test.csv' (HEADER);
```

First, read a file and pipe it to the `duckdb` CLI executable. As arguments to the DuckDB CLI, pass in the location of the database to open, in this case, an in memory database, and a SQL command that utilizes `/dev/stdin` as a file location.

```sql
cat test.csv | ./duckdb :memory: "SELECT * FROM read_csv_auto('/dev/stdin')"
```

```text
┌───────┐
│ woot  │
│ int32 │
├───────┤
│    42 │
│    43 │
└───────┘
```

To write back to stdout, the copy command can be used with the `/dev/stdout` file location.

```sql
cat test.csv | ./duckdb :memory: "COPY (SELECT * FROM read_csv_auto('/dev/stdin')) TO '/dev/stdout' WITH (FORMAT 'csv', HEADER)"
```

```csv
woot
42
43
```
