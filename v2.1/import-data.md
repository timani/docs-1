---
title: Import Data
summary: Learn how to import data into a CockroachDB cluster.
toc: false
---

CockroachDB supports importing data from CSV/TSV or SQL dump files.

{{site.data.alerts.callout_info}}To import/restore data from CockroachDB-generated <a href="backup.html">enterprise license backups</a>, see <a href="restore.html"><code>RESTORE</code></a>.{{site.data.alerts.end}}

<div id="toc"></div>

## Import from Tabular Data (CSV)

If you have data exported in a tabular format (e.g., CSV or TSV), you can use the [`IMPORT`](import.html) statement.

To use this statement, though, you must also have some kind of remote file server (such as Amazon S3 or a custom file server) that all your nodes can access.

## Import from Generic SQL Dump

You can execute batches of `INSERT` statements stored in `.sql` files (including those generated by [`cockroach dump`](sql-dump.html)) from the command line, importing data into your cluster.

{% include copy-clipboard.html %}
~~~ shell
$ cockroach sql --database=[database name] < statements.sql
~~~

{{site.data.alerts.callout_success}}Grouping each <code>INSERT</code> statement to include approximately 500-10,000 rows will provide the best performance. The number of rows depends on row size, column families, number of indexes; smaller rows and less complex schemas can benefit from larger groups of <code>INSERTS</code>, while larger rows and more complex schemas benefit from smaller groups.{{site.data.alerts.end}}

## Import from PostgreSQL Dump

If you're importing data from a PostgreSQL deployment, you can import the `.sql` file generated by the `pg_dump` command to more quickly import data.

{{site.data.alerts.callout_success}}The <code>.sql</code> files generated by <code>pg_dump</code> provide better performance because they use the <code>COPY</code> statement instead of bulk <code>INSERT</code> statements.{{site.data.alerts.end}}

### Create PostgreSQL SQL File

Which `pg_dump` command you want to use depends on whether you want to import your entire database or only specific tables:

- Entire database:

    {% include copy-clipboard.html %}
    ~~~ shell
    $ pg_dump [database] > [filename].sql
    ~~~

- Specific tables:

    {% include copy-clipboard.html %}
    ~~~ shell
    $ pg_dump -t [table] [table's schema] > [filename].sql
    ~~~

For more details, see PostgreSQL's documentation on [`pg_dump`](https://www.postgresql.org/docs/9.1/static/app-pgdump.html).

### Reformat SQL File

After generating the `.sql` file, you need to perform a few editing steps before importing it:

1. Remove all statements from the file besides the `CREATE TABLE` and `COPY` statements.
2. Manually add the table's [`PRIMARY KEY`](primary-key.html#syntax) constraint to the `CREATE TABLE` statement.
  This has to be done manually because PostgreSQL attempts to add the primary key after creating the table, but CockroachDB requires the primary key be defined upon table creation.
3. Review any other [constraints](constraints.html) to ensure they're properly listed on the table.
4. Remove any [unsupported elements](sql-feature-support.html).

### Import Data

After reformatting the file, you can import it through `psql`:

{% include copy-clipboard.html %}
~~~ shell
$ psql -p [port] -h [node host] -d [database] -U [user] < [file name].sql
~~~

For reference, CockroachDB uses these defaults:

- `[port]`: **26257**
- `[user]`: **root**

## See Also

- [SQL Dump (Export)](sql-dump.html)
- [Back up Data](back-up-data.html)
- [Restore Data](restore-data.html)
- [Use the Built-in SQL Client](use-the-built-in-sql-client.html)
- [Other Cockroach Commands](cockroach-commands.html)
